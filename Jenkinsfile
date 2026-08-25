// Jenkinsfile - Defines the CI/CD Pipeline

pipeline {
    agent any
    
    // Environment variables used throughout the pipeline
    environment {
        // !!! IMPORTANT: REPLACE THIS with your actual DockerHub username/image name
        DOCKER_IMAGE = "your-dockerhub-username/devops-app-image" 
        IMAGE_TAG = "build-${BUILD_NUMBER}" // Unique tag for each build
        K8S_DEPLOYMENT_FILE = "k8s-deployment.yaml"
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Jenkins automatically checks out the code from GitHub
                echo "Code checked out from SCM."
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Builds the image using the Dockerfile in the current directory
                    docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}", ".")
                    echo "Docker image built with tag: ${IMAGE_TAG}"
                }
            }
        }

        stage('Push Image to Registry') {
            steps {
                // Uses the 'dockerhub-creds' ID you set up in Jenkins credentials
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}"
                    
                    // Tag and push the specific build tag AND the 'latest' tag
                    sh "docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest"
                    sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                    
                    sh "docker logout"
                }
            }
        }
        
        stage('Deploy to Kubernetes (CD)') {
            steps {
                script {
                    // 1. Update the K8s manifest with the newly built image tag
                    // Using sed to replace the placeholder image name in the YAML file
                    // This creates an immutable deployment based on the current build
                    sh "sed -i.bak 's|image: .*|image: ${DOCKER_IMAGE}:${IMAGE_TAG}|g' ${K8S_DEPLOYMENT_FILE}"
                    
                    // 2. Apply the updated manifest to the Kubernetes cluster
                    // Jenkins agent must have `kubectl` configured to access the EKS cluster
                    sh "kubectl apply -f ${K8S_DEPLOYMENT_FILE}"
                }
            }
        }
    }
}
