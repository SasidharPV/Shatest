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
        stage('Build Docker Image') {
            steps {
                script {
                    bat 'docker build -t venkatasasidhar/shatest .'
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
                        bat 'docker push venkatasasidhar/shatest'
                    }
                }
            }
        }
        stage('Run Docker Container') {
            steps {
                script {
                    bat '''
                    docker stop sample-app-container || exit 0
                    docker rm sample-app-container || exit 0
                    docker run -d -p 3000:3000 --name sample-app-container venkatasasidhar/shatest
                    '''                
                    }
            }
        }
        stage('Generate Dockerfile Checksum') {
            steps {
                script {
                    bat 'certutil -hashfile Dockerfile SHA256 > Dockerfile.checksum'
                }
            }
        }
        stage('Commit Checksum to GitHub') {
            steps {
                script {
                    bat '''
                    git config user.name "Jenkins"
                    git config user.email "jenkins@example.com"
                    git checkout main
                    git add Dockerfile.checksum
                    git commit -m "Add Dockerfile checksum"
                    git push origin main
                    '''
                }
            }
        }
        stage('Cleanup Docker') {
            steps {
                script {
                    bat 'docker stop sample-app-container || exit 0'
                    bat 'docker rm sample-app-container || exit 0'
                    bat 'docker rmi venkatasasidhar/shatest || exit 0'
                }
            }
        }
    }
}