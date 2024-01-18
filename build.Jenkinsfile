pipeline {
    agent any

    environment {
        ECR_REGISTRY = "933060838752.dkr.ecr.eu-west-2.amazonaws.com"
        TIMESTAMP = new Date().format('yyyyMMdd_HHmmss')
        IMAGE_TAG = "${TIMESTAMP}-${BUILD_NUMBER}"
        KUBECONFIG_CREDENTIAL_ID = "kubeconfig"
        CLUSTER_NAME = "k8s-main"
        ECR_REGION = "eu-west-2"
        CLUSTER_REGION = "us-east-1"
        AWS_CREDENTIALS_ID = 'AWS credentials'
    }

    stages {
        stage('Login to AWS ECR') {
            steps {
                script {
                    withCredentials([aws(credentialsId: AWS_CREDENTIALS_ID, accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh "aws ecr get-login-password --region ${ECR_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                    }
                }
            }
        }

        stage('Build and Push') {
            steps {
                script {
                    dockerImage = docker.build("${ECR_REGISTRY}/lana_bot_container:${IMAGE_TAG}", "--no-cache .")
                    dockerImage.push()
                }
            }
        }

        stage('Deploy') {
//             steps {
//                 withCredentials([kubeconfigContent(credentialsId: KUBECONFIG_CREDENTIAL_ID, variable: 'KUBECONFIG')]) {
//                     sh "kubectl set image -f lanabot.yaml lana_bot_container=${ECR_REGISTRY}/lana_bot_container:${IMAGE_TAG} --kubeconfig=${KUBECONFIG}"
//                     sh "kubectl apply -f lanabot.yaml --kubeconfig=${KUBECONFIG}"
//                 }
//             }

        stage('Update Deployment Image') {
            steps {
                withCredentials([file(credentialsId: 'KUBECONFIG_CREDENTIAL_ID', variable: 'KUBECONFIG')]) {
                    sh "kubectl apply -f lanabot.yaml --kubeconfig=${KUBECONFIG}"
            }
        }
    }

    post {
        always {
            sh 'docker rmi $(docker images -q) -f || true'
        }
    }
}
