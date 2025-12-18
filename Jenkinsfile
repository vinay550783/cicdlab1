pipeline {
    agent any

    environment {
        IMAGE_NAME = "vinayapp/cicdexample"
        TAG = "035"

        // Jenkins credentials IDs
        DOCKER_CREDS = "dockerhub-creds"
        KUBE_CONFIG  = "kubeconfig-file"

        // GitHub Repo
        GIT_URL = "https://github.com/vinay550783/cicdlab1.git"
    }

    stages {

        stage('Checkout Code from GitHub') {
            steps {
                git branch: "main", url: "${GIT_URL}"
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %IMAGE_NAME%:%TAG% ."
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: DOCKER_CREDS,
                    usernameVariable: "DOCKER_USER",
                    passwordVariable: "DOCKER_PASS"
                )]) {

                    bat '''
                    docker login -u %DOCKER_USER% -p %DOCKER_PASS%
                    docker push %IMAGE_NAME%:%TAG%
                    docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: KUBE_CONFIG, variable: "KUBECONFIG")]) {

                    bat '''
                    echo ===== Checking kubeconfig =====
                    set KUBECONFIG=%KUBECONFIG%

                    kubectl config current-context
                    kubectl get nodes

                    kubectl apply -f k8s.yaml --validate=false
                    kubectl rollout restart deployment static-web-deployment
                    '''
                }
            }
        }
    }
}
