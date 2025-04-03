pipeline {
    agent any

    stages {
        stage('Install Dependencies') {
            steps {
                script {
                    bat 'npm install'
                }
            }
        }
        stage('Run Tests') {
            steps {
                script {
                    bat 'npm test'
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    echo 'Build step (if applicable)'
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    echo 'Deploy step (if applicable)'
                }
            }
        }
    }
}
