pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Smoke Test') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                    python3 tests/smoke_test.py
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t samratbhattarai999/ai-shopkeeper:latest .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh '''
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                        docker push samratbhattarai999/ai-shopkeeper:latest
                    '''
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker stop ai-shopkeeper-app || true
                    docker rm ai-shopkeeper-app || true
                    docker run -d -p 5000:5000 --name ai-shopkeeper-app samratbhattarai999/ai-shopkeeper:latest
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
    }

    post {
        always {
            sh 'docker logout || true'
        }
    }
}
