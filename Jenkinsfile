pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID="842485143083"
        AWS_DEFAULT_REGION="ap-southeast-1"
        IMAGE_REPO_NAME="javaspringpoc"
        IMAGE_TAG="${env.BUILD_NUMBER}"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
    }
    
    tools {
        maven 'maven-3.9.4' 
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Chiku0306/jenkinsspringbootpoc.git']])
            }
        }
        
        stage('Build Springboot and Package') {
            steps {
                script {
                    sh 'mvn clean package'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_REPO_NAME}:${IMAGE_TAG} ."
                }
            }
        }
        
        stage('Push Docker Image to ECR') {
            steps {
                script {
                    def ecrLogin = sh(script: "aws ecr get-login-password --region ${AWS_DEFAULT_REGION}", returnStdout: true).trim()
                    sh "docker login -u AWS -p '${ecrLogin}' ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                    sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}"
                    sh "docker push ${REPOSITORY_URI}"
                }
            }
        }
    }
}
