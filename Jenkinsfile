def ENV = "test"
def UNIQUE_POD_LABEL = "jenkins-pod-${UUID.randomUUID().toString()}"
def PROJECT_NAME = "spring-boot-app"
def IMAGE_NAME = "${PROJECT_NAME}-${ENV}"

pipeline {
    agent {
        kubernetes {
            label UNIQUE_POD_LABEL
            defaultContainer 'jnlp'
            yamlFile 'jenkins-pods.yaml'
        }
    }

    stages {
        stage('Maven build/test') {
            steps {
                container('maven') {
                    script {
                        sh "mvn package"
                    }
                }
            }
        }

        stage('Docker build') {
            steps {
                container('docker') {
                    script {
                        sh "docker build -t ${IMAGE_NAME}:latest ."
                        sh "docker tag ${IMAGE_NAME}:latest atamankina/${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('Docker push') {
            steps {
                container('docker') {
                    script {
                        withDockerRegistry([ credentialsId: "gal-dockerhub", url: "" ]){
                            sh "docker push atamankina/${IMAGE_NAME}:latest"
                        }
                    }
                }
            }
        }

        stage('Create/update kubernetes deployment') {
            steps {
                container('kubectl') {
                        sh "kubectl apply -f k8s"
                }
            }
        }
    }

    post {
        aborted {
            echo 'Pipeline aborted.'
        }
        failure {
            echo 'Pipeline failed.'
        }
        success {
            echo 'Built succesfully.'
        }
    }
}
