pipeline {
    agent any

    environment {
        IMAGE_NAME = 'new-note-app:latest'
        CONTAINER_NAME = 'new-note-app-container'
        PORT = '9092'
    }

    stages {
        stage ('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Falcon1201/new-note-app.git'
            }
        }

        stage ('Build Docker Image') {
            steps {
                sh '''
                echo "===== Building Docker Image ====="
                docker build -t $IMAGE_NAME .
                '''
            }
        }

        stage ('Stopping Old Containers') {
            steps {
                sh '''
                echo "===== Stopping Old Containers ====="
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true
                '''
            }
        }

        stage ('Run Container') {
            steps {
                sh '''
                echo "===== Running Container ====="
                docker run -d --name $CONTAINER_NAME -p $PORT:80 -v notes-data:/data $IMAGE_NAME
                '''
            }
        }

        stage ('Verify') {
            steps {
                sh '''
                echo "===== Verifing Deployment ====="
                sleep 5
                curl -s http://3.109.62.237:$PORT | head -n 20
                '''
            }
        }
    }

    post {
        success {
            echo "Notes App deployed successfully!: http://3.109.62.237:$PORT"
        }
        failure {
            echo "Deployment failed. Please check the logs for details."
        }
    }
}
