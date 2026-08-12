pipeline {
    agent any
    environment {
        IMAGE_NAME = "test-app:v1"
        NAMESPACE = "test-app"
    }
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build Docker Image') {
            steps { 
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    # Export and import into containerd
                    docker save ${IMAGE_NAME} -o /tmp/test-image.tar
                    sudo ctr -n k8s.io images import /tmp/test-image.tar
                    
                    # Update the deployment
                    kubectl set image deployment/test-app test-app=${IMAGE_NAME} -n ${NAMESPACE} --record
                    kubectl rollout restart deployment/test-app -n ${NAMESPACE}
                    kubectl rollout status deployment/test-app -n ${NAMESPACE} --timeout=2m
                """
            }
        }
        stage('Verify') {
            steps {
                sh "kubectl get pods -n ${NAMESPACE} -l app=test-app"
            }
        }
    }
    post {
        always {
            sh "rm -f /tmp/test-image.tar"
        }
    }
}
