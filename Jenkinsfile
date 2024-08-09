pipeline {
  agent any

  stages {
        stage('Checkout') {
            steps {
                script {
                    def branchName = params.BRANCH
                    echo "Checking out branch: ${branchName}"

                    checkout([$class: 'GitSCM',
                        branches: [[name: "*/${branchName}"]],
                        userRemoteConfigs: [[url: "${params.MY_CODE}"]]
                    ])
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    def imageTag = "latest-${env.BUILD_NUMBER}"
                    echo "Building Docker image with tag: ${imageTag}"
                    sh "docker build -t ${params.IMAGE_NAME}:${imageTag} ."
                    sh "docker tag ${params.IMAGE_NAME}:${imageTag} ${params.DOCKER_BFLASK_IMAGE}:${imageTag}"
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    def imageTag = "latest-${env.BUILD_NUMBER}"
                    echo "Running tests on Docker image with tag: ${imageTag}"
                    sh "docker run ${params.IMAGE_NAME}:${imageTag} python -m pytest app/tests/"
                }
            }
        }
        stage('Deploy') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${params.DOCKER_REGISTRY_CREDS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
                    def imageTag = "latest-${env.BUILD_NUMBER}"
                    echo "Pushing Docker image with tag: ${imageTag}"
                    sh "docker push ${params.DOCKER_BFLASK_IMAGE}:${imageTag}"
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
