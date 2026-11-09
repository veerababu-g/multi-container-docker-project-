
pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'veerababu324'
        IMAGE_NAME = 'multicp'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Downloading source code from GitHub'
                checkout scm
            }
        }

        stage('Validate Docker Compose') {
            steps {
                echo 'Validating Docker Compose configuration'
                sh 'docker compose config --quiet'
            }
        }

        stage('Build Application') {
            steps {
                echo 'Building Node.js Docker image'
                sh 'docker compose build app'
            }
        }

        stage('Deploy Containers') {
            steps {
                echo 'Starting Node.js and MongoDB containers'
                sh 'docker compose up -d'
            }
        }

        stage('Test Application') {
            steps {
                echo 'Checking application health endpoint'
                sh '''
                    docker compose exec -T app node -e "
                    fetch('http://localhost:3000/health')
                      .then(r => {
                        if (!r.ok) process.exit(1);
                        console.log('Application health check passed');
                      })
                      .catch(() => process.exit(1));
                    "
                '''
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'veerababu324',
                        passwordVariable: 'Veerababu2211@'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" |
                        docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
                    docker tag ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} \
                        ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest
                    docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Multi-container CI/CD pipeline completed!'
        }

        failure {
            echo 'FAILED: Check Jenkins console output for errors.'
        }

        always {
            sh 'docker compose ps || true'
        }
    }
}
