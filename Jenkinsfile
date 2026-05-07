pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'DOCKER_USERNAME/my-app'
        DOCKER_TAG = "v${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/yrqurm/my-app.git'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
                sh 'docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest'
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'docker-hub-password', variable: 'DOCKER_PASS')]) {
                    sh 'docker login -u DOCKER_USERNAME -p ${DOCKER_PASS}'
                }
                sh 'docker push ${DOCKER_IMAGE}:${DOCKER_TAG}'
                sh 'docker push ${DOCKER_IMAGE}:latest'
            }
        }
        
        stage('Update K8s Manifests') {
            steps {
                sh "sed -i 's|image: DOCKER_USERNAME/my-app:.*|image: DOCKER_USERNAME/my-app:${DOCKER_TAG}|' k8s/deployment.yaml"
                sh '''
                    git config user.email "jenkins@localhost"
                    git config user.name "Jenkins CI"
                    git add k8s/deployment.yaml
                    git commit -m "Update image to ${DOCKER_TAG}"
                    git push origin main
                '''
            }
        }
    }
}
