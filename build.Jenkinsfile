// pipeline {
//     agent any
//
//     stages {
//         stage('Build') {
//             steps {
//                 sh '''
//                 aws ecr get-login-password --region eu-west-2 | docker login --username AWS --password-stdin 933060838752.dkr.ecr.eu-west-2.amazonaws.com
//                 docker build -t lana_bot_container .
//                 docker tag lana_bot_container:latest 933060838752.dkr.ecr.eu-west-2.amazonaws.com/lana_bot_container:latest
//                 docker push 933060838752.dkr.ecr.eu-west-2.amazonaws.com/lana_bot_container:latest
//                 '''
//             }
//         }
//     }
// }

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
}