pipeline {
    agent any

    tools {
        jdk 'Open JDK 8' // Use the exact name from Global Tool Configuration
    }

    environment {
        DOCKER_IMAGE = "ramujava/springboot-crud-k8s:${env.BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS = credentials('docker') // Replace with your actual Docker credential ID
    }

    stages {
        stage('Checkout') {
            steps {
                // Clone your GitHub repository
                git branch: 'main', credentialsId: 'github-access', url: 'https://github.com/ramujava412/springboot-crud-k8s.git'
            }
        }

        stage('Build') {
            steps {
                // Build your project and skip tests if DB issues persist
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    // Build Docker image
                    def image = docker.build("${DOCKER_IMAGE}")
                    // Push Docker image to Docker Hub
                    docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
                        image.push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f db-deployment.yaml'
                    sh 'kubectl apply -f mysql-configMap.yaml'
                    sh 'kubectl apply -f mysql-secrets.yaml'
                    sh 'kubectl apply -f app-deployment.yaml'
                    // Update app deployment image with latest build
                    sh "kubectl set image deployment/app-deployment app=${DOCKER_IMAGE} --record"
                }
            }
        }
    }

    post {
        always {
            // Clean workspace after pipeline run
            cleanWs()
        }
    }
}
