pipeline {
  agent any
  options {
        buildDiscarder(logRotator(numToKeepStr: '5'))

  stages {
        stage('Checkout') {
            steps {
                script {
                    def branchName = params.BRANCH
                    echo "Checking out branch: ${branchName}"

                    checkout([$class: 'GitSCM',
                        branches: [[name: "*/${branchName}"]],
                        userRemoteConfigs: [[url: "${MY_CODE}"]]
                    ])
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    def imageTag = "latest-${env.BUILD_NUMBER}"
                    echo "Building Docker image with tag: ${imageTag}"
                    sh "docker build -t ${IMAGE_NAME}:${imageTag} ."
                    sh "docker tag ${IMAGE_NAME}:${imageTag} ${DOCKER_BFLASK_IMAGE}:${imageTag}"
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    def imageTag = "latest-${env.BUILD_NUMBER}"
                    echo "Running tests on Docker image with tag: ${imageTag}"
                    sh "docker run ${IMAGE_NAME}:${imageTag} python -m pytest app/tests/"
                }
            }
        }
        stage('Deploy') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    script {
                      def imageTag = "latest-${env.BUILD_NUMBER}"
                      echo "Logging in to Docker registry"
                      sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
                      echo "Pushing Docker image with tag: ${imageTag}"
                      sh "docker push ${DOCKER_BFLASK_IMAGE}:${imageTag}"
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "Logging out from Docker registry"
            sh 'docker logout'
        }
    }
}
