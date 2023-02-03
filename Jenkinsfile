pipeline {
    environment {
        IMAGE_NAME = "alpinehelloworld"
        APP_EXPOSED_PORT = "80"
        IMAGE_TAG = "latest"
        STAGING = "dajaj-staging"
        PRODUCTION = "dajaj-prod"
        DOCKERHUB_ID = "dajaj"
        DOCKERHUB_PASSWORD = credentials('DockerHub')
        APP_NAME = "dajaj"
        API_ENDPOINT = "34.203.201.141:1993"
        APP_ENDPOINT = "34.203.201.141:9000"
        INTERNAL_PORT = "80"
        EXTERNAL_PORT = "9000"
        CONTAINER_IMAGE = "${DOCKERHUB_ID}/${IMAGE_NAME}:${IMAGE_TAG}"
    }
    agent none
    stages {
       stage('Build image') {
           agent any
           steps {
              script {
                sh 'docker build -t ${CONTAINER_IMAGE} .'
              }
           }
       }

      stage ('Login and Push Image on docker hub') {
          agent any
          steps {
             script {
               sh '''
                  echo $DOCKERHUB_PASSWORD_PSW | docker login -u $DOCKERHUB_PASSWORD_USR --password-stdin
                  docker push ${CONTAINER_IMAGE}
               '''
             }
          }
      }

      stage('Deploy app') {
        agent any
        steps {
            script {
                sh """
                echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json 
                curl -v -X POST http://${API_ENDPOINT}/prod -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
                """
            }
            }
     
     }
  }
  post {
       success {
         slackSend (color: '#00FF00', message: "AURELE - SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) - URL => http://${APP_ENDPOINT}")
         }
      failure {
            slackSend (color: '#FF0000', message: "AURELE - FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
          }
    }
}