pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'raina1994/portfolio'

        DOCKER_TAG = "${BUILD_NUMBER}"

        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'

        GIT_REPO = ' https://github.com/rainabanakar1994/cwdraft.git'

        GIT_BRANCH = 'develop'
    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    stages {
        stage('Code Pull') {
            steps {
                echo '========== STAGE 1: Pulling source code from GitHub =========='

                git branch: "${GIT_BRANCH}",
                    url: "${GIT_REPO}",
                    credentialsId: 'github-credentials'

                echo "Successfully pulled code from branch: ${GIT_BRANCH}"
            }
        }

        stage('Image Build') {
            steps {
                echo '========== STAGE 2: Building Docker image =========='

                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")

                    echo "Docker image built successfully: ${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }

        stage('Push Image') {
            steps {
                echo '========== STAGE 3: Pushing image to Docker Hub =========='

                script {
                    docker.withRegistry('', "${DOCKER_CREDENTIALS_ID}") {
                        dockerImage.push("${DOCKER_TAG}")
                        dockerImage.push('latest')
                    }

                    echo "Image pushed to Docker Hub: ${DOCKER_IMAGE}:${DOCKER_TAG} and :latest"
                }
            }
        }

        stage('Deploy') {
            steps {
                echo '========== STAGE 4: Deploying to Kubernetes =========='

                sh 'kubectl apply -f k8s/deployment.yaml'

                sh 'kubectl apply -f k8s/service.yaml'

                sh 'kubectl rollout restart deployment/portfolio'

                sh 'kubectl rollout status deployment/portfolio --timeout=120s'

                echo '---------- Deployment Status ----------'
                sh 'kubectl get pods -l app=portfolio'
                sh 'kubectl get services portfolio-service'
            }
        }
    }

    post {
        success {
            echo '========== PIPELINE COMPLETED SUCCESSFULLY =========='
            echo "Application deployed. Access at http://<EC2_PUBLIC_IP>:30081"
        }
        failure {
            echo '========== PIPELINE FAILED =========='
            echo 'Check the stage logs above for error details.'
        }
        always {
            sh "docker rmi ${DOCKER_IMAGE}:${DOCKER_TAG} || true"
            sh "docker rmi ${DOCKER_IMAGE}:latest || true"
            echo 'Workspace cleaned up.'
        }
    }
}

