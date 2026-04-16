pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'raina1994/portfolio'

        DOCKER_TAG = "${BUILD_NUMBER}"

        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'

        GIT_REPO = ' https://github.com/rainabanakar1994/cwdraft.git'

        GIT_BRANCH = 'main'
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
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                   '''
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

