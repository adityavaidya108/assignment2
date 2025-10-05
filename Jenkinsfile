pipeline {
    agent any

    environment {
        DOCKERHUB_CREDS = 'dockerhub-creds'   // DockerHub credentials ID
        KUBECONFIG_FILE = 'kubeconfig-id'     // Kubeconfig file credential ID
        IMAGE_NAME = 'adityavaidya108/static-web-app'
        GITHUB_REPO = 'https://github.com/adityavaidya108/assignment2'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: "${GITHUB_REPO}", branch: 'master', credentialsId: 'github-pat'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Unique tag for this build
                    def UNIQUE_TAG = "${IMAGE_NAME}:${BUILD_NUMBER}"
                    def LATEST_TAG = "${IMAGE_NAME}:latest"

                    echo "Building Docker image with tags: ${UNIQUE_TAG} and ${LATEST_TAG}"
                    docker.build(UNIQUE_TAG, "--no-cache .")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKERHUB_CREDS}") {
                        echo "Pushing image with unique tag..."
                        docker.image("${IMAGE_NAME}:${BUILD_NUMBER}").push()
                        echo "Tagging and pushing latest..."
                        docker.image("${IMAGE_NAME}:${BUILD_NUMBER}").push("latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: "${KUBECONFIG_FILE}", variable: 'KUBECONFIG_PATH')]) {
                    script {
                        echo "Updating Kubernetes deployment to use new image..."
                        sh """
                        kubectl --kubeconfig=$KUBECONFIG_PATH set image deployment/mywebapp \
                            mywebapp=${IMAGE_NAME}:${BUILD_NUMBER} --record
                        kubectl --kubeconfig=$KUBECONFIG_PATH rollout status deployment/mywebapp
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Deployed build #${BUILD_NUMBER}."
        }
    }
}
