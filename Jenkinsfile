// Jenkinsfile - Updated for Azure Deployment
pipeline {
    agent any

    environment {
        // Docker Hub Configuration (no changes here)
        DOCKER_REGISTRY_USER = 'sriram016638'
        DOCKER_REGISTRY_CREDENTIAL_ID = 'dockerhub-credentials'
        DOCKER_IMAGE_NAME = "${DOCKER_REGISTRY_USER}/simple-node-app"

        // Azure Configuration (NEW SECTION)
        AZURE_WEBAPP_NAME = 'sriram-jenkins-app' // The unique name you gave your Web App
        AZURE_RESOURCE_GROUP = 'testing'    // The resource group you used
        AZURE_CREDENTIAL_ID = 'azure-credentials'
        AZURE_TENANT_ID_CREDENTIAL = 'azure-tenant-id'
    }

    stages {
        // Stages for Checkout, Build, Test, Build Image, and Push Image remain EXACTLY the same.
        // ... (copy your existing stages here) ...

        stage('Checkout SCM') {
            steps {
                echo 'Checking out code from Git...'
                git branch: 'main', url: 'https://github.com/sriram016638/test.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Installing Node.js dependencies...'
                sh 'docker run --rm -v $(pwd):/app -w /app node:18-slim npm install'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'docker run --rm -v $(pwd):/app -w /app node:18-slim npm test'
            }
        }

        stage('Build Image') {
            steps {
                echo "Building Docker image: ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}"
                sh "docker build -t ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} ."
                sh "docker tag ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} ${DOCKER_IMAGE_NAME}:latest"
            }
        }

        stage('Push Image') {
            steps {
                echo "Pushing image to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: DOCKER_REGISTRY_CREDENTIAL_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}"
                    sh "docker push ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}"
                    sh "docker push ${DOCKER_IMAGE_NAME}:latest"
                    sh "docker logout"
                }
            }
        }


        // STAGE 6: DEPLOY TO AZURE (THIS REPLACES THE OLD DEPLOY STAGE)
        stage('Deploy to Azure') {
            steps {
                echo "Deploying to Azure Web App: ${AZURE_WEBAPP_NAME}"
                // Securely access both the Service Principal and the Tenant ID
                withCredentials([
                    usernamePassword(credentialsId: AZURE_CREDENTIAL_ID, usernameVariable: 'AZURE_APP_ID', passwordVariable: 'AZURE_PASSWORD'),
                    string(credentialsId: AZURE_TENANT_ID_CREDENTIAL, variable: 'AZURE_TENANT_ID')
                ]) {
                    // Step 1: Login to Azure using the Service Principal
                    sh "az login --service-principal -u ${AZURE_APP_ID} -p ${AZURE_PASSWORD} --tenant ${AZURE_TENANT_ID}"

                    // Step 2: Tell the Azure Web App to pull and run the new image from Docker Hub
                    sh "az webapp config container set --name ${AZURE_WEBAPP_NAME} --resource-group ${AZURE_RESOURCE_GROUP} --docker-custom-image-name ${DOCKER_IMAGE_NAME}:latest"

                    // Step 3: Logout for security
                    sh "az logout"
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished. Cleaning up workspace...'
            sh 'docker image prune -f'
        }
    }
}