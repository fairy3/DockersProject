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
    }

    stages {

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
                  docker tag ${APP_IMAGE_NAME}:latest ${DOCKER_USERNAME}/${APP_IMAGE_NAME}:${IMAGE_TAG}
                  docker push ${DOCKER_USERNAME}/${APP_IMAGE_NAME}:${IMAGE_TAG}
                  docker tag ${WEB_IMAGE_NAME}:latest ${DOCKER_USERNAME}/${WEB_IMAGE_NAME}:${IMAGE_TAG}
                  docker push ${DOCKER_USERNAME}/${WEB_IMAGE_NAME}:${IMAGE_TAG}
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

