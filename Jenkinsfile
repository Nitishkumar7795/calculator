pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "nitishnatikar360/calculator:latest"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_PASS'
                )]) {
                    sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    string(credentialsId: 'k8s-sa-token', variable: 'K8S_TOKEN'),
                    string(credentialsId: 'k8s-ca-crt-string', variable: 'K8S_CA_CRT_STRING')
                ]) {
                    sh '''
                    echo "$K8S_CA_CRT_STRING" > ca.crt
                    kubectl config set-cluster k8s --server=https://10.0.20.144:6443 --certificate-authority=ca.crt
                    kubectl config set-credentials jenkins --token=$K8S_TOKEN
                    kubectl config set-context k8s --cluster=k8s --user=jenkins --namespace=jenprod
                    kubectl config use-context k8s
                    kubectl apply -f k8s.yaml -n jenprod
                    '''
                }
            }
        }
    }
}