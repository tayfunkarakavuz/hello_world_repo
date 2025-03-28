pipeline {
    agent any

    environment {
        ImageRegistry = "tayfunkarakavuz"
        EC2_IP = "3.72.233.76"
        DockerComposeFile = "docker-compose.yml"
        DotEnvFile = ".env"
    }

    stages {

        stage("buildImage") {
            steps {
                script {
                    echo "Building Docker Image..."
                    sh "docker build -t ${ImageRegistry}/${JOB_NAME}:${BUILD_NUMBER} ."
                }
            }
        }

        stage("pushImage") {
            steps {
                script {
                    echo "Pushing Image to DockerHub..."
                    withCredentials([usernamePassword(credentialsId: 'docker-login', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push ${ImageRegistry}/${JOB_NAME}:${BUILD_NUMBER}"
                    }
                }
            }
        }

        stage("deployCompose") {
            steps {
                script {
                    echo "Deploying with Docker Compose..."
                    sshagent(['ec2']) {
                        sh """
                        scp -o StrictHostKeyChecking=no ${DotEnvFile} ${DockerComposeFile} ec2-user@${EC2_IP}:/home/ec2-user
                        ssh -o StrictHostKeyChecking=no ec2-user@${EC2_IP} "docker compose -f /home/ec2-user/${DockerComposeFile} --env-file /home/ec2-user/${DotEnvFile} down"
                        ssh -o StrictHostKeyChecking=no ec2-user@${EC2_IP} "docker compose -f /home/ec2-user/${DockerComposeFile} --env-file /home/ec2-user/${DotEnvFile} up -d"
                        """
                    }
                }
            }
        }
    }
}