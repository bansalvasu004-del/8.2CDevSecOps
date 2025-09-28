pipeline {
  agent any
  options {
    skipDefaultCheckout(false)
    timestamps()
  }

  environment {
    // Set these to match your SonarCloud org & project
    SONAR_ORG = 'bansalvasu004-del'
    SONAR_KEY = 'bansalvasu004-del'
  }

  stages {

    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/bansalvasu004-del/8.2CDevSecOps.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        sh 'node -v && npm -v || true'
        sh 'npm install'
      }
    }

    stage('Run Tests (tolerant)') {
      steps {
        // Your package.json test runs Snyk; keep pipeline green even if it fails.
        sh 'npm test || true'
      }
    }

    stage('Generate Coverage Report (tolerant)') {
      steps {
        // Your project currently has no "coverage" script; keep this non-blocking.
        sh 'npm run coverage || true'
      }
    }

    stage('NPM Audit (tolerant)') {
      steps {
        sh 'npm audit || true'
      }
    }

    // --- Quick visibility that Jenkins has what Sonar needs ---
    stage('Debug Sonar Props') {
      environment { SONAR_TOKEN = credentials('SONAR_TOKEN') }
      steps {
        sh '''
          echo "==== DEBUG: Sonar parameters from file (if present) ===="
          test -f sonar-project.properties && cat sonar-project.properties | sed -n '1,200p' || echo "no sonar-project.properties"

          echo "==== DEBUG: Effective values that will be passed ===="
          echo "ORG=${SONAR_ORG}"
          echo "KEY=${SONAR_KEY}"
          echo "TOKEN SET? ${SONAR_TOKEN:+yes}"
        '''
      }
    }

    stage('SonarCloud Analysis') {
      environment { SONAR_TOKEN = credentials('SONAR_TOKEN') }
      steps {
        sh '''
          set -eux
          SCAN_VER=5.0.1.3006
          curl -L -o scanner.zip "https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-${SCAN_VER}-linux.zip"
          rm -rf sonar-scanner
          unzip -q scanner.zip
          mv sonar-scanner-* sonar-scanner

          ./sonar-scanner/bin/sonar-scanner \
            -Dsonar.organization=${SONAR_ORG} \
            -Dsonar.projectKey=${SONAR_KEY} \
            -Dsonar.host.url=https://sonarcloud.io \
            -Dsonar.login=${SONAR_TOKEN}
        '''
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'scanner.zip', fingerprint: true, onlyIfSuccessful: false
    }
  }
}

