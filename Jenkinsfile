pipeline {

    agent any

    environment {
        PROJECT_DIR = "/home/ubuntu/Project-Management-Web-Application-Template"
        EC2_PUBLIC_IP = "13.126.20.255"
        VITE_API_URL = "http://13.126.20.255:8000"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "======================================"
                echo "CHECKOUT SOURCE CODE"
                echo "======================================"

                checkout scm
            }
        }

        stage('Verify Project') {
            steps {
                echo "======================================"
                echo "VERIFY PROJECT"
                echo "======================================"

                sh '''
                    set -e

                    echo "Jenkins Workspace:"
                    echo "${WORKSPACE}"

                    echo ""
                    echo "Checking required files..."

                    test -f Dockerfile
                    test -f docker-compose.yml
                    test -f package.json
                    test -d src

                    test -d backend
                    test -f backend/Dockerfile
                    test -f backend/requirements.txt

                    echo ""
                    echo "All required files are available."
                '''
            }
        }

        stage('Copy Project') {
            steps {
                echo "======================================"
                echo "COPY PROJECT"
                echo "======================================"

                sh '''
                    set -e

                    echo "Jenkins Workspace:"
                    echo "${WORKSPACE}"

                    echo "Deployment Directory:"
                    echo "${PROJECT_DIR}"

                    echo "======================================"

                    sudo mkdir -p "${PROJECT_DIR}"

                    # Copy project without deleting existing files
                    sudo cp -r "${WORKSPACE}/." "{PROJECT_DIR}/"

                    echo ""
                    echo "Project copied successfully."

                    echo ""
                    echo "===== Project Files ====="

                    sudo ls -la "${PROJECT_DIR}"
                '''
            }
        }

        stage('Create Environment File') {
            steps {
                echo "======================================"
                echo "CREATE ENVIRONMENT FILE"
                echo "======================================"

                sh '''
                    set -e

                    cd "${PROJECT_DIR}"

                    cat > .env <<EOF
POSTGRES_DB=projectflow_db
POSTGRES_USER=projectflow_user
POSTGRES_PASSWORD=projectflow123

DATABASE_URL=postgresql://projectflow_user:projectflow123@db:5432/projectflow_db

BACKEND_PORT=8000
FRONTEND_PORT=3000

CORS_ORIGINS=http://${EC2_PUBLIC_IP}:3000
VITE_API_URL=${VITE_API_URL}
EOF

                    echo ""
                    echo "Environment file created."

                    echo ""
                    echo "===== Environment Configuration ====="

                    cat .env
                '''
            }
        }

        stage('Stop Old Containers') {
            steps {
                echo "======================================"
                echo "STOP OLD CONTAINERS"
                echo "======================================"

                sh '''
                    set +e

                    cd "${PROJECT_DIR}"

                    docker compose down

                    echo "Old containers stopped."

                    exit 0
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                echo "======================================"
                echo "BUILD DOCKER IMAGES"
                echo "======================================"

                sh '''
                    set -e

                    cd "${PROJECT_DIR}"

                    docker compose build --no-cache

                    echo ""
                    echo "Docker images built successfully."
                '''
            }
        }

        stage('Deploy Containers') {
            steps {
                echo "======================================"
                echo "DEPLOY CONTAINERS"
                echo "======================================"

                sh '''
                    set -e

                    cd "${PROJECT_DIR}"

                    docker compose up -d

                    echo ""
                    echo "Containers started successfully."
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "======================================"
                echo "VERIFY DEPLOYMENT"
                echo "======================================"

                sh '''
                    set -e

                    cd "${PROJECT_DIR}"

                    echo "Waiting for services..."
                    sleep 20

                    echo ""
                    echo "======================================"
                    echo "DOCKER COMPOSE STATUS"
                    echo "======================================"

                    docker compose ps

                    echo ""
                    echo "======================================"
                    echo "RUNNING CONTAINERS"
                    echo "======================================"

                    docker ps

                    echo ""
                    echo "======================================"
                    echo "DATABASE HEALTH CHECK"
                    echo "======================================"

                    docker compose ps db | grep -q "healthy"

                    echo "PostgreSQL is healthy."

                    echo ""
                    echo "======================================"
                    echo "BACKEND HEALTH CHECK"
                    echo "======================================"

                    curl -f http://localhost:8000/docs > /dev/null

                    echo "Backend is healthy."

                    echo ""
                    echo "======================================"
                    echo "FRONTEND HEALTH CHECK"
                    echo "======================================"

                    curl -f http://localhost:3000 > /dev/null

                    echo "Frontend is healthy."

                    echo ""
                    echo "======================================"
                    echo "APPLICATION DEPLOYED SUCCESSFULLY"
                    echo "======================================"
                '''
            }
        }

        stage('Show Application URLs') {
            steps {
                echo "======================================"
                echo "APPLICATION URLS"
                echo "======================================"

                echo "Frontend:"
                echo "http://${EC2_PUBLIC_IP}:3000"

                echo ""

                echo "Backend Swagger:"
                echo "http://${EC2_PUBLIC_IP}:8000/docs"

                echo ""

                echo "Backend API:"
                echo "http://${EC2_PUBLIC_IP}:8000"
            }
        }
    }

    post {

        success {
            echo "======================================"
            echo "SUCCESS"
            echo "======================================"

            echo "Project Management Web Application"
            echo "deployed successfully!"
        }

        failure {
            echo "======================================"
            echo "DEPLOYMENT FAILED"
            echo "======================================"

            sh '''
                set +e

                cd "${PROJECT_DIR}"

                echo ""
                echo "======================================"
                echo "DOCKER COMPOSE STATUS"
                echo "======================================"

                docker compose ps

                echo ""
                echo "======================================"
                echo "BACKEND LOGS"
                echo "======================================"

                docker compose logs --tail=100 backend

                echo ""
                echo "======================================"
                echo "FRONTEND LOGS"
                echo "======================================"

                docker compose logs --tail=100 frontend

                echo ""
                echo "======================================"
                echo "DATABASE LOGS"
                echo "======================================"

                docker compose logs --tail=100 db
            '''
        }

        always {
            echo "Jenkins pipeline execution completed."

            sh '''
                docker image prune -f || true
            '''
        }
    }
}
