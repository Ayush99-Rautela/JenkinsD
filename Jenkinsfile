pipeline {
    agent any

    tools {
        maven 'Maven_3.9.11'
    }

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
                    echo 🔐 Copying PEM file to temporary location...
                    set PEM_PATH=C:\\Users\\%USERNAME%\\jenkins_key.pem
                    copy "%EC2_PEM%" "%PEM_PATH%"

                    echo ✅ Setting read-only permissions on PEM file...
                    icacls "%PEM_PATH%" /inheritance:r /grant:r "%USERNAME%:R"

                    echo 📁 Creating remote directory on EC2...
                    wsl ssh -i "$(wslpath '%PEM_PATH%')" -o StrictHostKeyChecking=no %REMOTE_USER%@%REMOTE_HOST% "mkdir -p %REMOTE_DIR%"

                    echo 🚚 Copying JAR and Docker files...
                    wsl scp -i "$(wslpath '%PEM_PATH%')" -o StrictHostKeyChecking=no Authentication/target/*.jar %REMOTE_USER%@%REMOTE_HOST%:%REMOTE_DIR%/app.jar
                    wsl scp -i "$(wslpath '%PEM_PATH%')" -o StrictHostKeyChecking=no docker-compose.yml %REMOTE_USER%@%REMOTE_HOST%:%REMOTE_DIR%/
                    wsl scp -i "$(wslpath '%PEM_PATH%')" -o StrictHostKeyChecking=no Authentication/Dockerfile %REMOTE_USER%@%REMOTE_HOST%:%REMOTE_DIR%/
                    '''
                }
            }
        }

        stage('Run Docker Compose on EC2') {
            steps {
                echo "🐳 Running docker-compose on EC2..."
                withCredentials([file(credentialsId: 'ec2-ssh-key', variable: 'EC2_PEM')]) {
                    bat '''
                    set PEM_PATH=C:\\Users\\%USERNAME%\\jenkins_key.pem

                    echo 🚀 Running docker-compose remotely...
                    wsl ssh -i "$(wslpath '%PEM_PATH%')" -o StrictHostKeyChecking=no %REMOTE_USER%@%REMOTE_HOST% "cd %REMOTE_DIR% && docker-compose down && docker-compose up --build -d"
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
