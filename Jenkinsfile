pipeline {
    agent any

    environment {
        PATH = "/opt/homebrew/bin:/usr/local/bin:${env.PATH}"
        DOCKER_IMAGE = 'devops-demo'
        DEFAULT_DOCKER_USER = 'janhavi1418'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
                    python3 -m venv .venv || true
                    . .venv/bin/activate
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    . .venv/bin/activate
                    pytest
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:latest .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh '''
                        echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                        docker tag ${DOCKER_IMAGE}:latest ${DOCKERHUB_USER}/${DOCKER_IMAGE}:latest
                        docker push ${DOCKERHUB_USER}/${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker stop devops-container || true
                    docker rm devops-container || true
                    docker run -d --name devops-container -p 5000:5000 ${DOCKER_IMAGE}:latest
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
    }
}
