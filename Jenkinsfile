pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'samratbhattarai999'
        IMAGE_NAME      = 'ai-shopkeeper'
        REGISTRY        = "${DOCKER_HUB_USER}/${IMAGE_NAME}"
        BUILD_TAG       = "${BUILD_NUMBER}"
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

        stage('Deploy Container') {
            steps {
                sh '''
                    # Stop and remove old container if running
                    docker stop ai-shopkeeper-app || true
                    docker rm ai-shopkeeper-app || true
                    
                    # Run the fresh container from Docker Hub
                    docker run -d -p 5000:5000 --name ai-shopkeeper-app ${REGISTRY}:latest
                '''
            }
        }

        stage('Verification') {
            steps {
                sh 'docker ps --filter name=ai-shopkeeper-app'
            }
        }
    
      stage('Deploy to Minikube') {
                steps {
                    sh '''
                        export KUBECONFIG=/var/lib/jenkins/.kube/config
                        kubectl config use-context minikube || true
                        kubectl apply --validate=false -f k8s/deployment.yaml
                        kubectl apply --validate=false -f k8s/service.yaml
                        kubectl rollout restart deployment/ai-shopkeeper
                    '''
                }
            }
    post {
        always {
            sh 'docker logout || true'
        }
    }
}
