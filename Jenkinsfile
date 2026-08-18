pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 15, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
    }

    parameters {
        booleanParam(name: 'PUSH_TO_REGISTRY', defaultValue: false, description: 'Push the image to Docker Hub (requires docker-hub-creds)')
    }

    triggers {
        githubPush()
    }

    environment {
        IMAGE_NAME = 'workspace-web'
        CONTAINER_PORT = '8090'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Building from commit: ${env.GIT_COMMIT}"
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker compose build'
                echo "Image ${env.IMAGE_NAME} built"
            }
        }

        stage('Test / Smoke Check') {
            steps {
                bat 'docker compose config --quiet'
                bat 'docker compose up -d'
                bat 'powershell -Command "Start-Sleep -Seconds 3; exit $(if ((Invoke-WebRequest -Uri http://localhost:8090/ -UseBasicParsing -TimeoutSec 15).StatusCode -ne 200) { 1 } else { 0 })"'
            }
        }

        stage('Deploy Application') {
            steps {
                bat 'docker compose up -d'
                bat 'docker compose ps'
            }
        }

        stage('Verify Deployment') {
            steps {
                bat 'powershell -Command "$code = (Invoke-WebRequest -Uri http://localhost:8090/ -UseBasicParsing -TimeoutSec 15).StatusCode; Write-Host Site-responded-with-HTTP-status-$code; exit $(if ($code -ne 200) { 1 } else { 0 })"'
                echo "Deployment verified on port ${env.CONTAINER_PORT}"
            }
        }

        stage('Push to Registry') {
            when {
                expression { params.PUSH_TO_REGISTRY }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                    bat 'docker login -u %DOCKERHUB_USERNAME% -p %DOCKERHUB_PASSWORD%'
                    bat 'docker tag %IMAGE_NAME%:latest %DOCKERHUB_USERNAME%/%IMAGE_NAME%:latest'
                    bat 'docker push %DOCKERHUB_USERNAME%/%IMAGE_NAME%:latest'
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the Jenkins console output for the failing stage.'
        }
        always {
            echo "Pipeline run finished at ${new Date().format('yyyy-MM-dd HH:mm:ss')}"
        }
    }
}
