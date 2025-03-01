// Pipeline version: v1.0.22
pipeline {
    agent { label 'jenkins-jenkins-agent' }

    environment {
        IMAGE_NAME      = "d4rkghost47/python-circuit-svc-1v2"
        REGISTRY        = "https://index.docker.io/v1/"
        SHORT_SHA       = "${GIT_COMMIT[0..7]}"
        SONAR_PROJECT   = "python-circuit-svc-1v2"
        SONAR_HOST      = "http://sonarqube-sonarqube.sonarqube.svc.cluster.local:9000"
        TRIVY_HOST      = "http://trivy.trivy-system.svc.cluster.local:4954"
        TZ              = "America/Guatemala"  
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    echo "[CHECKOUT] [INFO] ${getTimestamp()} - Starting code checkout..."
                    checkout scm
                    echo "[CHECKOUT] [SUCCESS] ${getTimestamp()} - Code checkout completed."
                }
            }
        }


        stage('Static Code Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    script {
                        def startTime = System.currentTimeMillis()
                        echo "[ANALYSIS] [INFO] ${getTimestamp()} - Running static code analysis with SonarQube..."

                        sh """
                        sonar-scanner \
                            -Dsonar.projectKey=${env.SONAR_PROJECT} \
                            -Dsonar.sources=app \
                            -Dsonar.language=py \
                            -Dsonar.host.url=${env.SONAR_HOST} \
                            -Dsonar.login=\$SONAR_TOKEN
                        """

                        def duration = (System.currentTimeMillis() - startTime) / 1000
                        echo "[ANALYSIS] [SUCCESS] ${getTimestamp()} - SonarQube analysis completed in ${duration} seconds."
                    }
                }
            }
        }


        stage('Build Image') {
            steps {
                container('dind') {
                    script {
                        echo "[BUILD] [INFO] ${getTimestamp()} - Building Docker image..."
                        sh """
                        docker build -t ${IMAGE_NAME}:${env.SHORT_SHA} .
                        docker tag ${IMAGE_NAME}:${env.SHORT_SHA} ${IMAGE_NAME}:latest
                        """
                        echo "[BUILD] [SUCCESS] ${getTimestamp()} - Build completed."
                    }
                }
            }
        }


        stage('Security Scan') {
            steps {
                script {
                    echo "[SECURITY SCAN] [INFO] ${getTimestamp()} - Running Trivy security scan..."
                    sh """
                    trivy client --remote ${TRIVY_HOST} image ${IMAGE_NAME}:${env.SHORT_SHA} --severity HIGH,CRITICAL
                    """
                    echo "[SECURITY SCAN] [SUCCESS] ${getTimestamp()} - Security scan completed."
                }
            }
        }
    
    
        stage('Push Image') {
            steps {
                container('dind') {
                    script {
                        withCredentials([string(credentialsId: 'docker-token', variable: 'DOCKER_TOKEN')]) {
                            echo "[PUSH] [INFO] ${getTimestamp()} - Uploading Docker image..."
                            sh """
                            echo "\$DOCKER_TOKEN" | docker login -u "d4rkghost47" --password-stdin > /dev/null 2>&1
                            docker push ${IMAGE_NAME}:${env.SHORT_SHA}
                            docker push ${IMAGE_NAME}:latest
                            """
                            echo "[PUSH] [SUCCESS] ${getTimestamp()} - Image pushed successfully."
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "[PIPELINE] [SUCCESS] ${getTimestamp()} - Pipeline completed successfully."
        }
        failure {
            echo "[PIPELINE] [FAILURE] ${getTimestamp()} - Pipeline failed."
        }
    }
}

def getTimestamp() {
    return sh(script: "TZ='America/Guatemala' date '+%Y-%m-%d %H:%M:%S'", returnStdout: true).trim()
}

