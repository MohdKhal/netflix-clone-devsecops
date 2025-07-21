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

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "🐳 Building Docker image..."
                    docker build -t ${DOCKERHUB_CREDENTIALS_USR}/${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: 'docker-creds', url: '']) {
                    sh '''
                        echo "🚀 Pushing Docker image to Docker Hub..."
                        docker push ${DOCKERHUB_CREDENTIALS_USR}/${IMAGE_NAME}:latest
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build completed successfully!'
            script {
                def payload = """{
                    "text": "✅ *Jenkins Pipeline Success* - ${env.JOB_NAME} #${env.BUILD_NUMBER} (<${env.BUILD_URL}|View Build>)"
                }"""
                httpRequest httpMode: 'POST',
                    contentType: 'APPLICATION_JSON',
                    requestBody: payload,
                    url: "${SLACK_WEBHOOK}"
            }
        }

        failure {
            echo '❌ Build failed!'
            script {
                def payload = """{
                    "text": "❌ *Jenkins Pipeline Failed* - ${env.JOB_NAME} #${env.BUILD_NUMBER} (<${env.BUILD_URL}|View Build>)"
                }"""
                httpRequest httpMode: 'POST',
                    contentType: 'APPLICATION_JSON',
                    requestBody: payload,
                    url: "${SLACK_WEBHOOK}"
            }
        }
    }
}
