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

        stage('Checkout and Push to Another Repo') {
            steps {
                script {
                    // Print current workspace and its contents
                    pwd()
                    echo 'Current Workspace:'
                    sh 'ls -al'

                    // Read the content of the deployment file into a variable
                    def deploymentFileContent = readFile(file: DEPLOYMENT_FILE_PATH).trim()

                    // Checkout the code from the original repository
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/mohmaedabubaker09/lanabot-k8s.git']]])

                    // Print the original file path and contents
                    echo "Original File: ${DEPLOYMENT_FILE_PATH}"
                    echo 'Contents of New Workspace:'
                    sh 'ls -al'

                    // Write the content back to the new workspace
                    writeFile(file: DEPLOYMENT_FILE_NAME, text: deploymentFileContent)

                    // Verify the written file
                    sh "ls -al ${DEPLOYMENT_FILE_NAME}"
                    sh '''

                        git add .
                        git commit -m ${DEPLOYMENT_FILE_NAME}
                    '''
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