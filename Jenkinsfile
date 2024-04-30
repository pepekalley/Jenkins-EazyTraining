/* import shared library */
@Library('chocoapp-slack-share-library')_

pipeline {
    environment {
        IMAGE_NAME = "staticwebsite"
        APP_CONTAINER_PORT = "5000"
        APP_EXPOSED_PORT = "80"
        IMAGE_TAG = "latest"
        STAGING = "openpepe-staging"
        PRODUCTION = "openpepe-prod"
        DOCKERHUB_ID = "pepekalleydocker"
        DOCKERHUB_PASSWORD = credentials('dockerhub_password')
	GITHUB_ID = "pepekalley"
    }
    agent none
    stages {
       stage('Build image') {
           agent any
           steps {
              script {
                sh 'docker build -t ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG .'
              }
           }
       }
       stage('Run container based on builded image') {
          agent any
          steps {
            script {
              sh '''
                  echo "Cleaning existing container if exist"
                  docker ps -a | grep -i $IMAGE_NAME && docker rm -f $IMAGE_NAME
                  docker run --name $IMAGE_NAME -d -p $APP_EXPOSED_PORT:$APP_CONTAINER_PORT -e PORT=$APP_CONTAINER_PORT ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
                  sleep 5
              '''
             }
          }
       }
       stage('Test image') {
           agent any
           steps {
              script {
                sh '''
                   curl 172.17.0.1 | grep -i "Dimension"
                '''
              }
           }
       }
       stage('Clean container') {
          agent any
          steps {
             script {
               sh '''
                   docker stop $IMAGE_NAME
                   docker rm $IMAGE_NAME
               '''
             }
          }
      }

      stage ('Login and Push Image on docker hub') {
          agent any
          steps {
             script {
               sh '''
                   echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_ID --password-stdin
                   docker push ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
               '''
             }
          }
      }

      stage('Push image in staging and deploy it') {
       /* agent any */
        when {
            expression { GIT_BRANCH == 'origin/main' }
        }
	agent {
        	docker { image 'franela/dind' }
	}

        environment {
            GITHUB_API_KEY = credentials('github_api_key')
        }
        steps {
           script {
             sh '''
                echo $GITHUB_API_KEY | docker login ghcr.io -u $GITHUB_ID --password-stdin
             '''
           }
        }
     }
     stage('Push image in production and deploy it') {
       /* #agent any */
       when {
           expression { GIT_BRANCH == 'origin/main' }
       }
	agent {
        	docker { image 'franela/dind' }
	}
       environment {
           GITHUB_API_KEY = credentials('github_api_key')
       }
       steps {
          script {
            sh '''
               echo $GITHUB_API_KEY | docker login ghcr.io -u $GITHUB_ID --password-stdin
            '''
          }
       }
     }
  }
  post {
       success {
         slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
         }
      failure {
            slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
          }   
    }
}
