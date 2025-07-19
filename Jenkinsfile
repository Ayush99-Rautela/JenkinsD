pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = "spring-auth-app"
        COMPOSE_PROJECT_NAME = "auth-app-deployment"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Cloning the repository..."
                checkout scm
            }
        }

        stage('Build with Maven') {
            steps {
                dir('Authentication') {
                    echo "Building the project with Maven..."
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image from Authentication..."
                sh 'docker build -t $DOCKER_IMAGE_NAME ./Authentication'
            }
        }

        stage('Compose Up') {
            steps {
                echo "Deploying using docker-compose..."
                sh 'docker-compose down'
                sh 'docker-compose up --build -d'
            }
        }
    }

    post {
        success {
            echo '✅ Build and deployment successful!'
        }
        failure {
            echo '❌ Build or deployment failed!'
        }
    }
}
