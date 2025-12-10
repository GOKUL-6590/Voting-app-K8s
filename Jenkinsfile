pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'gokulmp7'
        BUILD_TAG = "${BUILD_NUMBER}"
        DEV_NAMESPACE = 'voting-app-dev'
        PROD_NAMESPACE = 'voting-app'
        KUBECONFIG = '/var/lib/jenkins/.kube/config'  
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                echo 'Pulling Code from GitHub'
                checkout scm
                sh 'git log -1 --oneline'
            }
        }
        
        stage('Build Vote Image') {
            steps {
                echo 'Building Vote Service'
                sh 'docker build -t ${DOCKER_REGISTRY}/vote-app:${BUILD_TAG} -t ${DOCKER_REGISTRY}/vote-app:latest -f vote/Dockerfile vote/'
            }
        }
        
        stage('Build Worker Image') {
            steps {
                echo 'Building Worker Service'
                sh 'docker build -t ${DOCKER_REGISTRY}/worker-app:${BUILD_TAG} -t ${DOCKER_REGISTRY}/worker-app:latest -f worker/Dockerfile worker/'
            }
        }
        
        stage('Build Result Image') {
            steps {
                echo 'Building Result Service'
                sh 'docker build -t ${DOCKER_REGISTRY}/result-app:${BUILD_TAG} -t ${DOCKER_REGISTRY}/result-app:latest -f result/Dockerfile result/'
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing to Docker Hub'
                withCredentials([usernamePassword(credentialsId: 'docker-id', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${DOCKER_REGISTRY}/vote-app:${BUILD_TAG}
                        docker push ${DOCKER_REGISTRY}/vote-app:latest
                        docker push ${DOCKER_REGISTRY}/worker-app:${BUILD_TAG}
                        docker push ${DOCKER_REGISTRY}/worker-app:latest
                        docker push ${DOCKER_REGISTRY}/result-app:${BUILD_TAG}
                        docker push ${DOCKER_REGISTRY}/result-app:latest
                        docker logout
                    '''
                }
            }
        }
        
        stage('Update Manifests') {
            steps {
                echo 'Updating K8s manifests'
                sh '''
                    sed -i "s|gokulmp7/vote-app:.*|gokulmp7/vote-app:${BUILD_TAG}|g" k8s-yaml/vote-deployment.yaml
                    sed -i "s|gokulmp7/worker-app:.*|gokulmp7/worker-app:${BUILD_TAG}|g" k8s-yaml/worker-deployment.yaml
                    sed -i "s|gokulmp7/result-app:.*|gokulmp7/result-app:${BUILD_TAG}|g" k8s-yaml/result-deployment.yaml
                '''
            }
        }
        
        stage('Deploy to DEV') {
            steps {
                echo 'Deploying to DEV'
                sh '''
                    kubectl create namespace ${DEV_NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -
                    kubectl apply -f k8s-yaml/ -n ${DEV_NAMESPACE}
                    kubectl get all -n ${DEV_NAMESPACE}
                '''
            }
        }
        
        stage('Approval') {
            steps {
                input message: 'Deploy to Production?', ok: 'Deploy'
            }
        }
        
        stage('Deploy to PROD') {
            steps {
                echo 'Deploying to PRODUCTION'
                sh '''
                    kubectl create namespace ${PROD_NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -
                    kubectl apply -f k8s-yaml/ -n ${PROD_NAMESPACE}
                    kubectl get all -n ${PROD_NAMESPACE}
                '''
            }
        }
    }
    
    post {
        always {
            sh 'docker image prune -af --filter "until=168h" || true'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
