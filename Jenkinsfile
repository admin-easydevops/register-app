pipeline {
    agent { label 'Jenkins-agent'}
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "camille94"
        DOCKER_PASS = 'dockerhub-PAT'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"

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
        // stage("Quality gate") {
        //     steps {
        //         script {
        //             waitForQualityGate abortPipeline: true, credentialsId: 'jenkins-sonarqube-PAT'
        //         }

        //     }
        // }
        stage("Build and Push docker image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }
        stage("Trivy scan") {
            steps {
                script {
                    sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image camille94/register-app-pipeline:latest --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }
        }
        stage("Cleanup Artifact") {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
        stage("Trigger CD pipeline") {
            steps {
                script {
                    sh "curl -v -k --user jenkins:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}'
                }
            }
        }

    }
}