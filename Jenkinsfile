pipeline {
    agent any

    environment {
        IMAGE_NAME = 'netflix-clone-devsecops'
        DOCKERHUB_CREDENTIALS = credentials('docker-creds')
        GIT_CREDENTIALS = credentials('git-creds')
        SLACK_WEBHOOK = credentials('slack_webhook_url')
    }

    stages {
        stage('Checkout Code') {
            steps {
                git credentialsId: "${GIT_CREDENTIALS}", url: 'https://github.com/MohdKhal/netflix-clone-devsecops.git', branch: 'main'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    echo "🔍 Running Trivy vulnerability scan..."
                    trivy image --exit-code 0 --severity MEDIUM,HIGH,CRITICAL ${DOCKERHUB_CREDENTIALS_USR}/${IMAGE_NAME} || true
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo "��� Pushing image to DockerHub..."
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
            echo "��� Cleaning up unused Docker images..."
            sh 'docker system prune -af || true'
        }
        success {
            echo "✅ Jenkins pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check console output for details."
        }
    }
}
