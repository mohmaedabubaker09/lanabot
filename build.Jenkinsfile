pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh '''
                aws ecr get-login-password --region eu-west-2 | docker login --username AWS --password-stdin 933060838752.dkr.ecr.eu-west-2.amazonaws.com
                docker build -t lana_bot_container .
                docker tag lana_bot_container:latest 933060838752.dkr.ecr.eu-west-2.amazonaws.com/lana_bot_container:latest
                docker push 933060838752.dkr.ecr.eu-west-2.amazonaws.com/lana_bot_container:latest
                '''
            }
        }
    }
}