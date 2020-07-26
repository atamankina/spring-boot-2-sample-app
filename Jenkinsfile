def ENV = "test"
def UNIQUE_POD_LABEL = "jenkins-pod-${UUID.randomUUID().toString()}"
def PROJECT_NAME = "spring-boot-app"
def IMAGE_NAME = "${PROJECT_NAME}-${ENV}"
def BUILD_NAME = "Build-#${BUILD_NUMBER}-${ENV}"

pipeline {
    agent {
        kubernetes {
            label UNIQUE_POD_LABEL
            defaultContainer 'jnlp'
            yamlFile 'jenkins-pods.yaml'
        }
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Maven build') {
                    steps {
                        container('maven') {
                            script {
                                sh "mvn -B -DskipTests clean package"
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
                        sh "docker push atamankina/${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Create/update kubernetes deployment') {
            steps {
                container('kubectl') {
                        sh "kubectl get ns"
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
