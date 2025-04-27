pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Docker Build') {
            steps {
                sh "docker build . -t dumalaramesh/hiring-app:${BUILD_NUMBER}"
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub', variable: 'auth')]) {
                    sh """
                    echo "${auth}" | docker login -u dumalaramesh --password-stdin
                    docker push dumalaramesh/hiring-app:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Checkout K8S manifest SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/RameshDumala1/Hiring-app-argocd.git'
            }
        }

        stage('Update K8S manifest & push to Repo') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'Github_server', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh """
                        echo "Deployment.yaml before updating:"
                        cat /var/lib/jenkins/workspace/$JOB_NAME/dev/deployment.yaml
                        
                        echo "Updating Deployment.yaml with build number ${BUILD_NUMBER}..."
                        sed -i "s|dumalaramesh/hiring-app:[0-35]\\+|dumalaramesh/hiring-app:${BUILD_NUMBER}|g" dev/deployment.yaml
                        
                        echo "Deployment.yaml after updating:"
                        cat /var/lib/jenkins/workspace/$JOB_NAME/dev/deployment.yaml
                        
                        git config --global user.email "jenkins@example.com"
                        git config --global user.name "Jenkins Pipeline"
                        
                        git add dev/deployment.yaml
                        git commit -m "Updated the deployment yaml to Build #${BUILD_NUMBER} | Jenkins Pipeline"
                        git remote -v
                        git push https://$GIT_USERNAME:$GIT_PASSWORD@github.com/RameshDumala1/Hiring-app-argocd.git main
                        """
                    }
                }
            }
        }
    }
}
