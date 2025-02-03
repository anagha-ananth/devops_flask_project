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
                        docker --config $DOCKER_CONFIG build -t $REGISTRY_URL/$DOCKER_IMAGE .
                        '''
                    }
                }
            }
        }
       stage('Push Docker Image') {
    steps {
        withCredentials([string(credentialsId: 'ibm-cloud-api-key', variable: 'IBMCLOUD_API_KEY')]) {
            script {
                sh '''
                export PATH=$HOME/ibmcloud-cli/Bluemix_CLI/bin:$PATH
                
                # Log in to IBM Cloud
                ibmcloud login --apikey "$IBMCLOUD_API_KEY" -r in-che || exit 1

                # Ensure IBM Cloud Container Registry plugin is installed
                ibmcloud plugin install container-registry || echo "Plugin already installed"

                # Log in to IBM Cloud Container Registry
                ibmcloud cr login || exit 1

                # Verify authentication before pushing
                ibmcloud cr info

                # Push the Docker image to IBM Cloud Registry
                docker push $REGISTRY_URL/$DOCKER_IMAGE
                '''
            }
        }
    }
}
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
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
