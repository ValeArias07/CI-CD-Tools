def qualityGateValidation(qg) {
  if (qg.status != 'OK') {
    return true
  }
  return false
}

pipeline {
  agent any

  tools {
      nodejs 'nodejs'
  }

  environment {
      PROJECT_ROOT = '/scrips'
      REGISTRY = 'alejova2023/icesi-health-back'
  }

  stages {
       stage('Buid'){
            steps {
                checkout scmGit(
                branches: [[name: 'main']],
                userRemoteConfigs: [[url: 'https://github.com/ValeArias07/icesiHealthApp-backend']])
            }
        }

      stage('Install dependencies') {
        steps {
          sh 'npm --version'
          sh "npm install"
        }
      }
      

      stage('Generate coverage report') {
        steps {
          sh " npm run coverage"
        }
      }

      stage('scan') {
          environment {
            // Previously defined in the Jenkins "Global Tool Configuration"
            scannerHome = tool 'sonar-scanner'
          }
          steps {
            // "sonarqube" is the server configured in "Configure System"
            withSonarQubeEnv('sonarqube') {
              // Execute the SonarQube scanner with desired flags
              sh "${scannerHome}/bin/sonar-scanner \
                          -Dsonar.projectKey=IcesiHealth-Back:Test \
                          -Dsonar.projectName=IcesiHealth-Back \
                          -Dsonar.projectVersion=0.0.${BUILD_NUMBER} \
                          -Dsonar.host.url=http://172.173.181.86:9000 \
                          -Dsonar.sources=./app.js \
                          -Dsonar.login=admin \
                          -Dsonar.password=admin \
                          -Dsonar.tests=./test \
                          -Dsonar.javascript.lcov.reportPaths=./coverage/lcov.info"
            }
            timeout(time: 3, unit: 'MINUTES') {
              // In case of SonarQube failure or direct timeout exceed, stop Pipeline
              waitForQualityGate abortPipeline: qualityGateValidation(waitForQualityGate())
            }
          }
      }

      /**
      stage('Build docker-image') {
          steps {
            sh "docker build -t ${REGISTRY}:${BUILD_NUMBER} . "
          }
        }
        stage('Deploy docker-image') {
          steps {
            sh 'docker login'
            sh "docker push alejova2023/icesi-health-back:${BUILD_NUMBER}"
          }
        }
      **/
  }
}