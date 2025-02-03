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
                withCredentials([usernamePassword(credentialsId: 'docker-registry-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD'),
                                 string(credentialsId: 'ibm-cloud-api-key', variable: 'IBMCLOUD_API_KEY')]) {
                    script {
                        docker.image('icr.io/ibm/ibmcloud-cli:latest').inside('--entrypoint=""') {
                            sh '''
                            ibmcloud login --apikey "$IBMCLOUD_API_KEY"
                            ibmcloud cr login
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
