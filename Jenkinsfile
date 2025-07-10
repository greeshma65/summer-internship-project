pipeline {
    agent {
        label 'slave'
    }

    tools {
     maven 'maven'
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
                sh 'docker build -t vikasprince/java-app:latest .'
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
                        sh "docker push vikasprince/java-app:latest"
                    }
                }
            }
        }

        stage('Deploy Application on EKS Cluster') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    credentialsId: 'k8s-token',
                    clusterName: 'java-eks',
                    namespace: 'java-app',
                    serverUrl: 'https://C7AAFF3B256A5F8A2A3792FF27CC4B0A.gr7.ap-south-1.eks.amazonaws.com'
                ]]) {
                    sh "kubectl apply -f k8s-deployment.yml -n java-app"
                }
            }
        }

        stage('Verify Pods') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    credentialsId: 'k8s-token',
                    clusterName: 'java-eks',
                    namespace: 'java-app',
                    serverUrl: 'https://C7AAFF3B256A5F8A2A3792FF27CC4B0A.gr7.ap-south-1.eks.amazonaws.com'
                ]]) {
                    sh "kubectl get pods -n java-app"
                }
            }
        }

        stage('Verify Services') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    credentialsId: 'k8s-token',
                    clusterName: 'java-eks',
                    namespace: 'java-app',
                    serverUrl: 'https://C7AAFF3B256A5F8A2A3792FF27CC4B0A.gr7.ap-south-1.eks.amazonaws.com'
                ]]) {
                    sh "kubectl get svc -n java-app"
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
