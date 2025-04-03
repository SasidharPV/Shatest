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
        stage('Commit Checksum to ChecksumSafe Repo') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-credentials-id', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                        bat '''
                        REM Clone the checksumsafe repository
                        git clone https://github.com/SasidharPV/checksumsafe.git checksum-repo

                        REM Copy the checksum file to the cloned repository
                        copy checksum\\Dockerfile.checksum checksum-repo\\Dockerfile.checksum

                        REM Navigate to the cloned repository
                        cd checksum-repo

                        REM Configure Git user details
                        git config user.name "sasidharpv"
                        git config user.email "pendyalasasidhar@outlook.com"

                        REM Commit and push the checksum file
                        git add Dockerfile.checksum
                        git commit -m "Add Dockerfile checksum"
                        git push 
                        '''
                    }
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