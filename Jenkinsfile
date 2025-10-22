pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "ramujava/springboot-crud-k8s:${env.BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS = credentials('docker')
    }

    stages {
        stage('Checkout') {
            steps {
                // Clone your GitHub repository
                git branch: 'main', credentialsId: 'github-access', url: 'https://github.com/yourusername/springboot-crud-k8s.git'
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
                    docker.build("${DOCKER_IMAGE}")
                }
                script {
                    // Push Docker image to Docker Hub
                    docker.withRegistry('https://index.docker.io/v1/', 'docker') {
                        docker.image("${DOCKER_IMAGE}").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // Apply all your YAMLs for DB and app deployments
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
