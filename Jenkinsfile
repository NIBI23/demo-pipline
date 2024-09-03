pipeline {
    agent any
    environment {
        STAGING_SERVER = '65.1.134.74'
        DOCKER_IMAGE = 'app:v1'
    }
    stages {
        stage('Build') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    // Run the Docker container for testing
                    sh "docker run -d --name flask-app-test -p 5000:5000 ${DOCKER_IMAGE}"
                    sleep 10
                    // Test the application by curling the local endpoint
                    sh "curl http://localhost:5000"
                    // Stop and remove the test container
                    sh "docker stop flask-app-test && docker rm flask-app-test"
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    // Use SSH credentials to deploy to the staging server
                    withCredentials([sshUserPrivateKey(credentialsId: 'staging-ssh-key', keyFileVariable: 'SSH_KEY_PATH', usernameVariable: 'SSH_USER')]) {
                        sh """
                        ssh -i ${SSH_KEY_PATH} -o StrictHostKeyChecking=no ${SSH_USER}@${STAGING_SERVER} << EOF
                            docker pull ${DOCKER_IMAGE}
                            docker stop flask-app || true
                            docker rm flask-app || true
                            docker run -d --name flask-app -p 5000:5000 ${DOCKER_IMAGE}
                        EOF
                        """
                    }
                }
            }
        }
    }
}

