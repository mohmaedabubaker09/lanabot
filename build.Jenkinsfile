pipeline {
    agent any

    environment {
        ECR_REGISTRY = "933060838752.dkr.ecr.eu-west-2.amazonaws.com"
        TIMESTAMP = new Date().format('yyyyMMdd_HHmmss')
        IMAGE_TAG = "${env.BUILD_NUMBER}_${TIMESTAMP}"
        ECR_REGION = "eu-west-2"
        AWS_CREDENTIALS_ID = 'AWS credentials'
        KUBE_CONFIG_CRED = 'KUBE_CONFIG_CRED'
        CLUSTER_NAME = "k8s-main"
        CLUSTER_REGION = "us-east-1"
    }

    stages {
        stage('Login to AWS ECR') {
            steps {
                script {
                    withCredentials([aws(credentialsId: AWS_CREDENTIALS_ID, accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh 'aws ecr get-login-password --region ${ECR_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}'
                    }
                }
            }
        }

        stage('Build and Push') {
            steps {
                script {
                    echo "IMAGE_TAG: ${IMAGE_TAG}"
                    dockerImage = docker.build("${ECR_REGISTRY}/lana_bot_container:${IMAGE_TAG}") //, "--no-cache .")
                    dockerImage.push()
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    withCredentials([aws(credentialsId: AWS_CREDENTIALS_ID, accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh 'aws eks update-kubeconfig --region ${CLUSTER_REGION} --name ${CLUSTER_NAME}'
                        withCredentials([file(credentialsId: 'KUBE_CONFIG_CRED', variable: 'KUBECONFIG')]) {
                            sh "sed -i 's|image: .*|image: ${ECR_REGISTRY}/lana_bot_container:${IMAGE_TAG}|' lana-bot-deployment.yaml"
                            sh 'kubectl apply -f lana-bot-deployment.yaml'
                        }
                    }
                }
            }
        }

stage('Checkout and Push to Another Repo') {
    steps {
        script {
            // Print current workspace and its contents
            pwd()
            echo 'Current Workspace:'
            sh 'ls -al'

            // Checkout the code from the original repository
            checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/mohmaedabubaker09/lanabot-k8s.git']]])

            // Print the original file path and contents
            echo "Original File: ${env.WORKSPACE}/lana-bot-deployment.yaml"
            echo 'Contents of New Workspace:'
            sh 'ls -al'

            // Copy the file to the new workspace
            sh "cp -f ${env.WORKSPACE}/lanabot-k8s/lana-bot-deployment.yaml ."

            // Verify the copied file
            sh 'ls -al lana-bot-deployment.yaml'

            // Use CopyArtifact to copy files from the original build
            copyArtifacts(
                projectName: 'lanabot-build-pipeline',
                filter: 'lana-bot-deployment.yaml',
                fingerprintArtifacts: true,
                flatten: true,
                target: '.'
            )
        }
    }
}

    }
    post {
        always {
            sh 'docker rmi $(docker images -q) -f || true'
        }
    }
}