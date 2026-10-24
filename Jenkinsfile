
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
        SWARM_MANAGER = '18.215.154.25'
        STACK_NAME = 'homework2'
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
                }

                sh '''
                    set -eu
                    docker build \
                      -t "${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}" .
                '''
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
                        set -eu
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

        stage('Deploy to Docker Swarm') {
            steps {
                sshagent(credentials: ['aws-manager-ssh']) {
                    sh '''
                        set -eu

                        # Copy the stack file to the manager
                        scp \
                          -o BatchMode=yes \
                          -o StrictHostKeyChecking=yes \
                          docker-stack.yml \
                          "ubuntu@${SWARM_MANAGER}:/home/ubuntu/docker-stack.yml"

                        # Login to ECR and deploy the exact image tag
                        ssh \
                          -o BatchMode=yes \
                          -o StrictHostKeyChecking=yes \
                          "ubuntu@${SWARM_MANAGER}" \
                          "aws ecr get-login-password --region '${AWS_REGION}' |
                           sudo docker login --username AWS --password-stdin '${ECR_REGISTRY}' &&
                           export ECR_REGISTRY='${ECR_REGISTRY}' ECR_REPOSITORY='${ECR_REPOSITORY}' IMAGE_TAG='${IMAGE_TAG}' &&
                           sudo --preserve-env=ECR_REGISTRY,ECR_REPOSITORY,IMAGE_TAG \
                             docker stack deploy \
                             --with-registry-auth \
                             --resolve-image always \
                             -c /home/ubuntu/docker-stack.yml '${STACK_NAME}'"
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sshagent(credentials: ['aws-manager-ssh']) {
                    sh '''
                        set -eu
                        ssh \
                          -o BatchMode=yes \
                          -o StrictHostKeyChecking=yes \
                          "ubuntu@${SWARM_MANAGER}" \
                          "sudo docker stack services '${STACK_NAME}'"
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
