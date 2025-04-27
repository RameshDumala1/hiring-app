pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"  // Explicitly use BUILD_NUMBER for Docker image tag
    }

    stages {
        stage('Docker Build') {
            steps {
                sh "docker build . -t dumalaramesh/hiring-app:${env.IMAGE_TAG}"  // Using env.IMAGE_TAG
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub', variable: 'DOCKER_PASSWORD')]) {
                    // Secure Docker login with Secret Text credentials
                    sh "echo ${DOCKER_PASSWORD} | docker login -u dumalaramesh --password-stdin"
                    sh "docker push dumalaramesh/hiring-app:${env.IMAGE_TAG}"  // Using env.IMAGE_TAG
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
                        sh '''
                        cat /var/lib/jenkins/workspace/$JOB_NAME/dev/deployment.yaml
                        sed -i "s/5/${BUILD_NUMBER}/g" /var/lib/jenkins/workspace/$JOB_NAME/dev/deployment.yaml
                        cat /var/lib/jenkins/workspace/$JOB_NAME/dev/deployment.yaml
                        git add .
                        git commit -m 'Updated the deploy yaml | Jenkins Pipeline'
                        git remote -v
                        git push https://$GIT_USERNAME:$GIT_PASSWORD@github.com/RameshDumala1/Hiring-app-argocd.git main
                        '''
                    }
                }
            }
        }
    }
}
