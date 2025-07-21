pipeline {
    agent any

    environment {
        IMAGE_NAME = 'netflix-clone'
        DOCKER_HUB_REPO = 'mohdibrahimk/netflix-clone'
        SLACK_WEBHOOK = credentials('slack_webhook_url')
    }

    stages {
        stage('Clone GitHub Repo') {
            steps {
                echo "🔄 Cloning GitHub repository..."
                git credentialsId: 'git-creds', url: 'https://github.com/MohdKhal/netflix-clone-devsecops.git', branch: 'main'
            }
        }

        stage('Trivy Scan') {
            steps {
                echo "🔍 Running Trivy config scan..."
                sh '''
                    if ! command -v trivy &> /dev/null; then
                        echo "❌ Trivy not found. Please install Trivy on Jenkins agent."
                        exit 1
                    fi
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

        stage('Push to DockerHub') {
            steps {
                echo "📦 Pushing Docker image to DockerHub..."
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
            echo "🧹 Cleaning up Docker..."
            sh 'docker system prune -af || true'
        }

        success {
            echo "✅ Pipeline succeeded!"
            slackSend (
                color: 'good',
                message: "✅ *Success*: Netflix Clone pipeline passed on `${env.JOB_NAME}` (<${env.BUILD_URL}|View Build>)",
                webhookUrl: "${SLACK_WEBHOOK}"
            )
        }

        failure {
            echo "❌ Pipeline failed."
            slackSend (
                color: 'danger',
                message: "❌ *Failed*: Netflix Clone pipeline failed on `${env.JOB_NAME}` (<${env.BUILD_URL}|View Build>)",
                webhookUrl: "${SLACK_WEBHOOK}"
            )
        }
    }
}

