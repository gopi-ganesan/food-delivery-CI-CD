pipeline {
    agent any

    environment {
            AWS_ACCOUT_ID = "562404438689"
            AWS_REGION = "us-east-1"
            ECR_REGISTRY = "${AWS_ACCOUT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

            BACKEND_REPO = "food-repo-backend" // the is ecr repository name
            FRONTEND_REPO = "food-repo-frontend"
            ADMIN_REPO = "food-repo-admin"

            IMAGE_TAG = "${BUILD_NUMBER}"

            LOCAL_BACKEND_IMAGE = "backend:${IMAGE_TAG}" // the is local image name docker image name
            LOCAL_FRONTEND_IMAGE = "frontend:${IMAGE_TAG}"
            LOCAL_ADMIN_IMAGE = "admin:${IMAGE_TAG}"

            ECS_CLUSTER_NAME = "food-delivery-cluster" // ecs cluster name
            ECS_BACKEND_SERVICE = "food-service"

    }

    stages {
        stage('git clone repo') {
            steps {
                git '',
                branch: 'main',
                credentialsId: 'github-token'
            }
        }

        stage('Build Docker Images') {
            steps {
                echo 'Building Docker Images...'
                sh "docker compose -f docker-compose-prod.yml build"
            }
        }

        stage('ecr login'){
            steps {
                echo 'Logging in to Amazon ECR...'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-cre']]) {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    """
                }
            }
        }

        stage('push to ecr '){  // push docker images to ecr 
            stage{
                echo 'Pushing Docker Images to ECR...'
                sh """
                    docker tag ${LOCAL_BACKEND_IMAGE} ${ECR_REGISTRY}/${BACKEND_REPO}:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/${BACKEND_REPO}:${IMAGE_TAG}
                    
                    docker tag ${LOCAL_FRONTEND_IMAGE} ${ECR_REGISTRY}/${FRONTEND_REPO}:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/${FRONTEND_REPO}:${IMAGE_TAG}

                    docker tag ${LOCAL_ADMIN_IMAGE} ${ECR_REGISTRY}/${ADMIN_REPO}:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/${ADMIN_REPO}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to ECS') {  // deploy to amazon ecs
            steps {
                echo 'Deploying to amazon ECS...'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-cre']]) {
                    sh """
                    aws ecs update-sevice \
                        --cluster ${ECS_CLUSTER_NAME} \  
                        --service ${ECS_BACKEND_SERVICE} \
                        --force-new-deployment

                    aws ecs update-sevice \
                        --cluster ${ECS_CLUSTER_NAME} \
                        --service ${ECS_FRONTEND_SERVICE} \
                        --force-new-deployment

                    aws ecs update-sevice \
                        --cluster ${ECS_CLUSTER_NAME} \
                        --service ${ECS_ADMIN_SERVICE} \
                        --force-new-deployment
                    """
                }
            }
        }
    }
}