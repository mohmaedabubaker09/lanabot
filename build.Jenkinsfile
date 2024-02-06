def gitCredentials // Define gitCredentials variable outside of the pipeline block
withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
    // Set Git credentials
    gitCredentials = "${env.GIT_USERNAME}:${env.GIT_PASSWORD}@github.com"
}


pipeline {
    agent any

    environment {
        ECR_REGISTRY = "933060838752.dkr.ecr.eu-west-2.amazonaws.com"

        TIMESTAMP = new Date().format('yyyyMMdd_HHmmss')
        IMAGE_TAG = "${env.BUILD_NUMBER}_${TIMESTAMP}"

        GITHUB_CREDENTIALS_ID = 'github'
        AWS_CREDENTIALS_ID = 'AWS credentials'
        KUBE_CONFIG_CRED = 'KUBE_CONFIG_CRED'

        ECR_REGION = "eu-west-2"
        CLUSTER_NAME = "k8s-main"
        CLUSTER_REGION = "us-east-1"

        DEPLOYMENT_FILE_NAME = 'lana-bot-deployment.yaml'
        DEPLOYMENT_FILE_PATH = "${env.WORKSPACE}/lanabot-k8s/${DEPLOYMENT_FILE_NAME}"
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
                            // Stash the deployment file
                            //stash includes: 'lana-bot-deployment.yaml', name: 'deploymentFile'
                        }
                    }
                }
            }
        }

        stage('Copy File to Agent') {
            steps {
                // Stash the file in the controller
                stash(name: 'yamlFile', includes: 'lana-bot-deployment.yaml')
            }
        }

        stage('Deploy on Agent') {
            agent {
                label 'agent1'
            }
            steps {
                // Unstash the file on the agent
                unstash 'yamlFile'

                // Output gitCredentials for debugging
                echo "Git Credentials: ${gitCredentials}"

                sh "rm -rf lanabot-k8s" // Remove existing directory

                // Clone the repository
                sh "git clone https://${gitCredentials}/mohmaedabubaker09/lanabot-k8s.git"

                // Copy the file to the cloned repository
                sh "cp lana-bot-deployment.yaml lanabot-k8s/"

                // Commit and push changes
                dir("lanabot-k8s") {
                    sh "git add ."
                    sh "git config user.email 'mohmaedabubaker09@gmail.com'" // Set email address
                    sh "git config user.name 'mohmaedabubaker09'" // Set user name
                    sh "git commit -m 'Add lana-bot-deployment.yaml'"
                    sh "git push origin main"
                }
            }
        }








//         stage('Checkout and Push to Another Repo') {
//             steps {
//                 script {
//                     pwd()
//                     echo 'Current Workspace:'
//                     sh 'ls -al'
//
//                     def deploymentFileContent = readFile(file: DEPLOYMENT_FILE_PATH).trim()
//                     checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/mohmaedabubaker09/lanabot-k8s.git', credentialsId: GITHUB_CREDENTIALS_ID]]])
//                     echo "Original File: ${DEPLOYMENT_FILE_PATH}"
//                     echo 'Contents of New Workspace:'
//                     sh 'ls -al'
//
//                     writeFile(file: DEPLOYMENT_FILE_NAME, text: deploymentFileContent)
//
//                     sh "ls -al ${DEPLOYMENT_FILE_NAME}"
//                 }
//             }
//         }


//                     withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
//                         sh '''
//                             git config --global user.name "${GIT_USERNAME}"
//                             git config --global user.password "${GIT_PASSWORD}"
//                             git add .
//                             git commit -m ${DEPLOYMENT_FILE_NAME}
//                             git push --set-upstream origin main
//                         '''
//                     }
//                 }
//             }
//         }
    }
    post {
        always {
            sh 'docker rmi $(docker images -q) -f || true'
        }
    }
}


// // Outside of the pipeline block, configure Git credentials
// withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
//     // Set Git credentials
//     gitCredentials = "${env.GIT_USERNAME}:${env.GIT_PASSWORD}@github.com"
// }
