pipeline {
    agent {
        docker { image 'python:3.14.0a1-bullseye' }
    }
    stages {
        stage('Test') {
            steps {
                sh 'python3 main.py'
            }
        }
    }
}