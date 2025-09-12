pipeline {
  agent any
  tools { nodejs 'Node 18 LTS' } // matches the name you configured in Tools

  options { timestamps(); ansiColor('xterm') }

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
        // Use npm ci if lockfile exists; else fallback to install
        bat 'IF EXIST package-lock.json (npm ci) ELSE (npm install)'
      }
    }

    stage('Run Tests') {
      steps {
        // Continue even if tests fail (per task brief)
        bat 'npm test || exit /b 0'
      }
      post {
        always {
          // If the project emits JUnit XMLs, they’ll be picked up; ok if none
          junit testResults: 'reports/junit/*.xml', allowEmptyResults: true
        }
      }
    }

    stage('Generate Coverage Report') {
      steps {
        bat 'npm run coverage || exit /b 0'
      }
      post {
        always {
          // Archive coverage artifacts if present
          archiveArtifacts artifacts: 'coverage/**/*, .nyc_output/**/*', allowEmptyArchive: true
        }
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        // Human-readable
        bat 'npm audit || exit /b 0'
        // Machine-readable evidence
        bat 'npm audit --json > audit-report.json || exit /b 0'
      }
      post {
        always {
          archiveArtifacts artifacts: 'audit-report.json', allowEmptyArchive: true
        }
      }
    }
  }
}
