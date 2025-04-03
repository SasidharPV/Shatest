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
        stage('Generate Dockerfile Checksum') {
            steps {
                script {
                    bat 'certutil -hashfile Dockerfile SHA256 > checksum/Dockerfile.checksum'
                }
            }
        }
        stage('Commit Checksum to GitHub') {
            steps {
                script {
                    bat '''
                    git config user.name "Jenkins"
                    git config user.email "jenkins@example.com"
                    git fetch origin
                    git checkout -B main origin/main
                    git add checksum/Dockerfile.checksum
                    git commit -m "Add Dockerfile checksum"
                    git push origin main
                    '''
                }
            }
        }
        stage('Verify Checksum') {
            steps {
                script {
                    bat '''
                    certutil -hashfile Dockerfile SHA256 > checksum/temp.checksum
                    fc checksum/Dockerfile.checksum checksum/temp.checksum > nul
                    if errorlevel 1 (
                        echo "Checksum verification failed!"
                        exit 1
                    ) else (
                        echo "Checksum verification passed."
                    )
                    '''
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