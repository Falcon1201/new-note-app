pipeline {
    agent { label 'dev' }

    environment {
        DOCKER_CREDS = credentials('docker-hub-id-pwd')   // This will fetch the docker hub user and password.
        IMAGE_NAME = 'new-note-app'
        IMAGE_TAG = "vi.${BUILD_NUMBER}"
        CONTAINER_NAME = 'new-note-app-container'
        PORT = '9093'
        HOST_IP = '3.110.83.94'
        EMAIL_TO = 'viveksingh12011992@gmail.com'
    }

    stages {
        stage ('Checkout') {
            steps {
                checkout scm  // This will checkout Jenkinsfile from the repo
            }
        }

        stage ('Building Image') {
            steps {
                sh '''
                echo "=== Building Image ==="
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage ('Push to Docker Hub') {
            steps {
                sh '''
                echo "=== Logging into Docker Hub ==="
                echo "$DOCKER_CREDS_PSW" | docker login \
                -u $DOCKER_CREDS_USR --password-stdin
                
                echo "===Tagging Image ==="
                docker tag $IMAGE_NAME:$IMAGE_TAG \
                $DOCKER_CREDS_USR/$IMAGE_NAME:$IMAGE_TAG

                echo "=== Pushing Image to Docker Hub ==="
                docker push $DOCKER_CREDS_USR/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage ('Deploy') {
            steps {
                sh '''
                echo "=== Stopping and removing existing container ==="
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true

                echo "=== Running New Container ==="
                docker run -d --name $CONTAINER_NAME \
                -p $PORT:80 \
                -v notes-data:/data \
                $DOCKER_CREDS_USR/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage ('verify') {
            steps {
                sh '''
                echo "=== Verifying Deployment ==="
                sleep 10
                curl -sf "http://localhost:$PORT" -o /dev/null
                echo "App is UP on $PORT"
                '''
            }
        }
    }

    post {
        success {
            echo "App deployed at: http://${HOST_IP}:${PORT}"
            emailext(
                to: "${EMAIL_TO}",
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                mimeType: 'text/plain',
                body: """Build Successful!

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Image Tag: ${IMAGE_TAG}
App URL: http://${HOST_IP}:${PORT}
Logs: ${env.BUILD_URL}console
"""
            )
            
        }

        failure {
            emailext(
                to: "${EMAIL_TO}",
                subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                mimeType: 'text/plain',
                body: """Build Failed!
                
Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Logs: ${env.BUILD_URL}console
"""
            )
        }

        always {
            echo "=== Cleaning up workspace ==="
            sh 'docker logout || true'
            sh 'docker image prune -f'  
        }
    }
}
