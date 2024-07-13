@Library("my-shared-library") _
pipeline {
    agent any

     options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(daysToKeepStr: '30'))
        timestamps()
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub') // Jenkins credentials ID
        APP_IMAGE_NAME = 'python-app-image'
        WEB_IMAGE_NAME = 'web-image'
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
        BUILD_DATE = new Date().format('yyyyMMdd-HHmmss')
        IMAGE_TAG = "v1.0.0-${BUILD_NUMBER}-${BUILD_DATE}"
        SNYK_TOKEN = credentials('snyk-token')
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "172.24.216.163:8888"
        NEXUS_REPOSITORY = "my-docker-repo"
        NEXUS_CREDENTIALS_ID = "nexus"
    }

    stages {
        stage ('Init user') {
           steps {
                wrap([$class: 'BuildUser']) {
                     echo "BUILD_USER that started this Pipeline: ${BUILD_USER}"
               }
           }
        }

        stage('Hello') {
           steps {

            wrap([$class: 'BuildUser']) {
              echo " Debug build user name: $BUILD_USER, $BUILD_USER_FIRST_NAME, $BUILD_USER_LAST_NAME, $BUILD_USER_ID"
              greet()
              }
           }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image using docker-compose
                    sh 'docker-compose -f ${DOCKER_COMPOSE_FILE} build'
                }
            }
        }

        stage('Snyk login') {
            steps {
                snykLogin('${SNYK_TOKEN}')
            }
        }

        stage('Snyk Container Test') {
            steps {
                script {
                    // Test Docker image for vulnerabilities
                    sh 'snyk container test ${APP_IMAGE_NAME}:latest --policy-path=.snyk'
               }
           }
       }        

       stage('Nexus login') {
            steps {
                nexusLogin("${NEXUS_CREDENTIALS_ID}","${NEXUS_PROTOCOL}","${NEXUS_URL}", "${NEXUS_REPOSITORY}")
            }
       }

      stage('Tag and Push To Nexus') {
            steps {
                script {
                sh '''
                    docker tag ${APP_IMAGE_NAME}:latest ${NEXUS_URL}/${APP_IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${NEXUS_URL}/${APP_IMAGE_NAME}:${IMAGE_TAG}
                    docker tag ${WEB_IMAGE_NAME}:latest ${NEXUS_URL}/${WEB_IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${NEXUS_URL}/${WEB_IMAGE_NAME}:${IMAGE_TAG}
                 '''
                }
            }
      }
    }

    post {
        always {
            // Clean up the workspace!
            cleanWs()
        }
    }
}

