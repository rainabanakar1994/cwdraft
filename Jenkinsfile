pipeline {
    agent any

    environment {
        GIT_REPO = ' https://github.com/rainabanakar1994/cwdraft.git'
        BRANCH = 'main'

        DOCKER_IMAGE = raina1994/portfolio
        IMAGE_TAG = "${BUILD_NUMBER}"

        KUBE_NAMESPACE = 'default'
        DEPLOYMENT_NAME = ' portfolio'
        CONTAINER_NAME = ' portfolio'
        SERVICE_NAME = ' portfolio-service'
    }

    stages {
        stage('Code Pull') {
            steps {
                git branch: "${BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Image Build') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .'
                sh 'docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest'
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
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                        docker push ${DOCKER_IMAGE}:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml

                    kubectl set image deployment/${DEPLOYMENT_NAME} \
                      ${CONTAINER_NAME}=${DOCKER_IMAGE}:${IMAGE_TAG} \
                      -n ${KUBE_NAMESPACE}

                    kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${KUBE_NAMESPACE}
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
            sh 'kubectl get pods'
            sh 'kubectl get svc'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}


