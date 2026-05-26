pipeline {
    agent any

    environment {
        // 1. Matches the Credential ID you will create in Jenkins Step 4
        DOCKERHUB_CREDENTIALS = credentials('spandhana-dockerhub-creds') 
        
        // 2. Change 'YOUR_DOCKERHUB_USERNAME' to your real DockerHub username
        DOCKER_IMAGE = "YOUR_DOCKERHUB_USERNAME/devops-calculator"  
        
        IMAGE_TAG = "${BUILD_NUMBER}"
        NAMESPACE = "spandhana-ns"  
    }

    stages {
        stage('Clone Repository') {
            steps {
                // 3. Change to your public GitHub Repository URL
                git url: 'https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPO_NAME.git', 
                    branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $DOCKER_IMAGE:$IMAGE_TAG ."
                }
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                script {
                    sh """
                    echo "$DOCKERHUB_CREDENTIALS_PSW" | docker login -u "$DOCKERHUB_CREDENTIALS_USR" --password-stdin
                    docker push $DOCKER_IMAGE:$IMAGE_TAG
                    docker logout
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                    sed 's|IMAGE_PLACEHOLDER|$DOCKER_IMAGE:$IMAGE_TAG|g' deployment.yaml > k8s-deployment-generated.yaml
                    kubectl apply -f k8s-deployment-generated.yaml
                    echo "Waiting to get the service IP..."
                    sleep 10
                    kubectl get po -n $NAMESPACE
                    kubectl get svc -n $NAMESPACE
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful!'
        }
        failure {
            echo '❌ Deployment failed.'
        }
    }
}