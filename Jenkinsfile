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
                    bat 'mvn clean package -DskipTests'
                }
            }
        }

        stage('SCP Files to EC2') {
            steps {
                echo "📤 Copying files to EC2..."
                withCredentials([file(credentialsId: 'ec2-ssh-key', variable: 'EC2_PEM')]) {
                    bat '''
                    echo Setting permissions for SSH key...
                    icacls "%EC2_PEM%" /inheritance:r /grant:r "%USERNAME%:R"

                    echo Creating remote directory...
                    bash -c "ssh -o StrictHostKeyChecking=no -i %EC2_PEM% %REMOTE_USER%@%REMOTE_HOST% 'mkdir -p %REMOTE_DIR%'"

                    echo Copying JAR and Docker files...
                    bash -c "scp -o StrictHostKeyChecking=no -i %EC2_PEM% Authentication/target/*.jar %REMOTE_USER%@%REMOTE_HOST%:%REMOTE_DIR%/app.jar"
                    bash -c "scp -o StrictHostKeyChecking=no -i %EC2_PEM% docker-compose.yml %REMOTE_USER%@%REMOTE_HOST%:%REMOTE_DIR%/"
                    bash -c "scp -o StrictHostKeyChecking=no -i %EC2_PEM% Authentication/Dockerfile %REMOTE_USER%@%REMOTE_HOST%:%REMOTE_DIR%/"
                    '''
                }
            }
        }

        stage('Run Docker Compose on EC2') {
            steps {
                echo "🐳 Running docker-compose on EC2..."
                withCredentials([file(credentialsId: 'ec2-ssh-key', variable: 'EC2_PEM')]) {
                    bat '''
                    echo Connecting to EC2 and running Docker Compose...
                    bash -c "ssh -o StrictHostKeyChecking=no -i %EC2_PEM% %REMOTE_USER%@%REMOTE_HOST% \\"cd %REMOTE_DIR% && docker-compose down && docker-compose up --build -d\\""
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
