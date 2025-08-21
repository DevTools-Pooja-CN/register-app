pipeline {
    agent any
    tools {
        jdk 'Java17'
        maven 'M398'
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
    }
}
