pipeline {
    agent any

    tools {
        jdk 'Java17'
        maven 'M398'
    }

    environment {
        DOCKER_IMAGE = "poojadocker404/register-app"
    }

    stages {
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

        stage("Build Docker Image") {
            steps {
                sh 'printenv'
                sh 'docker build -t ${DOCKER_IMAGE}:${GIT_COMMIT} .'
            }
        }

        stage("Push Docker Image") {
            steps {
                withCredentials([string(credentialsId: 'docker-hub-pat', variable: 'DOCKER_PAT')]) {
                    sh '''
                        echo "Logging in to Docker Hub..."
                        echo "$DOCKER_PAT" | docker login -u poojadocker404 --password-stdin
                        
                        echo "Pushing image to Docker Hub..."
                        docker push poojadocker404/register-app:$GIT_COMMIT
                    '''
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                sh """
                    trivy image --exit-code 1 --severity HIGH,CRITICAL \
                    --format json --output trivy-report.json ${DOCKER_IMAGE}:${GIT_COMMIT} || echo "Vulnerabilities found"
                """
            }
        }
    }
}
