pipeline {
    environment {
        IMAGE_NAME = "${PARAM_IMAGE_NAME}"
        APP_EXPOSED_PORT = "${PARAM_PORT_EXPOSED}"
        APP_NAME = "eazystacker"
        IMAGE_TAG = "v2"
        STAGING = "${APP_NAME}-staging"
        PRODUCTION = "${APP_NAME}-prod"
        DOCKERHUB_ID = "${DOCKERHUB_AUTH_USR}"
        DOCKERHUB_AUTH = credentials('DOCKERHUB_ID')
        STG_API_ENDPOINT = "http://ip10-0-5-5-d7r5q9u57ed0008ln4i0-1993.direct.docker.labs.eazytraining.fr"
        STG_APP_ENDPOINT = "http://ip10-0-5-5-d7r5q9u57ed0008ln4i0-80.direct.docker.labs.eazytraining.fr"
        PROD_API_ENDPOINT = "http://ip10-0-5-6-d7r5q9u57ed0008ln4i0-1993.direct.docker.labs.eazytraining.fr"
        PROD_APP_ENDPOINT = "http://ip10-0-5-6-d7r5q9u57ed0008ln4i0-80.direct.docker.labs.eazytraining.fr"
        EXTERNAL_PORT = "${PARAM_PORT_EXPOSED}"
        CONTAINER_IMAGE = "${DOCKERHUB_ID}/${IMAGE_NAME}:${IMAGE_TAG}"
    }

    parameters {
        // booleanParam(name: "RELEASE", defaultValue: false)
        // choice(name: "DEPLOY_TO", choices: ["", "INT", "PRE", "PROD"])
        string(name: 'PARAM_IMAGE_NAME', defaultValue: 'play_stacker', description: 'Image Name')
        string(name: 'PARAM_PORT_EXPOSED', defaultValue: '80', description: 'APP EXPOSED PORT')        
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
                  echo "Clean Environment"
              docker rm -f $IMAGE_NAME || echo "container does not exist"
              docker run --name $IMAGE_NAME -d -p ${APP_EXPOSED_PORT}:80 ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
              sleep 5
              curl http://172.17.0.1:${PORT_EXPOSED} | grep -q "Playbook Stacker"
              '''
             }
          }
       }
       stage('Test image') {
           agent any
           steps {
              script {
                sh '''
                   curl -v 172.17.0.1:$APP_EXPOSED_PORT | grep -q "Playbook Stacker"
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
                   docker login -u $DOCKERHUB_AUTH_USR -p $DOCKERHUB_AUTH_PSW
                   docker push ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
               '''
             }
          }
      }

      stage('STAGING - Deploy app') {
        when {
           expression { GIT_BRANCH == 'origin/main' }
       }
      agent any
      steps {
          script {
            sh """
              echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json 
              curl -v -X POST http://${STG_API_ENDPOINT}/staging -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
            """
          }
        }
     
     }
     stage('PROD - Deploy app') {
       when {
           expression { GIT_BRANCH == 'origin/main' }
       }
     agent any

       steps {
          script {
            sh """
              echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json 
              curl -v -X POST http://${PROD_API_ENDPOINT}/prod -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
            """
          }
       }
     }
  }
}
