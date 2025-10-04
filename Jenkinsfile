// Jenkinsfile
pipeline {
    // 1. Agent Configuration
    // Specifies where the pipeline will run. 'any' means any available agent.
    agent any

    // 2. Environment Variables
    // Define variables to be used throughout the pipeline.
    environment {
        // Use your Docker Hub username here
        DOCKER_REGISTRY_USER = 'sriram016638'
        // We will store the password securely in Jenkins Credentials
        DOCKER_REGISTRY_CREDENTIAL_ID = 'dockerhub-credentials'
        // Define the name of our Docker image
        DOCKER_IMAGE_NAME = "${DOCKER_REGISTRY_USER}/simple-node-app"
    }

    // 3. Stages: The main steps of the pipeline
    stages {
        // STAGE 1: Checkout Code
        stage('Checkout SCM') {
            steps {
                echo 'Checking out code from Git...'
                // Jenkins automatically checks out the code when the job runs
                git branch: 'main', url: 'https://github.com/sriram016638/test.git'
            }
        }

        // STAGE 2: Install Dependencies
        stage('Build') {
            steps {
                echo 'Installing Node.js dependencies...'
                // Run npm install inside a node docker container for a clean build environment
                sh 'docker run --rm -v $(pwd):/app -w /app node:18-slim npm install'
            }
        }

        // STAGE 3: Run Tests
        stage('Test') {
            steps {
                echo 'Running tests...'
                // Run the test script defined in package.json
                sh 'docker run --rm -v $(pwd):/app -w /app node:18-slim npm test'
            }
        }

        // STAGE 4: Build Docker Image
        stage('Build Image') {
            steps {
                echo "Building Docker image: ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}"
                // The BUILD_NUMBER is a unique, incrementing number provided by Jenkins
                sh "docker build -t ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} ."
                sh "docker tag ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} ${DOCKER_IMAGE_NAME}:latest"
            }
        }

        // STAGE 5: Push to Docker Hub
        stage('Push Image') {
            steps {
                echo "Pushing image to Docker Hub..."
                // Use the withCredentials block to securely handle the Docker Hub password
                withCredentials([usernamePassword(credentialsId: DOCKER_REGISTRY_CREDENTIAL_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}"
                    sh "docker push ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}"
                    sh "docker push ${DOCKER_IMAGE_NAME}:latest"
                    sh "docker logout"
                }
            }
        }

        // STAGE 6: Deploy
        stage('Deploy') {
            steps {
                echo 'Deploying container...'
                // Stop and remove any old container with the same name
                sh "docker stop simple-node-app || true"
                sh "docker rm simple-node-app || true"
                // Run the new container
                sh "docker run -d --name simple-node-app -p 8081:8080 ${DOCKER_IMAGE_NAME}:latest"
            }
        }
    }

    // 4. Post-build Actions
    // Actions that run at the end of the pipeline, regardless of status.
    post {
        always {
            echo 'Pipeline finished. Cleaning up workspace...'
            // Clean up old docker images to save space
            sh 'docker image prune -f'
        }
    }
}