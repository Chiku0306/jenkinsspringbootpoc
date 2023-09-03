pipeline {
    agent any
    
     environment {
        MAVEN_HOME = '/usr/share/maven' 
        NEXUS_URL = "http://nexus-repo.cloudgaurdtechnologies.com:8081"
        NEXUS_REPOSITORY = "tech-maven-mixed"
        NEXUS_CREDENTIAL_ID = "nexusCredentials"
        GITHUB_API_TOKEN = credentials('github')
        WORKSPACE_DIR = "${env.WORKSPACE}"
        README_FILE = "${WORKSPACE_DIR}/README.md"
    }


    stages {
        stage('Environment Variables') {
            steps {
                script {
                    // Read properties from env.properties
                    def envProperties = readProperties file: 'env.properties'

                    // Set environment variables from properties
                    env.AWS_ACCOUNT_ID = envProperties.AWS_ACCOUNT_ID
                    env.AWS_DEFAULT_REGION = envProperties.AWS_DEFAULT_REGION
                    env.REPOSITORY_URI = envProperties.REPOSITORY_URI
                    env.SONAR_LOGIN = envProperties.SONAR_LOGIN
                    env.SONAR_PROJECT_KEY = envProperties.SONAR_PROJECT_KEY
                    env.IMAGE_REPO_NAME = "javaspringpoc"
                    env.IMAGE_TAG = env.BUILD_NUMBER
                    env.MAVEN_HOME = '/usr/share/maven'
                    env.SONAR_SCANNER_HOME = tool name: 'sonarqube-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                }
            }
        }

        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Chiku0306/jenkinsspringbootpoc.git']])
            }
        }
          
        stage('Pull Artifacts from Nexus') {
            steps {
                script {
                    def mvnHome =  '/usr/share/maven'
                    sh "${mvnHome}/bin/mvn clean dependency:copy-dependencies"
                }
            }
        }  
          
        stage('Build Springboot and Package') {
            steps {
                script {
                    sh "${env.MAVEN_HOME}/bin/mvn clean package"
                }
            }
        }    
      
        stage('Unit Tests') {
            steps {
                script {
            sh "${env.MAVEN_HOME}/bin/mvn test"
            
                }
            }    
        }
        
        
        stage('Generate BOM') {
            steps {
                script {
                    def mvnHome = '/usr/share/maven'
                    sh "${mvnHome}/bin/mvn org.cyclonedx:cyclonedx-maven-plugin:makeBom"
        }
            }
                }
        
                
        stage ('Dependency Tracker') {
            steps {
                dependencyTrackPublisher artifact: 'target/bom.xml', projectId: 'ad7073e2-05bb-41e7-8753-78c3344fd4e4', synchronous: true
            }
        }     
            
       
        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonarqube-pipeline') {
                        sh """
                        ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                            -D sonar.projectVersion=0.0.1-SNAPSHOT \
                            -D sonar.login=${SONAR_LOGIN} \
                            -D sonar.projectBaseDir=/var/lib/jenkins/workspace/springboot-app \
                            -D sonar.projectKey=${SONAR_PROJECT_KEY} \
                            -D sonar.sourceEncoding=UTF-8 \
                            -D sonar.language=java \
                            -D sonar.sources=src/main/java/com/scalesec/vulnado \
                            -D sonar.tests=src/test/java/com/scalesec/vulnado \
                            -D sonar.java.binaries=target/classes/com/scalesec/vulnado \
                            -D sonar.jacoco.reportsPath=target/jacoco.exec \
                            -D sonar.host.url=https://sonarqube.cloudgaurdtechnologies.com/ 
                        """
                    }
                }
            }
        }
        
        stage('SonarQube Quality Gate') {
            steps {
              timeout(time: 5, unit: 'MINUTES') {
              waitForQualityGate abortPipeline: true
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
        
        stage('Publish Source Artifacts') {
            steps {
                archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
            }
        }
        
        stage('Deploy Artifact to Nexus') {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    def artifactPath = "${WORKSPACE}/target/vulando-0.0.1-SNAPSHOT.JAR"

                    sh "${MAVEN_HOME}/bin/mvn deploy:deploy-file -Durl=${NEXUS_URL}/repository/${NEXUS_REPOSITORY}/ -Dfile=${artifactPath} -DgroupId=${pom.groupId} -DartifactId=${pom.artifactId} -Dpackaging=${pom.packaging} -Dversion=${pom.version} -DrepositoryId=nexus -DgeneratePom=true -DuniqueVersion=false"
                }
            }
        }
        
        stage('Push Docker Image to ECR') {
            steps {
                script {
                    def ecrLogin = sh(script: "aws ecr get-login-password --region ${AWS_DEFAULT_REGION}", returnStdout: true).trim()
                    def dockerLoginCmd = "docker login -u AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                    sh(script: "echo '${ecrLogin}' | ${dockerLoginCmd}", returnStatus: true)
                    sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}"
                    sh "docker push ${REPOSITORY_URI}"
                }
            }
        }
        
       
                
                
        stage('Update README') {
            steps {
                script {
                   
                    def readmeContent = """[![Build Status](http://jenkins.cloudgaurdtechnologies.com:8080/job/springboot-app/badge/icon)](http://jenkins.cloudgaurdtechnologies.com:8080/job/springboot-app/)
[![Code Coverage](https://sonarqube.cloudgaurdtechnologies.com/api/project_badges/measure?project=springboot-app&metric=coverage&token=2072c8ea245c9c6537d7db59ea572c59f8076ce4)](https://sonarqube.cloudgaurdtechnologies.com/dashboard?id=springboot-app)
[![Quality Gate Status](https://sonarqube.cloudgaurdtechnologies.com/api/project_badges/measure?project=springboot-app&metric=alert_status&token=2072c8ea245c9c6537d7db59ea572c59f8076ce4)](https://sonarqube.cloudgaurdtechnologies.com/dashboard?id=springboot-app)
"""
                    writeFile file: README_FILE, text: readmeContent
                }
            }
        }
        
        stage('Write to README.md') {
            steps {
               sh 'echo "Hello, Jenkins!" > /var/lib/jenkins/workspace/springboot-app/README.md'
            }
        }    
        
    }   
   
}                
                    
                    
                    
                
            
        
    









        


