pipeline {
    agent any

    tools {
        jdk 'Open JDK 8' // Use the exact name as shown in your Jenkins JDK tool configuration
    }

    environment {
        DOCKER_IMAGE = "ramujava/springboot-crud-k8s:${env.BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS = credentials('docker') // DockerHub Credentials ID from your screenshot
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github-access', url: 'https://github.com/ramujava412/springboot-crud-k8s.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    def image = docker.build(env.DOCKER_IMAGE)
                    docker.withRegistry('https://index.docker.io/v1/', env.DOCKERHUB_CREDENTIALS) {
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
                    sh "kubectl set image deployment/app-deployment app=${DOCKER_IMAGE} --record"
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
