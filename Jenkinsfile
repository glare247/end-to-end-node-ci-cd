pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = "us-east-1"
        REPOSITORY_URI = "134857759301.dkr.ecr.us-east-1.amazonaws.com/node-repo1"
        IMAGE_TAG = "${BUILD_NUMBER}"


    }
    tools {

        nodejs 'node18'
        jdk 'jdk17'
    }
    stages{
        stage ('checkout code'){
            steps {
                checkout scm
            }


        }

        stage ('RunSonarCloudAnalysis'){
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        docker run --rm \
                        -e SONAR_TOKEN=$SONAR_TOKEN \
                        -v $(pwd):/usr/src \
                        sonarsource/sonar-scanner-cli \
                        -Dsonar.projectKey=abey-org_node-project \
                        -Dsonar.organization=abey-org \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.login=$SONAR_TOKEN
                         
 

                    
                    
                    '''
                }

            }
        }
        stage ('Login to ECR'){
            steps {
                withCredentials([aws(credentialsId: 'AWS-ECR-CRED', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY' )]){
                    sh '''
                        aws ecrbget-login-password --region $AWS_DEFAULT_REGION \
                        | docker login --username AWS --password-stdin $REPOSITORY_URI
                    
                    
                    
                    '''

                }
            }
        }
        stage('Build Docker Image') {
            steps{
                sh '''
                    docker build -t my-app:$IMAGE_TAG app/
                    docker my-app:$IMAGE_TAG $REPOSITORY_URI:$IMAGE_TAG
                
                
                '''

            }
        }

        stage ('PUSH to ECR '){
            steps{
                sh '''
                    docker push $REPOSITORY_URI:$IMAGE_TAG
                
                
                '''
            }
        }






    }

    post {
        success {
            echo 'sonarcloud analysis successful'
            echo 'Build and Docker Image Push Succesfull'
            echo "Pushed Image: $REPOSITORY_URI:$IMAGE_TAG"
        }
        failure {
            echo 'Build failed. check logs above'
        }
    } 
}
