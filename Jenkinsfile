pipeline {
    agent any

    environment {
        IMAGE_NAME = "raina1994/portfolio"
        IMAGE_TAG = "latest"
        CONTAINER_NAME = "portfolio-container"
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
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
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                echo '========== STAGE 4: Deploying to Kubernetes =========='

                sh 'kubectl apply -f k8s/deployment.yaml'

                sh 'kubectl apply -f k8s/service.yaml'

                

                sh 'kubectl rollout status deployment/portfolio --timeout=600s'

                echo '---------- Deployment Status ----------'
                sh 'kubectl get pods -l app=portfolio'
                sh 'kubectl get services portfolio-service'
            }
        }
    }
}

