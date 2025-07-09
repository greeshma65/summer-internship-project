pipeline {
    agent {'slave'}

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

        stage('Login to Docker Hub and Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker login"
                    }
                }
            }
        }

        stage('Login to Docker Hub and Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push vikasprince/java-app:latest"
                    }
                }
            }
        }

        stage('Verify Pods') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    credentialsId: 'k8s-token',
                    clusterName: 'java-eks',
                    namespace: 'java-app',
                    serverUrl: 'https://csi-aks-microservices-dns-x9c1s2k4.hcp.centralindia.azmk8s.io'
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
                    serverUrl: 'https://csi-aks-microservices-dns-x9c1s2k4.hcp.centralindia.azmk8s.io'
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
