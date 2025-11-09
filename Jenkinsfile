pipeline {
    agent any

    environment {
        COMPOSE_PROJECT_NAME = "jenkins_ci_app"
        VITE_API = "http://3.110.158.80:4000"   // ✅ Inject your frontend API endpoint here
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
                echo '🚀 Building and starting containers (no cache)...'
                sh '''
                    # Export VITE_API so Docker sees it
                    export VITE_API=${VITE_API}

                    # Rebuild everything without cache to pick up new .env and environment vars
                    docker compose build --no-cache
                    docker compose up -d
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
                    echo "⏳ Waiting up to 60s for backend and frontend to respond..."
                    for i in {1..12}; do
                      if curl -s http://localhost:4000 >/dev/null 2>&1; then
                        echo "✅ Backend is responding on port 4000"
                        break
                      else
                        echo "⏳ Waiting for backend... ($i/12)"
                        sleep 5
                      fi
                    done

                    for i in {1..12}; do
                      if curl -s http://localhost:8085 >/dev/null 2>&1; then
                        echo "✅ Frontend is responding on port 8085"
                        break
                      else
                        echo "⏳ Waiting for frontend... ($i/12)"
                        sleep 5
                      fi
                    done

                    echo "🧾 Backend logs (last 20 lines):"
                    docker logs backend_ci --tail 20 || true

                    echo "🧾 Frontend logs (last 20 lines):"
                    docker logs frontend_ci --tail 20 || true
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
