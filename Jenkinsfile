pipeline {
    agent any

    environment {
        IMAGE_NAME = "frontend-app"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/swap1408/Frontend.git', branch: 'main'
            }
        }

        stage('Install Dependencies') {
            steps {
                // Clean install to avoid peer dependency errors
                sh 'npm ci --legacy-peer-deps'
            }
        }

        stage('Build React App') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Docker Build') {
            steps {
                // Build Docker image locally
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Run Container') {
            steps {
                // Stop and remove existing container if it exists
                sh '''
                    CONTAINER_ID=$(docker ps -aq --filter "name=${IMAGE_NAME}")
                    if [ ! -z "$CONTAINER_ID" ]; then
                        echo "Stopping existing container..."
                        docker stop $CONTAINER_ID || true
                        docker rm $CONTAINER_ID || true
                    fi
                    echo "Starting new container..."
                    docker run -d -p 80:80 --name ${IMAGE_NAME} ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Cleanup') {
            steps {
                // Remove dangling images to save space
                sh 'docker image prune -f'
            }
        }
    }

    post {
        success {
            echo "✅ Frontend build and container run completed successfully!"
        }
        failure {
            echo "❌ Build failed. Check the logs above."
        }
    }
}
