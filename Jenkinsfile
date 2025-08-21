pipeline {
    agent any

    tools {
        jdk 'Java17'
        maven 'M398'
    }

    environment {
        IMAGE_NAME = 'poojadocker404/register-app'
        IMAGE_TAG = "${env.GIT_COMMIT}"
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
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([string(credentialsId: 'docker-hub-pat', variable: 'DOCKER_PAT')]) {
                    sh '''
                        echo "$DOCKER_PAT" | docker login -u poojadocker404 --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest
                    '''
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    docker pull aquasec/trivy:latest
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                        -v $(pwd):/root/.cache/ \
                        aquasec/trivy:latest image \
                        --exit-code 1 \
                        --severity HIGH,CRITICAL \
                        --format json \
                        --output trivy-report.json \
                        ${IMAGE_NAME}:latest || echo "Vulnerabilities found"
                '''
            }
        }
    }
}
