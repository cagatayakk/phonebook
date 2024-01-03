pipeline {
    agent any
    environment {
            PATH=sh(script:"echo $PATH:/usr/local/bin", returnStdout:true).trim()
            DOCKERHUB_CREDENTIALS = credentials('dockerhub')
            APP_REPO_NAME = "todo-app"
            REGISTRY = "cagatayakk"
            APP_NAME = "to-do"
        }

    stages {

        stage('Build App Docker Image') {
            steps {
                echo 'Building App Image'
                sh 'docker build --force-rm -t "$REGISTRY/$APP_REPO_NAME:latest" .'
                sh 'docker image ls'
            }
        }

        stage('Push Image to ECR Repo') {
            steps {
                echo 'Pushing App Image to ECR Repo'
                withDockerRegistry([ credentialsId: "dockerhub", url: "" ]) {
                sh 'docker push "$REGISTRY/$APP_REPO_NAME:latest"'
            }
        }

        stage('Deploy the App') {
            steps {
                echo 'Deploy the App'
                sh 'docker run -d -p 80:3000 "$REGISTRY/$APP_REPO_NAME:latest"'
             }
        }
    }


}