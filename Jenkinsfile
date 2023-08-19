def ecrRepoName = '842485143083.dkr.ecr.ap-southeast-1.amazonaws.com/javaspringpoc'
def dockerImageTag = "${env.BUILD_NUMBER}" // Use Jenkins build number as the tag

pipeline {
    agent any
    
  //  environment {
  //      AWS_CREDENTIALS = credentials('my-aws-credentials') // Use the ID you set for your AWS credentials
  //  }
    
    tools {
        maven 'Maven-3.9.4' // This should match the Maven installation name configured in Jenkins
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Chiku0306/jenkinsspringbootpoc.git']])
            }
        }
        
        stage('Build and Package') {
            steps {
                script {
                    // Build Maven project using the installed Maven version
                    sh 'mvn clean package'
                }
            }
        }
        
        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Build Docker image using the Dockerfile in the workspace directory
                    sh "docker build -t ${ecrRepoName}:${dockerImageTag} ."
                    
                    // Authenticate to AWS ECR using credentials from environment
                    sh "docker login -u AWS -p '${AWS_CREDENTIALS.getSecretKey()}' https://${ecrRepoName}"
                    
                    // Push Docker image to ECR
                    sh "docker push ${ecrRepoName}:${dockerImageTag}"
                }
            }
        }
    }
}
