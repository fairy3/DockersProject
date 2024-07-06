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
        NEXUS_URL = "172.24.216.163:8081"
        NEXUS_REPOSITORY = "my-docker-repo"
        NEXUS_CREDENTIALS_ID = "nexus"
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


        stage('Snyk Container Test') {
            steps {
                script {
                    // Test Docker image for vulnerabilities
                    withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                        sh 'snyk auth ${SNYK_TOKEN}'
                        sh 'snyk container test ${APP_IMAGE_NAME}:latest --policy-path=.snyk'                        
                }
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


      stage('Login to Nexus Repository') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${NEXUS_CREDENTIALS_ID}", passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                        sh "docker login -u ${USERNAME} -p ${PASSWORD} http://${NEXUS_URL}/repository/${NEXUS_REPOSITORY}"
                    }
                }
            }
      }

      stage('Tag and Push To Nexus') {
            steps {
                script {
                sh '''
                    docker tag ${DOCKER_USERNAME}/${APP_IMAGE_NAME}:${IMAGE_TAG} ${APP_IMAGE_NAME}:${IMAGE_TAG}
                    docker.image("${APP_IMAGE_NAME}:${IMAGE_TAG}").push()
                    docker tag ${DOCKER_USERNAME}/${WEB_IMAGE_NAME}:${IMAGE_TAG} ${WEB_IMAGE_NAME}:${IMAGE_TAG}
                    docker.image("${WEB_IMAGE_NAME}:${IMAGE_TAG}").push()
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

