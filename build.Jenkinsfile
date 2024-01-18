pipeline {
    agent any

    environment {
        ECR_REGISTRY = "933060838752.dkr.ecr.eu-west-2.amazonaws.com"
        TIMESTAMP = new Date().format('yyyyMMdd_HHmmss')
        IMAGE_TAG = "${TIMESTAMP}-${BUILD_NUMBER}"
        KUBECONFIG_CREDENTIAL_ID = "k8s-main-credentials"  // Replace with your credential ID
        CLUSTER_NAME = "k8s-main"
        CLUSTER_REGION = "us-east-1"
    }

    stages {
        stage('Build') {
            steps {
                script {
                    withCredentials([aws(credentialsId: 'jenkins-aws-credentials', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        dockerImage = docker.build("${ECR_REGISTRY}/lana_bot_container:${IMAGE_TAG}", "--no-cache .")
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([kubeconfigContent(credentialsId: KUBECONFIG_CREDENTIAL_ID, variable: 'KUBECONFIG')]) {
                    sh "kubectl apply -f lanabot.yaml --kubeconfig=${KUBECONFIG}"
                }
            }
        }
    }

    post {
        always {
            sh '''
                docker rmi $(docker images -q) -f
            '''
        }
    }
}
