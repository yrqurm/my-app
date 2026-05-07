pipeline {
    agent any
    
    environment {
        REGISTRY = '192.168.1.18:5000'
        IMAGE_NAME = 'my-app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/yrqurm/my-app.git',
                    credentialsId: 'github-credentials'
            }
        }
        
        stage('Build Image') {
            steps {
                sh """
                    docker build -t ${REGISTRY}/${IMAGE_NAME}:v${BUILD_NUMBER} .
                    docker tag ${REGISTRY}/${IMAGE_NAME}:v${BUILD_NUMBER} ${REGISTRY}/${IMAGE_NAME}:latest
                """
            }
        }
        
        stage('Push to Registry') {
            steps {
                sh """
                    docker push ${REGISTRY}/${IMAGE_NAME}:v${BUILD_NUMBER}
                    docker push ${REGISTRY}/${IMAGE_NAME}:latest
                """
            }
        }
        
        stage('Update Manifests') {
            steps {
                sh """
                    sed -i 's|${REGISTRY}/${IMAGE_NAME}:.*|${REGISTRY}/${IMAGE_NAME}:v${BUILD_NUMBER}|' k8s/deployment.yaml
                    git config user.email "jenkins@localhost"
                    git config user.name "Jenkins CI"
                    git add k8s/deployment.yaml
                    git diff --cached --quiet || git commit -m "Update image to v${BUILD_NUMBER} [skip ci]"
                """
                withCredentials([string(credentialsId: 'github-token', variable: 'TOKEN')]) {
                    sh "git push https://yrqurm:${TOKEN}@github.com/yrqurm/my-app.git main"
                }
            }
        }
    }
}
