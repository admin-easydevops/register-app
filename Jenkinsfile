pipeline {
    agent { label 'Jenkins-agent'}
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    stages{
        stage("Cleanup workspace"){
            steps {
            cleanWs()    
            }
        }
        stage("checkout from SCM"){
            steps {
                git branch: 'feature/jenkins', credentialsId:'github-PAT', url: 'https://github.com/admin-easydevops/register-app.git'
            }
        }
        stage("Build application") {
            steps {
                sh "mvn clean package"

            }
        }
        stage("Test application") {
            steps {
                sh "mvn test"
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-PAT') {
                    sh "mvn sonar:sonar"
                    }
                }
            }
        }
        stage("Quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: true, credentialsId: 'jenkins-sonarqube-PAT'
                }

            }
        }

    }
}