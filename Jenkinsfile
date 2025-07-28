pipeline {
    agent any

    environment {
        IMAGE_NAME = 'todo-flask-app'
        DOCKERHUB_REPO = "${DOCKERHUB_USERNAME}/${IMAGE_NAME}"
    }

    stages {
        stage('Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/XEESHANAKRAM/Project-1-TODO-Flask-Web-App.git'
            }
        }

        stage('Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Scan with Trivy') {
            steps {
                sh "trivy image ${IMAGE_NAME}"
            }
        }

        stage("Docker Build & Push"){
             steps{
                 script{
                   withDockerRegistry(credentialsId: 'dockerhub') {
                      sh "docker build -t ${IMAGE_NAME} ."
                      sh "docker tag ${IMAGE_NAME} xeeshanakram/${IMAGE_NAME}:latest "
                      sh "docker push xeeshanakram/${IMAGE_NAME}:latest "
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@18.208.179.64 <<EOF
                        docker pull xeeshanakram/todo-flask-app:tagname
                        docker rm -f todo-flask-app || true
                        docker run -d --name todo-flask-app -p 5000:5000 xeeshanakram/todo-flask-app:tagname
                        EOF
                    '''
                }
            }
        }
    }
}
