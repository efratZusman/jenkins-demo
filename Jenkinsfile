pipeline {
    agent any
    stages {
        stage('Check Docker') {
            steps {
                sh 'docker version'
                sh 'docker-compose version'
            }
        }
    }
}
