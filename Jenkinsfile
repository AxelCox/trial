pipeline {
    agent any
    environment {
        PATH=sh(script:"echo $PATH:/usr/local/bin", returnStdout:true).trim()
        APP_NAME="status-ok"
        APP_REPO_NAME="insider-repo/${APP_NAME}-app-dev"
        AWS_ACCOUNT_ID=sh(script:'export PATH="$PATH:/usr/local/bin" && aws sts get-caller-identity --query Account --output text', returnStdout:true).trim()
        AWS_REGION="us-east-1"
        ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    }
    
    stages {
        stage('Create ECR Repo') {
            steps {
                echo "Creating ECR Repo for ${APP_NAME} app"
                sh '''
                aws ecr describe-repositories --region ${AWS_REGION} --repository-name ${APP_REPO_NAME} || \
                         aws ecr create-repository \
                         --repository-name ${APP_REPO_NAME} \
                         --image-scanning-configuration scanOnPush=true \
                         --image-tag-mutability MUTABLE \
                         --region ${AWS_REGION}
                '''
            }
        }

        stage('Prepare Tags for Docker Image') {
            steps {
                echo 'Preparing Tag for Docker Image'
                script {
                    env.IMAGE_TAG_STATUS_OK="${ECR_REGISTRY}/${APP_REPO_NAME}:b${BUILD_NUMBER}"
                }
            }
        }
        stage('Build App Docker Image') {
            steps {
                echo 'Building App Dev Image'
                sh "cd status-ok/app/"
                sh 'docker build --force-rm -t "${IMAGE_TAG_STATUS_OK}" ${WORKSPACE}/status-ok/app'
                sh 'docker image ls'
            }
        }
        stage('Push Image to ECR Repo') {
            steps {
                echo "Pushing ${APP_NAME} App Image to ECR Repo"
                sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                sh 'docker push "${IMAGE_TAG_STATUS_OK}"'
            }
        }

        stage('Create Cluster with EKS') {
            steps {
                echo 'Creating EKS Cluster for Dev Environment'
                sh "aws eks describe-cluster --name insider-cluster --query 'cluster.name' || eksctl create cluster -f ./infrastructure/cluster.yaml"
            }
        }

        stage('Wait Creation of EKS Cluster') {
            steps {
                echo 'Waiting EKS Cluster for Dev Environment'
                sh "aws eks wait cluster-active --name insider-cluster"
                sh "aws eks wait nodegroup-active --cluster-name insider-cluster --nodegroup-name ng-1"
            }
        }

        stage('Deploy App on Kubernetes cluster'){
            steps {
                echo 'Deploying App on Kubernetes'
                sh "kubectl apply -f ${WORKSPACE}/status-ok/yamlfiles/deploy-svc-app.yaml"
                sh "sleep 120s"
            }
        }     
    }

    post {
        success {
            echo 'ALB Endpoint'
            sh "kubectl get svc status-ok-svc -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'"
        }

        failure {
            echo 'Deleting all local image'
            sh 'docker image prune -af'
            echo 'Delete the Image Repository on ECR'
            sh """
                aws ecr delete-repository \
                  --repository-name ${APP_REPO_NAME} \
                  --region ${AWS_REGION}\
                  --force
                """
            echo 'Tear down the Kubernetes Cluster'
            sh """
                cd infrastructure
                eksctl delete cluster insider-cluster --region us-east-1
                """
        }
    }   
}