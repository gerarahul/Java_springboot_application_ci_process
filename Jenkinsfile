pipeline {
    agent{
        label "slave"
    }
    environment {     
    DOCKERHUB_CREDENTIALS= credentials('docker_credentials')     
    }
    stages{
        stage("Git Checkout"){
            steps{
                git branch: 'main', url: 'https://github.com/gerarahul/sample--project.git'
            }
        }
        stage("Unit Testing"){
            steps{
                sh "mvn test"
            }
        }
        stage("Integration testing"){
            steps{
                sh 'mvn verify -DskipUnitTests'
            }
        }
        stage('Maven Build'){
            steps{
                sh 'mvn clean install'
            }
        }
        stage('Static code analysis'){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'Sonar-api-key'){
                        sh 'mvn clean package sonar:sonar'
                    }
                }
            }
        }
        stage('Quality gate status'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-api-key'
                }
            }
        }
        stage('Upload jar file to nexus'){
            steps{
                script{
                    nexusArtifactUploader artifacts: 
                    [
                        [
                        artifactId: 'springboot', 
                        classifier: '', file: 'target/Uber.jar', 
                        type: 'jar'
                        ]
                    ], 
                    credentialsId: 'nexus-jenkins-authentiation', 
                    groupId: 'com.example', 
                    nexusUrl: '13.233.167.76:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'demoapp-release', 
                    version: '1.3.0'
                }
            }
        }
        stage('Docker image build'){
            steps{
                script{
                    sh 'docker image build -t $JOB_NAME:v1.$BUILD_ID .'
                    sh 'docker tag $JOB_NAME:v1.$BUILD_ID rgera0901/$JOB_NAME:v1.$BUILD_ID'
                    sh 'docker tag $JOB_NAME:v1.$BUILD_ID rgera0901/$JOB_NAME:latest'
                    echo 'Docker Image build succesfully'
                }
            }
        }
        stage('Login to Docker Hub'){      
            steps{
                sh 'docker logout'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'                 
                echo 'Login Completed'                
            }           
        }  
        stage("Push image to docker hub"){
            steps{
                sh 'docker push rgera0901/$JOB_NAME:v1.$BUILD_ID'
                sh 'docker push rgera0901/$JOB_NAME:latest'
                echo "Image pushed to docker_hub successfully"
                }
        }
    }
}
