pipeline {
    agent any
    environment {
            PATH=sh(script:"echo $PATH:/usr/local/bin", returnStdout:true).trim()
            AWS_REGION = "us-east-1"
            AWS_ACCOUNT_ID=sh(script:'export PATH="$PATH:/usr/local/bin" && aws sts get-caller-identity --query Account --output text', returnStdout:true).trim()
            ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
            APP_REPO_NAME = "cagatayakk/phonebook-app"
            APP_NAME = "phonebook"
        }

    stages {

        stage('Create ECR Repo') {
            steps {
                echo 'Creating ECR Repo for App'
                sh '''
                aws ecr describe-repositories --region ${AWS_REGION} --repository-name ${APP_REPO_NAME} || \
                aws ecr create-repository \
                  --repository-name ${APP_REPO_NAME} \
                  --image-scanning-configuration scanOnPush=false \
                  --image-tag-mutability MUTABLE \
                  --region ${AWS_REGION}
                '''
            }
        }

        stage('Build App Docker Image') {
            steps {
                echo 'Building App Image'
                sh 'docker build --force-rm -t "$ECR_REGISTRY/$APP_REPO_NAME:latest" -f .'
                sh 'docker image ls'
            }
        }

        stage('Push Image to ECR Repo') {
            steps {
                echo 'Pushing App Image to ECR Repo'
                sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin "$ECR_REGISTRY"'
                sh 'docker push "$ECR_REGISTRY/$APP_REPO_NAME:latest"'
            }
        }

        stage('Deploy the App') {
            steps {
                echo 'Deploy the App'
                sh 'docker stack deploy -c docker-compose.yml phonebook'
             }
        }

    post {
        always {
            echo 'Deleting all local images'
            sh 'docker stack rm phonebook'
            sh 'docker image prune -af'
        }
        failure {

            echo 'Delete the Image Repository on ECR due to the Failure'
            sh """
                aws ecr delete-repository \
                  --repository-name ${APP_REPO_NAME} \
                  --region ${AWS_REGION}\
                  --force
                """
        }
    }
    
}