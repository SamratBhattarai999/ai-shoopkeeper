pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'samratbhattarai999'
        IMAGE_NAME      = 'ai-shopkeeper'
        REGISTRY        = "${DOCKER_HUB_USER}/${IMAGE_NAME}"
        BUILD_TAG       = "${BUILD_NUMBER}"
        KUBECONFIG      = '/var/lib/jenkins/.kube/config'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'git status'
            }
        }

        stage('Build & Smoke Test') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                    if [ -f tests/smoke_test.py ]; then
                        python3 tests/smoke_test.py
                    else
                        python3 -m py_compile backend/app.py
                    fi
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ${REGISTRY}:${BUILD_TAG} .
                    docker tag ${REGISTRY}:${BUILD_TAG} ${REGISTRY}:latest
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                        docker push ${REGISTRY}:${BUILD_TAG}
                        docker push ${REGISTRY}:latest
                    '''
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                sh '''
                    sed -i "s|DOCKER_HUB_USERNAME/ai-shopkeeper:IMAGE_TAG|${REGISTRY}:${BUILD_TAG}|g" k8s/deployment.yaml
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    kubectl rollout status deployment/ai-shopkeeper-deployment --timeout=120s
                '''
            }
        }

        stage('Verification') {
            steps {
                sh '''
                    kubectl get pods -l app=ai-shopkeeper
                    kubectl get svc ai-shopkeeper-service
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
