pipeline {
    agent any

    environment {
        ECR_REGISTRY = "933060838752.dkr.ecr.eu-west-2.amazonaws.com"
        TIMESTAMP = new Date().format('yyyyMMdd_HHmmss')
        IMAGE_TAG = "${TIMESTAMP}-${BUILD_NUMBER}"
    }

    stages {
        stage('Build') {
            steps {
                script {
                    sh "aws ecr get-login-password --region eu-west-2 | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                    dockerImage = docker.build("${ECR_REGISTRY}/lana_bot_container:${IMAGE_TAG}")
                    dockerImage.push()
                }
            }
        }
    }
    post {
        always {
            sh "docker image prune -f"
            // sh "docker rmi $(docker images --filter dangling=true -q)"
        }
    }
}
