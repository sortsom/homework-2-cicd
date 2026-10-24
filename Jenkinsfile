
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REGISTRY = '753674273284.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPOSITORY = 'homework2-app'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    env.IMAGE_TAG = sh(
                        script: 'git rev-parse --short=12 HEAD',
                        returnStdout: true
                    ).trim()

                    sh '''
                        docker build \
                          -t ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG} .
                    '''
                }
            }
        }

        stage('Push to Amazon ECR') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-ecr-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        set +x
                        export AWS_DEFAULT_REGION="${AWS_REGION}"

                        aws ecr get-login-password \
                          --region "${AWS_REGION}" |
                        docker login \
                          --username AWS \
                          --password-stdin "${ECR_REGISTRY}"

                        docker push \
                          "${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}"

                        docker logout "${ECR_REGISTRY}"
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Jenkins pipeline finished.'
        }
    }
}
