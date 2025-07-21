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

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                sh 'docker build -t $DOCKER_HUB_REPO:latest .'
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
            echo "✅ Jenkins pipeline completed successfully!"

            script {
                def slackMessage = "*✅ Build Successful*\nJob: ${env.JOB_NAME}\nBuild #: ${env.BUILD_NUMBER}\nStatus: SUCCESS"
                sh """
                    curl -X POST -H 'Content-type: application/json' \
                    --data '{"text": "${slackMessage}"}' ${SLACK_WEBHOOK_URL}
                """
            }
        }

        failure {
            echo "❌ Pipeline failed."

            script {
                def slackMessage = "*❌ Build Failed*\nJob: ${env.JOB_NAME}\nBuild #: ${env.BUILD_NUMBER}\nStatus: FAILURE"
                sh """
                    curl -X POST -H 'Content-type: application/json' \
                    --data '{"text": "${slackMessage}"}' ${SLACK_WEBHOOK_URL}
                """
            }
        }
    }
}
