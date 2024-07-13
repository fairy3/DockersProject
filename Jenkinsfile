@Library("my-shared-library") _
pipeline {
    agent any

     options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(daysToKeepStr: '30'))
        timestamps()
    }

    environment {
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
        stage('Hello') {
           steps {
            wrap([$class: 'BuildUser']) {
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

        stage('Unit Test') {
            steps {
                echo "Running unittests"
                sh '''
                    pip install pytest unittest
                    python -m pytest --junitxml results.xml /test
                '''
            }
        }

        stage('Lint') {
            steps {
                echo "Pylint running"
                //sh 'pylint src/'
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

      stage('Deploy') {
            steps {
              echo "Deploying to k8s cluster"
            }
        }
    }

    post {
        always {
            // Clean up the workspace!
            cleanWs()
        }

        success {
            // Actions that should run only if the pipeline succeeds, e.g., notifications
            mail to: 'team@example.com',
                 subject: "Pipeline Success: ${currentBuild.fullDisplayName}",
                 body: "Good news! The pipeline has succeeded."
        }
        failure {
            // Actions that should run only if the pipeline fails, e.g., notifications
            mail to: 'team@example.com',
                 subject: "Pipeline Failure: ${currentBuild.fullDisplayName}",
                 body: "Unfortunately, the pipeline has failed. Please check the logs."
        }
    }
}

