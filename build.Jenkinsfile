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
                    // Save the current workspace directory
                    def currentWorkspace = pwd()
                    echo "Current Workspace: ${currentWorkspace}"

                    // Print the contents of the current workspace
                    echo "Contents of Current Workspace:"
                    sh 'ls -al'

                    // Checkout to the new repository's main branch
                    checkout([$class: 'GitSCM', branches: [[name: 'main']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'SubmoduleOption', disableSubmodules: false, parentCredentials: true, recursiveSubmodules: false, reference: '', trackingSubmodules: false]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/mohmaedabubaker09/lanabot-k8s.git']]])

                    // Check if the file exists in the original workspace
                    def originalFilePath = "${currentWorkspace}/lana-bot-deployment.yaml"
                    echo "Original File: ${originalFilePath}"

                    // Print the contents of the new workspace after checkout
                    echo "Contents of New Workspace:"
                    sh 'ls -al'

                    // Use shell commands to copy the file to the new workspace
                    sh "cp -f ${originalFilePath} . || true"  // Use -f to force copy, and || true to ignore errors if the file doesn't exist

                    // Verify if the file has been copied successfully
                    sh 'ls -al lana-bot-deployment.yaml || true'

                    def copiedFile = new File("${currentWorkspace}/lana-bot-deployment.yaml")
                    if (copiedFile.exists()) {
                        // Configure git in the new workspace
                        sh 'git config --local user.email "mohmaedabubaker09@gmail.com"'
                        sh 'git config --local user.name "Mohamed Abu Baker"'

                        // Add, commit, and push the lana-bot-deployment.yaml file to the new repository
                        sh 'git add lana-bot-deployment.yaml'
                        sh 'git commit -m "Add lana-bot-deployment.yaml"'
                        sh 'git push origin main'
                    } else {
                        error "Failed to copy lana-bot-deployment.yaml to the new workspace."
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
