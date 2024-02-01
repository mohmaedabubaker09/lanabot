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
        GITHUB_REPO_URL = 'https://github.com/mohmaedabubaker09/lanabot-k8s.git'
        GITHUB_CREDENTIALS_ID = 'github'
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

        stage('Update File') {
            steps {
                script {
                    sh "sed -i 's|image: .*|image: ${ECR_REGISTRY}/lana_bot_container:${IMAGE_TAG}|' lana-bot-deployment.yaml"
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                script {
                    withCredentials([aws(credentialsId: AWS_CREDENTIALS_ID, accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh 'aws eks update-kubeconfig --region ${CLUSTER_REGION} --name ${CLUSTER_NAME}'
                        withCredentials([file(credentialsId: 'KUBE_CONFIG_CRED', variable: 'KUBECONFIG')]) {
                            sh 'kubectl apply -f lana-bot-deployment.yaml' //--validate=false'

                        }
                    }
                }
            }
        }

        stage('Clone Repository lanabot-k8s') {
            steps {
                script {
//                    sh 'mkdir ./lanabot-k8s'
                   dir('./lanabot-k8s') {
                      git branch: 'main', credentialsId: 'github', url: 'https://github.com/mohmaedabubaker09/lanabot-k8s.git'
                   }
                }
            }
        }

        stage('Update GitHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: GITHUB_CREDENTIALS_ID, usernameVariable: 'GITHUB_USERNAME', passwordVariable: 'GITHUB_PASSWORD')]) {
                        sh 'git config user.email "mohmaedabubaker09@gmail.com"'
                        sh 'git config user.name "Mohamed Abu Baker"'
                        sh 'ls -la'
                        sh 'cp ../lana-bot-deployment.yaml ./'
                        sh 'git add lana-bot-deployment.yaml'
                        sh 'git commit -m "Committing a new version of lana-bot-deployment.yaml"'

                        def remoteExists = sh(script: 'git remote -v | grep origin', returnStatus: true).isSuccess()


                        if (remoteExists) {
                            sh "git push https://${GITHUB_USERNAME}:${GITHUB_PASSWORD}@github.com/mohmaedabubaker09/lanabot-k8s.git main"
                        } else {
                            sh "git remote add origin https://${GITHUB_USERNAME}:${GITHUB_PASSWORD}@github.com/mohmaedabubaker09/lanabot-k8s.git"
                            sh 'git push -u origin main'
                        }
                        sh 'ls ../ -la'
                    }
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
