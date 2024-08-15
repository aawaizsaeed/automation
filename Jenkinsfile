pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }

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
                    sh "docker stop ${CONTAINER_NAME} || true"
                    sh "docker rm ${CONTAINER_NAME} || true"
                    sh "docker run -d --name ${CONTAINER_NAME} -p 8000:5000 ${IMAGE_NAME}:${imageTag}"
                }
            }
        }
        stage('Deploy') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'DOCKER_REGISTRY_CREDS', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
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
        stage('Deploy to K8s') {
            steps {
                sshagent(credentials: ['k8s']) {
                    script {     
                        echo "Copying Kubernetes deployment file to remote machine"
                        sh "scp -v -o StrictHostKeyChecking=no deployment.yaml kubemaster@192.168.95.155:/tmp/"

                        echo "Applying Kubernetes configuration"
                        try {
                            sh "ssh kubemaster@192.168.95.155 kubectl apply -f ."
                        } catch (Exception e) {
                            echo "Failed to apply configuration. Creating resources instead."
                            sh "ssh kubemaster@192.168.95.155 kubectl create -f ."
                        }
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
