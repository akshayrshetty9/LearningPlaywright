pipeline {
    agent any

    environment {
        IMAGE_NAME = 'playwright-tests'
        CONTAINER_NAME = 'playwright-container'
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Pulling latest code from Git...'
                git branch: 'main', url: 'https://github.com/your-org/your-repo.git'
            }
        }

        stage('Build Project') {
            steps {
                echo 'Installing npm dependencies (locally or cached)...'
                bat 'npm install'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                bat "docker build -t %IMAGE_NAME% ."
            }
        }

        stage('Run Tests in Container') {
            steps {
                echo 'Running Playwright tests in container...'
                // Remove container if it already exists
                bat "docker rm -f %CONTAINER_NAME% || exit 0"
                
                // Run container with Allure volume mount
                bat """
                    docker run --name %CONTAINER_NAME% --rm ^
                    -v %cd%\\allure-results:/app/allure-results ^
                    -v %cd%\\allure-report:/app/allure-report ^
                    %IMAGE_NAME% npm run test:report
                """
            }
        }

        stage('Publish Allure Report') {
            steps {
                allure([
                    reportBuildPolicy: 'ALWAYS',
                    results: [[path: 'allure-results']]
                       ])
            }
        }

    post {
        always {
            echo 'Cleaning up Docker resources...'
            bat "docker rm -f %CONTAINER_NAME% || exit 0"
        }
    }
}

