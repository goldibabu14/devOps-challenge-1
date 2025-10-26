pipeline {
    agent any
    
    environment {
        // Docker Hub credentials - Add these in Jenkins Credentials
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        // Docker image name - Update with your DockerHub username
        DOCKER_IMAGE = 'goldibabu/flask-devops-app'
        // Docker image tag
        IMAGE_TAG = "${BUILD_NUMBER}"
        // Container name for deployment
        CONTAINER_NAME = 'flask-app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from repository...'
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building Docker image...'
                script {
                    // Build the Docker image
                    sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
                    sh "docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running tests...'
                script {
                    // Run tests inside a temporary container
                    sh """
                        docker run --rm ${DOCKER_IMAGE}:${IMAGE_TAG} \
                        sh -c 'pip install pytest pytest-cov && pytest test/ -v'
                    """
                }
            }
        }
        
        stage('Push') {
            steps {
                echo 'Pushing Docker image to DockerHub...'
                script {
                    // Login to DockerHub
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                    
                    // Push both tagged and latest version
                    sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying container locally...'
                script {
                    // Stop and remove existing container if it exists
                    sh """
                        docker stop ${CONTAINER_NAME} || true
                        docker rm ${CONTAINER_NAME} || true
                    """
                    
                    // Run the new container
                    sh """
                        docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p 5000:5000 \
                        ${DOCKER_IMAGE}:${IMAGE_TAG}
                    """
                    
                    // Wait a few seconds for the container to start
                    sh 'sleep 5'
                    
                    // Verify the container is running
                    sh "docker ps | grep ${CONTAINER_NAME}"
                }
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up...'
            // Logout from DockerHub
            sh 'docker logout'
        }
        success {
            echo 'Pipeline completed successfully!'
            echo "Application deployed and running on http://localhost:5000"
        }
        failure {
            echo 'Pipeline failed!'
            // Clean up on failure
            sh """
                docker stop ${CONTAINER_NAME} || true
                docker rm ${CONTAINER_NAME} || true
            """
        }
    }
}
