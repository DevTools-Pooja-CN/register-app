pipeline {
    agent any
    tools {
        jdk 'Java17'
        maven 'M398'
    }

    environment {
        IMAGE_NAME = 'poojadocker404/register-app'
        IMAGE_TAG = "${GIT_COMMIT}"
        DOCKER_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/DevTools-Pooja-CN/register-app'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-qube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-qube-token'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'printenv'
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([string(credentialsId: 'docker-hub-pat', variable: 'DOCKER_PAT')]) {
                    sh """
                        echo "Logging in to Docker Hub..."
                        echo "$DOCKER_PAT" | docker login -u poojadocker404 --password-stdin

                        echo "Pushing image to Docker Hub..."
                        docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                sh """
                trivy image --exit-code 1 --severity HIGH,CRITICAL --format json --output trivy-report.json ${DOCKER_IMAGE} || echo "Vulnerabilities found"
                """
            }
        }
    }
}
