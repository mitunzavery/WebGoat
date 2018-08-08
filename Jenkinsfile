pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"

        input 'Build with maven?'

        sh '''
                    env
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                    echo "Build Number = ${BUILD_ID}"
                    mvn -B install -Dmaven.test.skip=true
                '''
        input 'Continue?'
      }
    }
    stage('Scan App - Build Container') {
      parallel {
        stage('IQ-BOM') {
          steps {
            input 'Scan with IQ at Build?'

            nexusPolicyEvaluation(iqApplication: 'webgoat8', iqStage: 'build', iqScanPatterns: [[scanPattern: '']])
          }
        }
        stage('Static Analysis') {
          steps {
            echo '...run SonarQube or other SAST tools here'
          }
        }
        stage('Build Container') {
          steps {
    
            input 'Build with docker?'

            sh '''cd webgoat-server
                  docker build -t webgoat/webgoat-8.0-${BUILD_ID} .
                    '''
          }
        }
      }
    }
    stage('Test Container') {
      parallel {
        stage('Test Container') {
          steps {
            catchError() {
              echo 'Test Container here'
            }

          }
        }
        stage('IQ-Scan Container') {
          steps {
            input 'Scan with IQ at Stage-Release?'

            sh 'docker save webgoat/webgoat-8.0-${BUILD_ID} -o $WORKSPACE/webgoat.tar'
            nexusPolicyEvaluation(iqStage: 'stage-release', iqApplication: 'webgoat8')
          }
        }
      }
    }
    stage('Publish Container') {
      when {
        branch 'develop'
      }
      steps {
        input 'Push to Nexus Repo?'

        sh '''
                    docker tag webgoat/webgoat-8.0-${BUILD_ID} local-mike:19447/webgoat/webgoat-8.0-${BUILD_ID}:8.0-${BUILD_ID}
                    docker push local-mike:19447/webgoat/webgoat-8.0-${BUILD_ID}
                '''
      }
    }
    stage('Create Tag') {
      when {
        branch 'develop'
      }
      steps {
        input 'Create tag in Nexus Repo?'

        createTag nexusInstanceId: 'nexus3-demo', tagName: 'RBCDemoJenkinsfile'
      }
    }
  }
  tools {
    maven 'Maven 3.5.3 - Local Install'
  }
}