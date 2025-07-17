pipeline {
    agent any

    environment {
        IMAGE_NAME = 'netflix-clone'
        DOCKER_HUB_REPO = 'mohdibrahimk/netflix-clone' // Replace if needed
    }

    stages {
        stage('Clone GitHub Repo') {
            steps {
                echo "Ì≥• Cloning repository..."
                git credentialsId: 'git-creds', url: 'https://github.com/MohdKhal/netflix-clone-devsecops.git', branch: 'main'
            }
        }


        stage('Build Docker Image') {
            steps {
                echo "Ì∞≥ Building Docker image..."
                sh 'docker build -t $DOCKER_HUB_REPO:latest .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo "Ì∫Ä Pushing image to DockerHub..."
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
            echo "Ì∑π Cleaning up unused Docker images..."
            sh 'docker system prune -af || true'
        }
        success {
            echo "‚úÖ Jenkins pipeline completed successfully!"
        }
        failure {
            echo "‚ùå Pipeline failed. Check console output for details."
        }
    }
}

