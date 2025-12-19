pipeline {
    agent any

    environment {
        AWS_REGION = ""
        ECR_REGISTRY = ""
        ECR_REPO = ""
        IMAGE_TAG = ""
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build App') {
            steps {
                sh '''
                echo "Building application environment"
                cp env.sample .env || true

                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                . venv/bin/activate
                python manage.py test
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh '''
                echo "Logging in to Amazon ECR"
                aws --version
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin $ECR_REGISTRY
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "Building Docker image"
                docker build -t $ECR_REPO:$IMAGE_TAG .
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                docker tag $ECR_REPO:$IMAGE_TAG \
                $ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                echo "Pushing Docker image to ECR"
                docker push $ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-jenkins-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@<> \
                    "bash /var/www/<project_directory>/scripts/deploy.sh"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully"
        }

        failure {
            mail bcc: '',
                 body: """
                 <b>Failed Jenkins Build</b><br>
                 Project: ${env.JOB_NAME}<br>
                 Build Number: ${env.BUILD_NUMBER}<br>
                 URL: ${env.BUILD_URL}
                 """,
                 cc: '',
                 charset: 'UTF-8',
                 from: '',
                 mimeType: 'text/html',
                 replyTo: '',
                 subject: "❌ CI Failed -> ${env.JOB_NAME}",
                 to: ""
        }
    }
}

