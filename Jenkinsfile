pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "markbosire/express-server"
        GCP_PROJECT = "onyancha-ni-goat"
        GKE_CLUSTER = "express-gke-cluster"
        GKE_ZONE = "us-central1-a"
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/markbosire/express-server.git', branch: 'main'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${env.BUILD_ID}")
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-creds') {
                        docker.image("${DOCKER_IMAGE}:${env.BUILD_ID}").push()
                    }
                }
            }
        }
        stage('Deploy to GKE') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GCP_KEY')]) {
                        // Authenticate to GCP
                        sh 'gcloud auth activate-service-account --key-file=$GCP_KEY'
                        // Configure kubectl to connect to GKE cluster
                        sh "gcloud container clusters get-credentials ${GKE_CLUSTER} --zone ${GKE_ZONE} --project ${GCP_PROJECT}"
                        // Update image tag in deployment.yaml
                        sh "sed -i 's|IMAGE_TAG|${env.BUILD_ID}|g' deployment.yaml"
                        // Apply Kubernetes manifests
                        sh "kubectl apply -f deployment.yaml"

                        sh "kubectl apply -f service.yaml"
                    }
                }
            }
        }
    }
}