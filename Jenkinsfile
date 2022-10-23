pipeline {
    agent any
    tools { 
        maven 'maven-3.8.6' 
    }
    stages {
        stage('Checkout git') {
            steps {
               git branch: 'sonar', url: 'https://github.com/darinpope/java-web-app'
            }
        }
        
        stage ('Build & JUnit Test') {
            steps {
                sh 'mvn install' 
            }
            post {
               success {
                    junit 'target/surefire-reports/**/*.xml'
                }
                
            }


        }
        
        
        stage('Sonarqube Analysis'){
            steps{
                   withSonarQubeEnv(installationName: 'sonarqube') {
                        sh 'mvn clean org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.0.2155:sonar'
                    }
            }
        }
        stage('Building Docker Image'){
            steps{
                sh '''
                sudo docker build -t nanditasahu/devsecops-demo:$BUILD_NUMBER .
                sudo docker images
                '''
            }
        }
        stage('Image Scanning Trivy'){
            steps{
               sh 'sudo trivy image nanditasahu/devsecops-demo:$BUILD_NUMBER > $WORKSPACE/trivy-image-scan/trivy-image-scan-$BUILD_NUMBER.txt'
               
            }
        }
        stage('Pushing Docker Image into Docker Hub'){
            steps{
                withCredentials([vaultString(credentialsId: 'vault-dockerhub-password', variable: 'DOCKERHUB_PASSWORD')]) {
                sh '''
                sudo docker login -u nanditasahu -p $DOCKERHUB_PASSWORD
                sudo docker push nanditasahu/devsecops-demo:$BUILD_NUMBER
                '''
               }
            }
        }        
        
        stage ('Uploading Reports to Cloud Storage'){
            steps{
                   withCredentials([vaultFile(credentialsId: 'cloud-storage-access', variable: 'CLOUD_CREDS')]) {
                   sh '''
                   gcloud version
                   gcloud auth activate-service-account --key-file="$CLOUD_CREDS"
                   gsutil cp -r $WORKSPACE/trivy-image-scan/trivy-image-scan-$BUILD_NUMBER.txt gs://devsecops-reports
                   gsutil ls gs://devsecops-reports
                   '''
                }

            }
        }
        stage('Cleaning up DockerImage'){
            steps{
                sh 'sudo docker rmi nanditasahu/devsecops-demo:$BUILD_NUMBER'
            }
        }
    }
  }
        
