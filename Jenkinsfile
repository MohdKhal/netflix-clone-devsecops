pipeline {
    agent any

    environment {
        IMAGE_NAME = 'netflix-clone'
        DOCKER_HUB_REPO = 'mohdibrahimk/netflix-clone'
    }

    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/mohdkhal/netflix-clone-devsecops.git'
            }
        }

        stage('Install Trivy') {
            steps {
                sh '''
                if ! command -v trivy &> /dev/null; then
                  echo "Installing Trivy..."
                  wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_0.50.1_Linux-64bit.deb
                  sudo dpkg -i trivy_0.50.1_Linux-64bit.deb
                fi
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy fs --exit-code 0 --severity HIGH .'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_HUB_REPO:latest ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                      docker push $DOCKER_HUB_REPO:latest
                    '''
                }
            }
        }
    }
}

