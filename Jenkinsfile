pipeline {
    agent { label 'docker-agent' }
    environment {
        IMAGE_NAME = "shridhar8899/devops-project"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }
    stages {
        stage('Clone Repo') {
            steps {
                deleteDir()
                sh 'git clone https://github.com/biradarshashank413-sudo/devops-project.git .'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
            }
        }
        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh "echo \$PASS | docker login -u \$USER --password-stdin"
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }
        stage('Deploy to GKE') {
            steps {
                withCredentials([file(
                    credentialsId: '113962110009254531075',
                    variable: 'GCP_KEY'
                )]) {
                    sh """
                        gcloud auth activate-service-account --key-file=\$GCP_KEY
                        gcloud config set project jenkins-doc
                        gcloud container clusters get-credentials cluster-jenkins \
                            --zone us-central1-c
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                        kubectl set image deployment/flask-app \
                            flask-app=${IMAGE_NAME}:${IMAGE_TAG}
                        kubectl rollout status deployment/flask-app
                    """
                }
            }
        }
    }
    post {
        success { echo '✅ Deployed to GKE!' }
        failure { echo '❌ Build failed!' }
    }
}
