pipeline {
    agent { label 'AWS-Slave' }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def dockerImage = "mujimmy/sample-app:${env.BRANCH_NAME}"
                    sh "docker build -t ${dockerImage} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        def dockerImage = "mujimmy/sample-app:${env.BRANCH_NAME}"
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        sh "docker push ${dockerImage}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def namespace = ''

                    if (env.BRANCH_NAME == 'dev') {
                        namespace = 'dev'
                    } else if (env.BRANCH_NAME == 'staging') {
                        namespace = 'staging'
                    } else if (env.BRANCH_NAME == 'production') {
                        namespace = 'production'
                    }

                    withCredentials([file(credentialsId: 'kind-kubeconfig', variable: 'KUBECONFIG')]) {
                        def dockerImage = "mujimmy/sample-app:${env.BRANCH_NAME}"
                        sh "export KUBECONFIG=${KUBECONFIG}"
                        sh "kubectl set image deployment/sample-app sample-app=${dockerImage} --namespace=${namespace}"
                        sh "kubectl apply -f service.yaml --namespace=${namespace}"
                    }
                }
            }
        }
    }
