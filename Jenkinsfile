pipeline {
    agent any

    environment {
        // Set the location for IBM Cloud CLI
        IBMCLOUD_CLI_DIR = "/var/jenkins_home/ibmcloud-cli"
        // Make sure IBM Cloud CLI is included in PATH
        PATH = "${IBMCLOUD_CLI_DIR}/bin:/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
        // IBM Cloud API Key stored in Jenkins credentials
        IBMCLOUD_API_KEY = credentials('ibm-cloud-api-key') // Ensure to store API key securely in Jenkins credentials
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
stage('Install IBM Cloud CLI') {
    steps {
        script {
            echo 'Checking if IBM Cloud CLI is installed...'

            // Check if IBM Cloud CLI is available
            def ibmCloudInstalled = sh(script: 'command -v ibmcloud', returnStatus: true)

            // If not installed, install it using the installation script
            if (ibmCloudInstalled != 0) {
                echo 'IBM Cloud CLI not found. Installing...'
                
                // Use the installation script for IBM Cloud CLI
                sh """
                    curl -fsSL https://clis.cloud.ibm.com/install/linux | bash
                """
            } else {
                echo 'IBM Cloud CLI is already installed.'
            }
        }
    }
}
        stage('Login to IBM Cloud') {
            steps {
                script {
                    echo 'Logging into IBM Cloud...'
                    
                    // Login to IBM Cloud CLI using the API key
                    sh """
                        ibmcloud login --apikey ${IBMCLOUD_API_KEY} -g Default
                    """
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
                withCredentials([string(credentialsId: 'ibm-cloud-api-key', variable: 'IBMCLOUD_API_KEY')]) {
                    script {
                        sh '''
                        # Log in to IBM Cloud with API key
                        ibmcloud login --apikey "$IBMCLOUD_API_KEY" -g Default

                        # Log in to IBM Cloud Container Registry
                        ibmcloud cr login

                        # Push the Docker image to IBM Cloud Registry
                        docker push $REGISTRY_URL/$DOCKER_IMAGE
                        '''
                    }
                }
            }
        }

        stage('Deploy to IBM Cloud') {
            steps {
                echo 'Deploying application to IBM Cloud...'
                // Add your deployment commands here
                // For example: sh 'ibmcloud app push my-app'
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            // Clean up any resources or artifacts
            sh 'rm -rf ibmcloud-cli.tar.gz'
        }
    }
}
