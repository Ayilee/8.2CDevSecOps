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
        bat 'npm test || exit /b 0'
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        bat 'npm audit || exit /b 0'
        bat 'npm audit --json > audit-report.json || exit /b 0'
      }
    }

    stage('Finalize & Notify') {
      steps {
        script {
          if (fileExists('audit-report.json')) {
            archiveArtifacts artifacts: 'audit-report.json', allowEmptyArchive: true
          } else {
            echo 'No audit-report.json found to archive.'
          }
        }
      }
      post {
        success {
          emailext(
            to: 'ayadiscord123123@gmail.com',
            subject: "${env.JOB_NAME} #${env.BUILD_NUMBER} SUCCESS",
            body: "Build URL: ${env.BUILD_URL}"
          )
        }
        failure {
          emailext(
            to: 'ayadiscord123123@gmail.com',
            subject: "${env.JOB_NAME} #${env.BUILD_NUMBER} FAILED",
            body: "Build URL: ${env.BUILD_URL}"
          )
        }
        always {
          archiveArtifacts artifacts: 'coverage/**', allowEmptyArchive: true
        }
      }
    }
  }
}
