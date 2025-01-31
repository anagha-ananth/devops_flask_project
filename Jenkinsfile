pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'flask-app'
        REGISTRY_URL = 'icr.io/ibmproject/flask-form-app'
        CLUSTER_NAME = 'minikube'
        DOCKER_CLI = 'docker:latest'
        DOCKER_CONFIG = '/var/jenkins_home/workspace/.docker'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/anagha-ananth/devops_flask_project.git'
            }
        }
        stage('Prepare Workspace') {
            steps {
                script {
                    sh '''
                    # Ensure workspace has correct permissions
                    mkdir -p $DOCKER_CONFIG
                    chmod 700 $DOCKER_CONFIG
                    '''
                }
            }
        }
        stage('Lint Dockerfile') {
            steps {
                script {
                    sh '''
                    # Lint Dockerfile for best practices
                    docker run --rm -i hadolint/hadolint < Dockerfile
                    '''
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.image(DOCKER_CLI).inside('--entrypoint=""') {
                        sh '''
                        # Build Docker image
                        docker --config $DOCKER_CONFIG build -t $REGISTRY_URL/$DOCKER_IMAGE .
                        '''
                    }
                }
            }
        }
                 stage('Push Docker Image') {
    steps {
        withCredentials([usernamePassword(credentialsId: 'docker-registry-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD'),
                         string(credentialsId: 'ibm-cloud-api-key', variable: 'IBMCLOUD_API_KEY')]) {
            script {
                // Use IBM Cloud CLI to log in using the API key
                docker.image('ibmcloud/cli').inside('--entrypoint=""') {
                    sh '''
                    # Log in to IBM Cloud with the API key
                    echo "$IBMCLOUD_API_KEY" | ibmcloud login --apikey "$IBMCLOUD_API_KEY"
                    
                    # Log in to IBM Cloud Container Registry
                    echo "$DOCKER_PASSWORD" | ibmcloud cr login -u $DOCKER_USERNAME --password-stdin
                    
                    # Push the Docker image to IBM Cloud Registry
                    docker --config $DOCKER_CONFIG push $REGISTRY_URL/$DOCKER_IMAGE
                    '''
                }
            }
        }
    }
}
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                    # Apply Kubernetes configurations
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs.'
        }
    }
}
