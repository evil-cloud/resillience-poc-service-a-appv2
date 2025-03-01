// Pipeline - v1.0.7
pipeline {
    agent { label 'jenkins-jenkins-agent' }

    environment {
        IMAGE_NAME = "d4rkghost47/python-circuit-svc-1v2"
        REGISTRY = "https://index.docker.io/v1/"
        SHORT_SHA = "${GIT_COMMIT[0..7]}"
        ARGOCD_APP_NAME = "python-svc-1v2"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                container('dind') {
                    script {
                        echo "🐳 Construyendo imagen con SHA: ${env.SHORT_SHA}"
                        sh """
                        docker build -t ${IMAGE_NAME}:${env.SHORT_SHA} .
                        docker tag ${IMAGE_NAME}:${env.SHORT_SHA} ${IMAGE_NAME}:latest
                        """
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                container('dind') {
                    script {
                        withCredentials([string(credentialsId: 'docker-token', variable: 'DOCKER_TOKEN')]) {
                            sh """
                            echo "\$DOCKER_TOKEN" | docker login -u "d4rkghost47" --password-stdin
                            docker push ${IMAGE_NAME}:${env.SHORT_SHA}
                            docker push ${IMAGE_NAME}:latest
                            """
                        }
                    }
                }
            }
        }

        stage('Update Image in ArgoCD') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'argocd-token', variable: 'ARGOCD_TOKEN')]) {
                        sh """
                        argocd login argocd.infraops.us --username admin --password \$ARGOCD_TOKEN --grpc-web
                        argocd app set ${ARGOCD_APP_NAME} --parameter image.tag=${SHORT_SHA}
                        argocd app sync ${ARGOCD_APP_NAME}
                        """
                    }
                }
            }
        }
    }
}
