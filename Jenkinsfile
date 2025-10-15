pipeline {
    agent any

    tools {
        maven 'Maven3' // Must match the name from Global Tool Configuration
    }

    environment {
        // CHANGE THIS to your Docker Hub username
        DOCKERHUB_USERNAME = "cityisbetter" 
        // CHANGE THIS to the EKS cluster name from Terraform output
        EKS_CLUSTER_NAME = "my-eks-cluster" 
        // CHANGE THIS to your AWS region
        AWS_REGION = "us-east-1" 
        DOCKER_IMAGE = "${DOCKERHUB_USERNAME}/spring-boot-demo:${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/maheshpaulj/spring-boot-docker-demo.git', branch: 'main'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }
        
        stage('Configure Kubectl') {
            steps {
                sh "aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl set image deployment/spring-boot-demo-deployment spring-boot-demo-container=${DOCKER_IMAGE}"
                sh "kubectl rollout status deployment/spring-boot-demo-deployment"
            }
        }
    }
    
    post {
        always {
            sh 'docker logout'
        }
    }
}