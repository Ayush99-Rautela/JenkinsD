pipeline {
    agent any

    environment {
        REMOTE_USER = "ubuntu"
        REMOTE_HOST = "your.ec2.ip.address"
        REMOTE_DIR = "/home/ubuntu/auth-app"
        GIT_REPO = "https://github.com/your-username/your-repo.git"
    }

    stages {
        stage('Deploy to EC2') {
            steps {
                withCredentials([file(credentialsId: 'ec2-ssh-key', variable: 'EC2_PEM')]) {
                    sh '''
                    chmod 400 $EC2_PEM

                    ssh -o StrictHostKeyChecking=no -i $EC2_PEM $REMOTE_USER@$REMOTE_HOST '
                        echo "🔧 Preparing directory..."
                        mkdir -p $REMOTE_DIR
                        cd $REMOTE_DIR

                        echo "📦 Cloning or pulling the repo..."
                        if [ -d ".git" ]; then
                            git pull
                        else
                            git clone $GIT_REPO .
                        fi

                        echo "🛠️ Building with Maven..."
                        cd Authentication
                        mvn clean package -DskipTests
                        cd ..

                        echo "🐳 Docker compose up..."
                        docker-compose down
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
