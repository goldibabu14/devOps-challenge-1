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
                    bat "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
                    bat "docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running tests...'
                script {
                    // Run tests inside a temporary container with required env vars
                    // Skip database connectivity test as Redis/Postgres are not available in CI
                    bat "docker run --rm -e SECRET_KEY=test-secret-key -e DATABASE_URL=sqlite:///test.db ${DOCKER_IMAGE}:${IMAGE_TAG} sh -c \"pip install pytest pytest-cov && pytest test/ -v -k 'not test_up_databases'\""
                }
            }
        }
        
        stage('Push') {
            steps {
                echo 'Pushing Docker image to DockerHub...'
                script {
                    // Login to DockerHub
                    bat "docker login -u %DOCKERHUB_CREDENTIALS_USR% -p %DOCKERHUB_CREDENTIALS_PSW%"
                    
                    // Push both tagged and latest version
                    bat "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                    bat "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying container locally...'
                script {
                    // Stop and remove existing container if it exists
                    bat "docker stop ${CONTAINER_NAME} || exit 0"
                    bat "docker rm ${CONTAINER_NAME} || exit 0"
                    
                    // Run the new container
                    bat "docker run -d --name ${CONTAINER_NAME} -p 5000:5000 ${DOCKER_IMAGE}:${IMAGE_TAG}"
                    
                    // Wait a few seconds for the container to start
                    bat "timeout /t 5 /nobreak"
                    
                    // Verify the container is running
                    bat "docker ps | findstr ${CONTAINER_NAME}"
                }
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up...'
            // Logout from DockerHub
            bat 'docker logout || exit 0'
        }
        success {
            echo 'Pipeline completed successfully!'
            echo "Application deployed and running on http://localhost:5000"
        }
        failure {
            echo 'Pipeline failed!'
            // Clean up on failure
            bat "docker stop ${CONTAINER_NAME} || exit 0"
            bat "docker rm ${CONTAINER_NAME} || exit 0"
        }
    }
}
