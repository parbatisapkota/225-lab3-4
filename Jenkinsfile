pipeline {
    agent any 

    environment {
        DOCKER_CREDENTIALS_ID = 'roseaw-dockerhub'
        DOCKER_IMAGE = 'cithit/sapkotp2'
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        GITHUB_URL = 'https://github.com/parbatisapkota/225-lab3-4'
        KUBECONFIG_CREDENTIAL_ID = 'sapkotp2-225-sp26'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: "${GITHUB_URL}"]]])
            }
        }

        stage('Lint HTML') {
            steps {
                sh 'npm install htmlhint --save-dev'
                sh 'npx htmlhint *.html'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to Dev Environment') {
            steps {
                script {
                    withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIAL_ID}", variable: 'KUBECONFIG_FILE')]) {
                        sh "kubectl --kubeconfig=$KUBECONFIG_FILE apply -f deployment-dev.yaml"
                        sh "kubectl --kubeconfig=$KUBECONFIG_FILE get pods --namespace=default"
                    }
                }
            }
        }

        stage('Deploy to Prod Environment') {
            steps {
                script {
                    withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIAL_ID}", variable: 'KUBECONFIG_FILE')]) {
                        sh "kubectl --kubeconfig=$KUBECONFIG_FILE apply -f deployment-prod.yaml"
                        sh "kubectl --kubeconfig=$KUBECONFIG_FILE get pods --namespace=default"
                    }
                }
            }
        }

        stage('Check Kubernetes Cluster') {
            steps {
                script {
                    withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIAL_ID}", variable: 'KUBECONFIG_FILE')]) {
                        sh "kubectl --kubeconfig=$KUBECONFIG_FILE get pods --namespace=default"
                        sh "kubectl --kubeconfig=$KUBECONFIG_FILE get services --namespace=default"
                        sh "kubectl --kubeconfig=$KUBECONFIG_FILE get deploy --namespace=default"
                    }
                }
            }
        }
    }

    post {
        success {
            slackSend color: "good", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
        unstable {
            slackSend color: "warning", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
        failure {
            slackSend color: "danger", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
    }
}
