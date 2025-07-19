pipeline {
    agent any

    environment {
        REMOTE_USER = "ubuntu"
        REMOTE_HOST = "13.48.48.60"
        REMOTE_DIR = "/home/ubuntu/auth-app"
        DOCKER_IMAGE_NAME = "spring-auth-app"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "📥 Cloning the repository..."
                checkout scm
            }
        }

        stage('Build JAR') {
            steps {
                dir('Authentication') {
                    echo "🛠️ Building Spring Boot application..."
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('SCP Files to EC2') {
            steps {
                echo "📤 Copying files to EC2..."
                withCredentials([file(credentialsId: 'ec2-ssh-key', variable: 'EC2_PEM')]) {
                    sh '''
                        chmod 400 "$EC2_PEM"

                        echo "📁 Creating remote directory..."
                        ssh -o StrictHostKeyChecking=no -i "$EC2_PEM" "$REMOTE_USER@$REMOTE_HOST" "mkdir -p '$REMOTE_DIR'"

                        echo "🚚 Transferring JAR file..."
                        scp -o StrictHostKeyChecking=no -i "$EC2_PEM" Authentication/target/*.jar "$REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR/app.jar"

                        echo "📦 Transferring Dockerfile..."
                        scp -o StrictHostKeyChecking=no -i "$EC2_PEM" Authentication/Dockerfile "$REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR/Dockerfile"

                        echo "⚙️ Transferring docker-compose.yml..."
                        scp -o StrictHostKeyChecking=no -i "$EC2_PEM" docker-compose.yml "$REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR/docker-compose.yml"
                    '''
                }
            }
        }

        stage('Deploy via Docker Compose on EC2') {
            steps {
                echo "🐳 Running Docker Compose on EC2..."
                withCredentials([file(credentialsId: 'ec2-ssh-key', variable: 'EC2_PEM')]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -i "$EC2_PEM" "$REMOTE_USER@$REMOTE_HOST" '
                            cd "$REMOTE_DIR"
                            echo "🧼 Stopping old containers..."
                            docker-compose down
                            echo "🚀 Starting new containers..."
                            docker-compose up --build -d
                        '
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Remote deployment successful!'
        }
        failure {
            echo '❌ Remote deployment failed!'
        }
    }
}
