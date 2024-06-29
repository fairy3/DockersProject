pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub') // Jenkins credentials ID
        APP_IMAGE_NAME = 'pyton-app-image:latest'
        WEB_IMAGE_NAME = 'web-image:latest'
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from repository
                checkout scm
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


        stage('Tag and push images') {
          steps {
             script {
                withCredentials(
                [usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASS')]){
               sh '''
                  docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASS}
                  docker tag ${APP_IMAGE_NAME}:latest ${DOCKER_USERNAME}/${APP_IMAGE_NAME}:${BUILD_NUMBER}
                  docker push ${DOCKER_USERNAME}/${APP_IMAGE_NAME}:${BUILD_NUMBER}
                  docker tag ${WEB_IMAGE_NAME} ${DOCKER_USERNAME}/${WEB_IMAGE_NAME}:${BUILD_NUMBER}
                  docker push ${DOCKER_USERNAME}/${WEB_IMAGE_NAME}:${BUILD_NUMBER}
               '''
              }
           }
        }
      }
    }


    post {
        always {
            // Clean up the workspace
            cleanWs()
        }
    }
}

