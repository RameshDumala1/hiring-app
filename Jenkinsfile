pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/RameshDumala1/Hiring-app-argocd.git'
        GIT_BRANCH = 'main'
        GIT_USER = 'RameshDumala1'
        GIT_EMAIL = 'your-email@example.com'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    // Your Docker build steps go here, e.g.:
                    sh 'docker build -t dumalaramesh/hiring-app:latest .'
                }
            }
        }

        stage('Checkout K8S manifest SCM') {
            steps {
                withCredentials([string(credentialsId: 'github-pat', variable: 'GIT_TOKEN')]) {
                    script {
                        echo 'Checking out K8S manifest repository...'
                        sh """
                            git config --global user.name '${GIT_USER}'
                            git config --global user.email '${GIT_EMAIL}'
                            git remote set-url origin https://${GIT_USER}:${GIT_TOKEN}@github.com/${GIT_USER}/Hiring-app-argocd.git
                            git fetch --tags --force --progress -- https://github.com/${GIT_USER}/Hiring-app-argocd.git +refs/heads/*:refs/remotes/origin/*
                            git checkout ${GIT_BRANCH}
                        """
                    }
                }
            }
        }

        stage('Update K8S manifest & push to Repo') {
            steps {
                withCredentials([string(credentialsId: 'git', variable: 'GIT_TOKEN')]) {
                    script {
                        echo 'Updating deployment.yaml file...'
                        sh """
                            echo 'Deployment.yaml before updating:'
                            cat dev/deployment.yaml
                            echo 'Updating Deployment.yaml with build number 8...'
                            sed -i 's|dumalaramesh/hiring-app:[0-35]\\+|dumalaramesh/hiring-app:8|g' dev/deployment.yaml
                            echo 'Deployment.yaml after updating:'
                            cat dev/deployment.yaml

                            git add dev/deployment.yaml
                            git commit -m 'Updated the deployment yaml to Build #8 | Jenkins Pipeline'
                            git push https://${GIT_USER}:${GIT_TOKEN}@github.com/${GIT_USER}/Hiring-app-argocd.git ${GIT_BRANCH}
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
