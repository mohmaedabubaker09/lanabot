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

//         stage('Deploy') {
//             steps {
//                 script {
//                     withCredentials([aws(credentialsId: AWS_CREDENTIALS_ID, accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
//                         sh 'aws eks update-kubeconfig --region ${CLUSTER_REGION} --name ${CLUSTER_NAME}'
//                         withCredentials([file(credentialsId: 'KUBE_CONFIG_CRED', variable: 'KUBECONFIG')]) {
//                             // sh 'aws eks --region us-east-1 update-kubeconfig --name k8s-main'
//                             // sh 'kubectl config set-context --current --namespace=lanabot-dev-ns'
//                             sh "sed -i 's|image: .*|image: ${ECR_REGISTRY}/lana_bot_container:${IMAGE_TAG}|' lana-bot-deployment.yaml"
//                             // sh "cat lana-bot-deployment.yaml"
//                             sh 'kubectl apply -f lana-bot-deployment.yaml' //--validate=false'
//                         }
//                     }
//                     // withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG_FILE')]) {
//                     //    sh "aws eks update-kubeconfig --region us-east-1 --name k8s-main --kubeconfig \$KUBECONFIG_FILE"
//                     // }
//                 }
//             }
//         }

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

        stage('Update GitHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: GITHUB_CREDENTIALS_ID, usernameVariable: 'GITHUB_USERNAME', passwordVariable: 'GITHUB_PASSWORD')]) {
                        dir('path/to/your/repository') {
                            sh 'git config user.email "mohamedabubaker09@gmail.com"'
                            sh 'git config user.name "Mohamed Abu Baker"'
                            sh 'git add lana-bot-deployment.yaml'
                            sh 'git commit -m "Update lana-bot-deployment.yaml"'
                            sh 'git push https://${GITHUB_USERNAME}:${GITHUB_PASSWORD}@${GITHUB_REPO_URL} master'
                        }
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

