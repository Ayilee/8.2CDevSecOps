pipeline {
  agent any

  options { timestamps() }

  environment {
    NPM_CONFIG_FUND  = 'false'
    NPM_CONFIG_AUDIT = 'false'
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/Ayilee/8.2CDevSecOps.git'
      }
    }

    stage('Verify Tools') {
      steps {
        bat 'node -v && npm -v && git --version'
      }
    }

    stage('Install Dependencies') {
      steps {
        bat 'IF EXIST package-lock.json (npm ci) ELSE (npm install)'
      }
    }

    stage('Run Tests') {
      steps {
        bat 'npm test 1>test.log 2>&1 || exit /b 0'
        script {
          emailext(
            to: 'ayadiscord123123@gmail.com',
            subject: "${env.JOB_NAME} #${env.BUILD_NUMBER} - Test Stage Completed",
            body: """Build URL: ${env.BUILD_URL}
Stage: Run Tests
Status: Completed (pipeline continues even if tests fail)
""",
            attachmentsPattern: 'test.log',
            compressLog: true
          )
        }
      }
    }

    stage('Generate Coverage Report') {
      steps {
        bat 'npm run coverage 1>coverage.log 2>&1 || exit /b 0'
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        bat 'npm audit 1>audit.log 2>&1 || exit /b 0'
        bat 'npm audit --json > audit-report.json || exit /b 0'
        script {
          emailext(
            to: 'ayadiscord123123@gmail.com',
            subject: "${env.JOB_NAME} #${env.BUILD_NUMBER} - Security Scan (npm audit)",
            body: """Build URL: ${env.BUILD_URL}
Stage: NPM Audit (Security Scan)
Status: Completed (pipeline continues even if issues found)
Attachments: audit.log (readable), audit-report.json (machine JSON)
""",
            attachmentsPattern: 'audit.log,audit-report.json',
            compressLog: true
          )
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'audit-report.json,coverage/**,*.log', allowEmptyArchive: true
    }
    success {
      emailext(
        to: 'ayadiscord123123@gmail.com',
        subject: "${env.JOB_NAME} #${env.BUILD_NUMBER} SUCCESS",
        body: "Build URL: ${env.BUILD_URL}",
        compressLog: true
      )
    }
    failure {
      emailext(
        to: 'ayadiscord123123@gmail.com',
        subject: "${env.JOB_NAME} #${env.BUILD_NUMBER} FAILED",
        body: "Build URL: ${env.BUILD_URL}",
        compressLog: true
      )
    }
  }
}
