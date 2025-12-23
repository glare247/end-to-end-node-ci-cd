pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = "us-east-1"
        ECR_REGISTRY = "134857759301.dkr.ecr.us-east-1.amazonaws.com"
        REPOSITORY_URI = "134857759301.dkr.ecr.us-east-1.amazonaws.com/node-repo1"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    tools {
        nodejs 'node18'
        jdk 'jdk17'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Run SonarCloud Analysis') {
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
                          -Dsonar.host.url=https://sonarcloud.io
                    '''
                }
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([
                    aws(credentialsId: 'AWS-ECR-CRED',
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                        aws ecr get-login-password --region $AWS_DEFAULT_REGION \
                        | docker login --username AWS --password-stdin $ECR_REGISTRY
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t my-app:$IMAGE_TAG app/
                    docker tag my-app:$IMAGE_TAG $REPOSITORY_URI:$IMAGE_TAG
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                    docker push $REPOSITORY_URI:$IMAGE_TAG
                '''
            }
        }

        stage ('Deploy to Kubernetes'){
            steps {
                withCredentials([file(credentialsId: 'KUBECONFIG_DEVOPS', variable: 'KUBECONFIG'),
                aws(credentialsId: 'AWS-ECR-CRED', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG
                        export AWS_DEFAULT_REGION=us-east-1

                        echo "installing prometheus monitor...."
                        helm repo add prometheus-community https://prometheus-community.github.io/helm-charts || true

                        helm repo update
                        helm upgrade --install prometheus \
                          prometheus-community/kube-prometheus-stack \
                          --namespace monitoring --create-namespace

                        echo "Updating image tag in deployment.yaml.."
                        sed -i "s|ECR_URI:latest|${REPOSITORY_URI}:${IMAGE_TAG}|g" K8s/deployment.yaml

                        echo "Applying Kubernetes manifests..."
                        kubectl apply -f K8s/

                        echo "Verifying rollout..."
                        kubectl rollout status deployment/node-app
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'SonarCloud analysis successful'
            echo 'Docker image built and pushed successfully'
            echo "Pushed Image: $REPOSITORY_URI:$IMAGE_TAG"
            echo 'Kubernetes deployment successful!'
        }
        failure {
            echo 'Build failed. Check logs above'
        }
    }
}
