pipeline {
    agent any

    environment {
        IMAGE_NAME = 'netflix-clone'
        DOCKER_HUB_REPO = 'mohdibrahimk/netflix-clone'
        SLACK_WEBHOOK_URL = credentials('slack_webhook_url')
    }

    stages {
        stage('Clone GitHub Repo') {
            steps {
                echo "📥 Cloning repository..."
                git credentialsId: 'git-creds', url: 'https://github.com/MohdKhal/netflix-clone-devsecops.git', branch: 'main'
            }
        }

        stage('Trivy Scan') {
            steps {
                echo "🔍 Running Trivy vulnerability scan..."
                sh '''
                  if ! command -v trivy &> /dev/null; then
                      echo "Installing Trivy..."
                      sudo apt-get update -y && sudo apt-get install wget apt-transport-https gnupg lsb-release -y
                      wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
                      echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
                      sudo apt-get update -y && sudo apt-get install trivy -y
                  fi

                  echo "Running Trivy on Dockerfile..."
                  trivy config .
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                sh 'docker build -t $DOCKER_HUB_REPO:latest .'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                echo "🔎 Scanning Docker image with Trivy..."
                sh 'trivy image --severity HIGH,CRITICAL $DOCKER_HUB_REPO:latest || true'
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo "📦 Pushing image to DockerHub..."
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                      docker push $DOCKER_HUB_REPO:latest
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "🧹 Cleaning up unused Docker images..."
            sh 'docker system prune -af || true'
        }

        success {
            echo "✅ Pipeline completed successfully!"
            script {
                def msg = "*✅ Jenkins Build Successful*\nJob: ${env.JOB_NAME}\nBuild #: ${env.BUILD_NUMBER}"
                sh """
                    curl -X POST -H 'Content-type: application/json' \
                    --data '{"text": "${msg}"}' ${SLACK_WEBHOOK_URL}
                """
            }
        }

        failure {
            echo "❌ Pipeline failed!"
            script {
                def msg = "*❌ Jenkins Build Failed*\nJob: ${env.JOB_NAME}\nBuild #: ${env.BUILD_NUMBER}"
                sh """
                    curl -X POST -H 'Content-type: application/json' \
                    --data '{"text": "${msg}"}' ${SLACK_WEBHOOK_URL}
                """
            }
        }
    }
}
