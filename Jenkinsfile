pipeline {
    agent any

    tools {
        maven 'Maven_3.9.11'
    }

    environment {
        REMOTE_USER = "ubuntu"
        REMOTE_HOST = "13.48.48.60"
        REMOTE_DIR = "/home/ubuntu/auth-app"
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
                    script {
                        def pemPath = "${env.WORKSPACE}\\jenkins_key.pem".replace('/', '\\')
                      bat """
echo 🔐 Copying PEM file to Jenkins workspace...
copy "%EC2_PEM%" "${pemPath}"

echo ✅ Setting permissions on PEM file...
wsl chmod 600 "\$(wslpath '${pemPath}')"

echo 📁 Creating remote directory...
wsl ssh -i "\$(wslpath '${pemPath}')" -o StrictHostKeyChecking=no %REMOTE_USER%@%REMOTE_HOST% "mkdir -p %REMOTE_DIR%"

echo 🚚 Copying JAR and Docker files...
wsl scp -i "\$(wslpath '${pemPath}')" -o StrictHostKeyChecking=no Authentication/target/*.jar %REMOTE_USER%@%REMOTE_HOST%:%REMOTE_DIR%/app.jar
wsl scp -i "\$(wslpath '${pemPath}')" -o StrictHostKeyChecking=no docker-compose.yml %REMOTE_USER%@%REMOTE_HOST%:%REMOTE_DIR%/
wsl scp -i "\$(wslpath '${pemPath}')" -o StrictHostKeyChecking=no Authentication/Dockerfile %REMOTE_USER%@%REMOTE_HOST%:%REMOTE_DIR%/
"""

                    }
                }
            }
        }

        stage('Run Docker Compose on EC2') {
            steps {
                echo "🐳 Running docker-compose on EC2..."
                withCredentials([file(credentialsId: 'ec2-ssh-key', variable: 'EC2_PEM')]) {
                    script {
                        def pemPath = "${env.WORKSPACE}\\jenkins_key.pem".replace('/', '\\')
                        bat """
                        echo 🚀 Running docker-compose remotely...
                        wsl ssh -i \$(wslpath "${pemPath}") -o StrictHostKeyChecking=no %REMOTE_USER%@%REMOTE_HOST% "cd %REMOTE_DIR% && docker-compose down && docker-compose up --build -d"
                        """
                    }
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
