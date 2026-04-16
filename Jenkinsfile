pipeline {
    agent any

    environment {
        IMAGE_NAME = "raina1994/portfolio"
        IMAGE_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = "portfolio-container"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
    }

    stages {

        stage('Code Pull') {
            steps {
                checkout scm
            }
        }

        stage('Image Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
                sh 'docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest'
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
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                echo '========== STAGE 4: Deploying to Kubernetes =========='

                sh '''
                    kubectl get nodes
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    kubectl set image deployment/portfolio ${CONTAINER_NAME}=${IMAGE_NAME}:${IMAGE_TAG}
                    kubectl rollout status deployment/portfolio --timeout=300s
                    echo '---------- Deployment Status ----------'
                    kubectl get pods -l app=portfolio
                    kubectl get services portfolio-service
                '''
            }
        }
    }
}
