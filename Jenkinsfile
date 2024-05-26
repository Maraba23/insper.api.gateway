pipeline {
    agent any
    stages {
        stage('Build Auth') {
            steps {
                build job: 'api.auth', wait: true
            }
        }

        stage('Build') { 
            steps {
                sh 'mvn clean package'
            }
        }   
        
        stage('Build Image') {
            steps {
                script {
                    auth = docker.build("pasilva2023/auth:${env.BUILD_ID}", "-f Dockerfile .")
                }
            }
        }
        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credential') {
                        auth.push("${env.BUILD_ID}")
                        auth.push("latest")
                    }
                }
            }
        }
        stage('Deploy on Local K8s') {
            when { 
                environment name: 'LOCAL', value: 'true'
            }
            steps {
                withCredentials([ string(credentialsId: 'minikube-credential', variable: 'api_token') ]) {
                    sh "kubectl --token $api_token --server https://host.docker.internal:${env.K8S_LOCAL_PORT}  --insecure-skip-tls-verify=true apply -f ./k8s/deployment.yaml"
                    sh "kubectl --token $api_token --server https://host.docker.internal:${env.K8S_LOCAL_PORT}  --insecure-skip-tls-verify=true apply -f ./k8s/service.yaml"
                }
            }
        }
        stage('Deploy on MicroK8s') {
            when { 
                environment name: 'MicroK8s', value: 'true'
            }
            steps {
                sh "microk8s kubectl apply -f ./k8s/deployment.yaml"
                sh "microk8s kubectl apply -f ./k8s/service.yaml"
            }
        }
        stage('Deploy on AWS K8s') {
            when { 
                environment name: 'AWS', value: 'true'
            }
            steps {
                sh "kubectl apply -f ./k8s/deployment.yaml"
                sh "kubectl apply -f ./k8s/service.yaml"
            }
        }
    }
}