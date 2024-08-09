pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
          script {
                    def imageTag = "latest-${env.BUILD_NUMBER}"
                    sh 'docker build -t ${IMAGE_NAME} .'
                    sh 'docker tag my-flask-app ${DOCKER_BFLASK_IMAGE}:${imageTag}'
          }
       }
    }
    stage('Test') {
      steps {
        sh 'docker run ${IMAGE_NAME}:${imageTag} python -m pytest app/tests/'
      }
    }
    stage('Deploy') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
          sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
          sh 'docker push $DOCKER_BFLASK_IMAGE'
        }
      }
    }
  }
  post {
    always {
      sh 'docker logout'
    }
  }
}
