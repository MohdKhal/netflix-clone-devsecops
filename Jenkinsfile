pipeline {
    agent any

    environment {
        IMAGE_NAME = 'netflix-clone'
        DOCKER_HUB_REPO = 'mohdibrahimk/netflix-clone'
        SLACK_WEBHOOK = credentials('slack_webhook_url')
        DOCKER_CREDENTIALS_ID = 'docker-creds'
        GIT_CREDENTIALS_ID = 'git-creds'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git credentialsId: "${GIT_CREDENTIALS_ID}", url: 'https://github.com/MohdKhal/netflix-clone-devsecops.git', branch: 'main'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy image --exit-code 0 --severity MEDIUM,HIGH,CRITICAL ${DOCKER_HUB_REPO}:latest || true'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_HUB_REPO}:${BUILD_NUMBER} .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_HUB_REPO}:${BUILD_NUMBER}
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline succeeded!"
            script {
                def payload = """{
                    "text": "✅ *Success*: Netflix Clone pipeline passed on `${env.JOB_NAME}` (<${env.BUILD_URL}|View Build>)"
                }"""
                httpRequest httpMode: 'POST', contentType: 'APPLICATION_JSON',
                            requestBody: payload, url: "${SLACK_WEBHOOK}"
            }
        }

        failure {
            echo "❌ Pipeline failed."
            script {
                def payload = """{
                    "text": "❌ *Failed*: Netflix Clone pipeline failed on `${env.JOB_NAME}` (<${env.BUILD_URL}|View Build>)"
                }"""
                httpRequest httpMode: 'POST', contentType: 'APPLICATION_JSON',
                            requestBody: payload, url: "${SLACK_WEBHOOK}"
            }
        }
    }
}
