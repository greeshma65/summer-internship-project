pipeline {
    agent {
        label 'slave'
    }

    tools {
     maven 'Maven'
    }

    stages {
        
        stage('Compile the Source Code') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t greeshma369/java-app:latest .'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker login"
                    }
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push greeshma369/java-app:latest"
                    }
                }
            }
        }

        stage('Deploy Application on EKS Cluster') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    credentialsId: 'k8s-token',
                    clusterName: 'EKS-1',
                    namespace: 'webapps',
                    serverUrl: 'https://6C81B218D96318D670F6470A105ACDED.gr7.ap-south-1.eks.amazonaws.com'
                ]]) {
                    sh "kubectl apply -f k8s-deployment.yml -n webapps"
                }
            }
        }

        stage('Verify Pods') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    credentialsId: 'k8s-token',
                    clusterName: 'EKS-1',
                    namespace: 'webapps',
                    serverUrl: 'https://6C81B218D96318D670F6470A105ACDED.gr7.ap-south-1.eks.amazonaws.com'
                ]]) {
                    sh "kubectl get pods -n webapps"
                }
            }
        }

        stage('Verify Services') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    credentialsId: 'k8s-token',
                    clusterName: 'EKS-1',
                    namespace: 'webapps',
                    serverUrl: 'https://6C81B218D96318D670F6470A105ACDED.gr7.ap-south-1.eks.amazonaws.com'
                ]]) {
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
