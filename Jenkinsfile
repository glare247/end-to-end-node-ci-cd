pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = "us-east-1"
        REPOSITORY_URI = "134857759301.dkr.ecr.us-east-1.amazonaws.com/node-repo1"


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
                        -Dsonar.projectKey=abey-org_node-project
                        -Dsonar.organization=Abey-org \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.login=$SONAR_TOKEN
                         
 

                    
                    
                    '''
                }

            }
        }


    }

    post {
        success {
            echo 'sonarcloud analysis successful'
        }
        failure {
            echo 'Build failed. check logs above'
        }
    } 
}
