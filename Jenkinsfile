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
        stage('Install IBM Cloud CLI') {
            steps {
                script {
                    sh '''
                    # Define IBM Cloud CLI directory
                    export IBMCLOUD_CLI_DIR=/var/jenkins_home/ibmcloud-cli
                    export PATH=$IBMCLOUD_CLI_DIR/bin:$PATH

                    # Check if IBM Cloud CLI is already installed
                    if ! command -v ibmcloud &> /dev/null
                    then
                        echo "IBM Cloud CLI not found. Installing..."
                        curl -fsSL https://clis.cloud.ibm.com/download/bluemix-cli/latest/linux64 -o ibmcloud-cli.tar.gz
                        
                        # Check if the file was downloaded correctly
                        if file ibmcloud-cli.tar.gz | grep -q 'gzip compressed data'; then
                            echo "File downloaded successfully, extracting..."
                            mkdir -p $IBMCLOUD_CLI_DIR
                            tar -xzf ibmcloud-cli.tar.gz -C $IBMCLOUD_CLI_DIR --strip-components=1
                            chmod +x $IBMCLOUD_CLI_DIR/bin/ibmcloud
                            export PATH=$IBMCLOUD_CLI_DIR/bin:$PATH
                        else
                            echo "Error: Downloaded file is not in gzip format."
                            exit 1
                        fi
                    else
                        echo "IBM Cloud CLI is already installed."
                    fi

                    # Verify installation
                    ibmcloud --version
                    '''
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
