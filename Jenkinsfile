pipeline {
    agent any

    tools {
       nodejs 'nodejs'
    }

    environment {
        MONGO_USER="admin"
        MONGO_PASS="admin"
        MONGO_DB="app"
        MONGO_HOST="localhost"
        DOCKER_CREDS=credentials('docker-hub')
        IMAGE_NAME="$DOCKER_CREDS_USR/todo-app:$BUILD_TAG"
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    git branch: 'main', credentialsId: 'gethub-private-key', url: 'git@github.com:Ahmed-Elhgawy/dockerized-web-app.git'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                script {
                    sh "npm install --no-audit"
                }
            }
        }
        stage('Check Dependencies') {
            steps {
                script {
                    sh "npm audit --audit-level=high"
                    sh "npm audit --audit-level=critical"
                }
            }
        }
        stage('Test Application') {
            steps {
                script {
                    // Run MongoDB as a docker For Test Environment
                    sh """
                        docker run -d -p 27017:27017 --name mongodb-test \
                        -e MONGO_INITDB_ROOT_USERNAME=$MONGO_USER \
                        -e MONGO_INITDB_ROOT_PASSWORD=$MONGO_PASS \
                        -e MONGO_INITDB_DATABASE=$MONGO_DB \
                        mongo:latest
                    """
                    sh "sleep 30"
                    sh """
                        export MONGO_HOST=$MONGO_HOST
                        export MONGO_USER=$MONGO_USER
                        export MONGO_PASS=$MONGO_PASS
                        export MONGO_DB=$MONGO_DB
                        npm test
                    """
                }
            }
            post {
                always {
                    script {
                        // Stop and remove the MongoDB test container
                        sh "docker stop mongodb-test && docker rm mongodb-test"
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t todo-app:latest ."
                }
            }
        }
        stage('push to Docker Hub') {
            steps {
                script {
                    sh "docker tag todo-app:latest $IMAGE_NAME"
                    withDockerRegistry(credentialsId: 'docker-hub', url: 'https://index.docker.io/v1/') {
                        sh "docker push $IMAGE_NAME"
                    }
                }
            }
            post {
                success {
                    script {
                        sh "docker rmi $IMAGE_NAME"
                    }
                }
            }
        }
        stage('Deploy to EC2 instance') {
            steps {
                script {
                    withAWS(credentials: 'aws-credentials',region: 'us-east-1') {
                        sh "aws ec2 describe-instances --filters Name=instance-state-name,Values=running --query \"Reservations[*].Instances[*].PublicDnsName\" --output text > public_dns.txt"
                    }
                    def EC2_IP=readFile('public_dns.txt').trim()
                    sshagent(['devops-linux-private-key']) {
                        sh """
                            scp -o StrictHostKeyChecking=no docker-compose.yml ec2-user@$EC2_IP:/home/ec2-user/docker-compose.yml
                            ssh -o StrictHostKeyChecking=no ec2-user@$EC2_IP '
                                echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin
                                export IMAGE_NAME=$IMAGE_NAME
                                export MONGO_USER=$MONGO_USER
                                export MONGO_PASS=$MONGO_PASS
                                export MONGO_DB=$MONGO_DB
                                docker pull $IMAGE_NAME
                                envsubst < docker-compose.yml | docker-compose -f - up -d
                            '
                        """
                    }
                }
            }
        }
    }
}