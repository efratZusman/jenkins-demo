pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "my-web-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
        TEST_PORT = "8081"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from Git...'
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Run Container for Testing') {
            steps {
                echo 'Running container for testing...'
                script {
                    sh """
                        # מחיקת container קודם אם קיים
                        docker rm -f test-container 2>/dev/null || true
                        
                        # הרצת container חדש על פורט פנוי
                        docker run -d --name test-container -p ${TEST_PORT}:80 ${DOCKER_IMAGE}:${DOCKER_TAG}
                        
                        # המתנה שהשרת יעלה
                        sleep 10
                        
                        # בדיקת השירות
                        curl -f http://host.docker.internal:${TEST_PORT} || exit 1
                        
                        echo "Container is running successfully!"
                    """
                }
            }
        }
        
        stage('Cleanup') {
            steps {
                echo 'Cleaning up test container...'
                script {
                    sh 'docker stop test-container 2>/dev/null || true'
                    sh 'docker rm test-container 2>/dev/null || true'
                }
            }
        }
    }
    
    post {
        success {
            echo "✅ Pipeline completed! Docker image ${DOCKER_IMAGE}:${DOCKER_TAG} is ready!"
        }
        failure {
            echo '❌ Pipeline failed!'
            script {
                sh 'docker stop test-container 2>/dev/null || true'
                sh 'docker rm test-container 2>/dev/null || true'
            }
        }
    }
}
