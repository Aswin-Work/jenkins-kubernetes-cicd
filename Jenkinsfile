
pipeline {
    agent any

    options {
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/Aswin-Work/jenkins-kubernetes-cicd.git'
            }
        }

        stage('Validate Kubernetes Manifests') {
            steps {
                withCredentials([
                    file(credentialsId: 'minikube-kubeconfig',
                         variable: 'KUBECONFIG')
                ]) {
                    sh '''
                        kubectl apply --dry-run=client -f k8s/deployment.yaml
                        kubectl apply --dry-run=client -f k8s/service.yaml
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(credentialsId: 'minikube-kubeconfig',
                         variable: 'KUBECONFIG')
                ]) {
                    sh '''
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([
                    file(credentialsId: 'minikube-kubeconfig',
                         variable: 'KUBECONFIG')
                ]) {
                    sh '''
                        kubectl rollout status deployment/hello-app-deployment \
                            --timeout=120s

                        kubectl get pods -l app=hello-app
                        kubectl get service hello-app-service
                    '''
                }
            }
        }

        stage('Test Application') {
            steps {
                withCredentials([
                    file(credentialsId: 'minikube-kubeconfig',
                         variable: 'KUBECONFIG')
                ]) {
                    sh '''
                        set -e

                        kubectl port-forward \
                            --address 127.0.0.1 \
                            service/hello-app-service 18085:85 \
                            > /tmp/hello-app-port-forward.log 2>&1 &

                        PF_PID=$!
                        trap 'kill "$PF_PID" 2>/dev/null || true' EXIT

                        for i in $(seq 1 30); do
                            if curl -fsS http://127.0.0.1:18085/ \
                                -o /tmp/hello-app-response.txt; then
                                break
                            fi
                            sleep 2
                        done

                        cat /tmp/hello-app-response.txt

                        grep -q "Hello, world!" \
                            /tmp/hello-app-response.txt

                        echo "Application test successful!"
                    '''
                }
            }
        }
    }
}
