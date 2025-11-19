pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID   = "562404438689"
        AWS_REGION       = "us-east-1"
        ECR_REGISTRY     = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

        BACKEND_REPO    = "backend/food-ewb"
        FRONTEND_REPO   = "frontend/food-ewb"
        ADMIN_REPO      = "admin/food-ewb"

        IMAGE_TAG        = "${BUILD_NUMBER}"

        LOCAL_BACKEND_IMAGE  = "backend:${IMAGE_TAG}"
        LOCAL_FRONTEND_IMAGE = "frontend:${IMAGE_TAG}"
        LOCAL_ADMIN_IMAGE    = "admin:${IMAGE_TAG}"

        ECS_CLUSTER_NAME      = "food-delivery-cluster"
        ECS_BACKEND_SERVICE   = "food-backend-service"
        ECS_FRONTEND_SERVICE  = "food-frontend-service"
        ECS_ADMIN_SERVICE     = "food-admin-service"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git(
                    url: 'https://github.com/gopi-ganesan/food-delivery-CI-CD.git',
                    branch: 'main',
                    credentialsId: 'github-token'
                )
            }
        }

        stage('Build Docker Images') {
            steps {
                echo 'Building Docker Images...'
                sh "docker-compose -f docker-compose.yml build"
            }
        }

        stage('Login to ECR') {
            steps {
                echo 'Logging in to Amazon ECR...'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws']]) {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} \
                        | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    """
                }
            }
        }

        stage('Push Docker Images to ECR') {
            steps {
                echo 'Pushing Images...'
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

        stage('Deploy to ECS') {
            steps {
                echo 'Deploying Services to ECS...'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws']]) {
                    sh """
                        aws ecs update-service \
                            --cluster ${ECS_CLUSTER_NAME} \
                            --service ${ECS_BACKEND_SERVICE} \
                            --force-new-deployment

                        aws ecs update-service \
                            --cluster ${ECS_CLUSTER_NAME} \
                            --service ${ECS_FRONTEND_SERVICE} \
                            --force-new-deployment

                        aws ecs update-service \
                            --cluster ${ECS_CLUSTER_NAME} \
                            --service ${ECS_ADMIN_SERVICE} \
                            --force-new-deployment
                    """
                }
            }
        }
    }
}
