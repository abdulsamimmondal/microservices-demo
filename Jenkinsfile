pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials')
        DOCKER_HUB_USER = 'samimmondal'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                script {
                    def services = ['order-service', 'product-service', 'user-service']
                    services.each { svc ->
                        dir(svc) {
                            sh 'mvn clean install'
                        }
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'main'
                }
            }
            steps {
                script {
                    def services = ['order-service', 'product-service', 'user-service']
                    services.each { svc ->
                        def imageName = "${DOCKER_HUB_USER}/${svc}:1.0"
                        dir(svc) {
                            sh "docker build -t ${imageName} ."
                            withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                                sh "docker push ${imageName}"
                            }
                        }
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                branch 'main'
            }
            steps {
                script {
                    def services = ['order-service', 'product-service', 'user-service']
                    services.each { svc ->
                        dir("${svc}/k8s") {
                            sh "kubectl apply -f deployment.yaml"
                            sh "kubectl apply -f service.yaml"
                        }
                    }
                }
            }
        }
    }
}

