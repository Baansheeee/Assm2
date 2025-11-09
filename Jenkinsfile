pipeline {
    agent any

    environment {
        COMPOSE_PROJECT_NAME = "jenkins_ci_app"
    }

    stages {
        stage('Checkout Code from GitHub') {
            steps {
                echo '📦 Cloning repository...'
                git branch: 'jenkins', url: 'https://github.com/Baansheeee/Assm2.git'
            }
        }

        stage('Set Up Docker Environment') {
            steps {
                echo '⚙️ Checking Docker and Docker Compose installation...'
                sh '''
                    docker --version
                    if docker compose version >/dev/null 2>&1; then
                        echo "✅ Docker Compose v2 detected"
                    else
                        echo "⚠️ Installing Docker Compose plugin..."
                        apt-get update -y && apt-get install -y docker-compose-plugin
                    fi
                    docker compose version
                '''
            }
        }

        stage('Clean Previous Containers') {
            steps {
                echo '🧹 Cleaning up old containers and volumes...'
                sh '''
                    echo "🔍 Removing existing CI containers if any..."
                    docker ps -aq --filter "name=_ci" | xargs -r docker rm -f || true

                    echo "🔍 Bringing down any docker-compose project..."
                    docker compose down --volumes --remove-orphans || true

                    echo "🧼 Pruning unused images, networks, and volumes..."
                    docker system prune -af || true
                    docker volume prune -f || true
                '''
            }
        }

        stage('Build and Run Application') {
            steps {
                echo '🚀 Building and starting containers...'
                sh '''
                    docker compose up -d --build
                '''
            }
        }

        stage('Verify Containers') {
            steps {
                echo '🔍 Listing running containers...'
                sh 'docker ps'
            }
        }

        stage('Application Health Check') {
            steps {
                echo '🩺 Checking if backend and frontend are accessible...'
                sh '''
                    sleep 10
                    echo "Backend (port 4000):"
                    curl -I http://localhost:4000 || true
                    echo "Frontend (port 8085):"
                    curl -I http://localhost:8085 || true
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Build pipeline completed successfully! Website should be live on EC2.'
        }
        failure {
            echo '❌ Build pipeline failed. Check Jenkins logs for details.'
        }
    }
}
