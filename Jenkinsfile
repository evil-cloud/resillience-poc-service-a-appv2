// Pipeline version: 1.0.12
// Pipeline version: 1.0.17
pipeline {
    agent { label 'jenkins-jenkins-agent' }

    environment {
        IMAGE_NAME      = "d4rkghost47/python-circuit-svc-1v2"
        REGISTRY        = "https://index.docker.io/v1/"
        SHORT_SHA       = "${GIT_COMMIT[0..7]}"
        ARGOCD_APP_NAME = "python-svc-1v2"
        SONAR_PROJECT   = "python-circuit-svc-1v2"
        SONAR_HOST      = "http://sonarqube-sonarqube.sonarqube.svc.cluster.local:9000"
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    def startTime = System.currentTimeMillis()
                    echo "[CHECKOUT] [INFO] $(date '+%Y-%m-%d %H:%M:%S') - Starting code checkout..."
                    checkout scm
                    def endTime = System.currentTimeMillis()
                    def duration = (endTime - startTime) / 1000
                    echo "[CHECKOUT] [SUCCESS] $(date '+%Y-%m-%d %H:%M:%S') - Code checkout completed in ${duration} seconds."
                }
            }
        }

        stage('Build Image') {
            steps {
                container('dind') {
                    script {
                        def startTime = System.currentTimeMillis()
                        echo "[BUILD] [INFO] $(date '+%Y-%m-%d %H:%M:%S') - Building Docker image..."
                        sh """
                        docker build -t ${IMAGE_NAME}:${env.SHORT_SHA} . || { echo "[BUILD] [ERROR] $(date '+%Y-%m-%d %H:%M:%S') - Build failed."; exit 1; }
                        docker tag ${IMAGE_NAME}:${env.SHORT_SHA} ${IMAGE_NAME}:latest
                        """
                        def endTime = System.currentTimeMillis()
                        def duration = (endTime - startTime) / 1000
                        echo "[BUILD] [SUCCESS] $(date '+%Y-%m-%d %H:%M:%S') - Build completed in ${duration} seconds."
                    }
                }
            }
        }

        stage('Static Code Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    script {
                        def startTime = System.currentTimeMillis()
                        echo "[ANALYSIS] [INFO] $(date '+%Y-%m-%d %H:%M:%S') - Running static code analysis with SonarQube..."
                        sh """
                        sonar-scanner \
                            -Dsonar.projectKey=${env.SONAR_PROJECT} \
                            -Dsonar.sources=app \
                            -Dsonar.language=py \
                            -Dsonar.host.url=${env.SONAR_HOST} \
                            -Dsonar.login=\$SONAR_TOKEN || { echo "[ANALYSIS] [ERROR] $(date '+%Y-%m-%d %H:%M:%S') - SonarQube analysis failed."; exit 1; }
                        """
                        def endTime = System.currentTimeMillis()
                        def duration = (endTime - startTime) / 1000
                        echo "[ANALYSIS] [SUCCESS] $(date '+%Y-%m-%d %H:%M:%S') - SonarQube analysis completed in ${duration} seconds."
                    }
                }
            }
        }

        stage('Push Image') {
            steps {
                container('dind') {
                    script {
                        withCredentials([string(credentialsId: 'docker-token', variable: 'DOCKER_TOKEN')]) {
                            def startTime = System.currentTimeMillis()
                            echo "[PUSH] [INFO] $(date '+%Y-%m-%d %H:%M:%S') - Uploading Docker image to registry..."
                            sh """
                            echo "\$DOCKER_TOKEN" | docker login -u "d4rkghost47" --password-stdin > /dev/null 2>&1
                            
                            docker push ${IMAGE_NAME}:${env.SHORT_SHA} || { echo "[PUSH] [ERROR] $(date '+%Y-%m-%d %H:%M:%S') - Failed to push SHA-tagged image."; exit 1; }
                            docker push ${IMAGE_NAME}:latest || { echo "[PUSH] [ERROR] $(date '+%Y-%m-%d %H:%M:%S') - Failed to push latest-tagged image."; exit 1; }
                            """
                            def endTime = System.currentTimeMillis()
                            def duration = (endTime - startTime) / 1000
                            echo "[PUSH] [SUCCESS] $(date '+%Y-%m-%d %H:%M:%S') - Image pushed successfully in ${duration} seconds."
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "[PIPELINE] [SUCCESS] $(date '+%Y-%m-%d %H:%M:%S') - Pipeline completed successfully."
        }
        failure {
            echo "[PIPELINE] [FAILURE] $(date '+%Y-%m-%d %H:%M:%S') - Pipeline failed."
        }
    }
}

