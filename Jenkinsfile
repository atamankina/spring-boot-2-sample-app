def ENV = "test"
def UNIQUE_POD_LABEL = "jenkins-pod-${UUID.randomUUID().toString()}"
def PROJECT_NAME = "spring-boot-app"
def IMAGE_NAME = "${PROJECT_NAME}-${ENV}"
def SERVICE_IP = ""

pipeline {
    agent {
        kubernetes {
            label UNIQUE_POD_LABEL
            defaultContainer 'jnlp'
            yamlFile 'jenkins-pods.yaml'
        }
    }

    stages {
        stage('Maven build') {
            steps {
                container('maven') {
                    script {
                        sh "mvn -B -DskipTests clean package"
                    }
                }
            }
        }

        stage('Unit tests') {
            steps {
                container('maven') {
                    script {
                        sh "mvn test"
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

        stage('Integration tests') {
            steps {
                container('kubectl') {
                    script {
                        SERVICE_IP = sh(
                            script: 'kubectl get svc spring-boot-service -o jsonpath="{.status.loadBalancer.ingress[*].ip}"',
                            returnStdout: true
                        ).trim()
                    }
                }
                container('maven') {
                    script {
                        sh "mvn verify -Dserver.host=http://${SERVICE_IP}:8080/"
                    }
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
