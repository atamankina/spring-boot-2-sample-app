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
                    sh "mvn -B -DskipTests clean package"
                }
            }
        }

        stage('Unit tests') {
            steps {
                container('maven') {
                    sh "mvn test"
                }
            }
        }

        stage('Docker build') {
            steps {
                container('docker') {
                    sh "docker build -t ${IMAGE_NAME}:latest ."
                    sh "docker tag ${IMAGE_NAME}:latest atamankina/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Docker push') {
            steps {
                container('docker') {
                    withDockerRegistry([ credentialsId: "gal-dockerhub", url: "" ]){
                        sh "docker push atamankina/${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('Create/update kubernetes deployment') {
            steps {
                container('kubectl') {
                    sh "kubectl apply -f k8s"
                    sh "kubectl rollout restart deployment spring-boot-deployment"
                }
            }
        }

        stage('Check availability') {
            steps {
                sh 'sleep 90'
                container('kubectl'){
                    script{
                        SERVICE_IP = sh(
                            script: 'kubectl get svc spring-boot-service -o jsonpath="{.status.loadBalancer.ingress[*].ip}"',
                            returnStdout: true
                        ).trim()
                    }
                }
                script{
                    waitUntil {
                        try {
                            sh "curl -s --head --request GET http://${SERVICE_IP}:8080/actuator/health | grep '200'"
                            return true
                        } catch (Exception e) {
                            return false
                        }
                    }
                }
            }
        }

        stage('Integration tests') {
            steps {
                container('maven') {
                    sh "mvn verify -Dserver.host=http://${SERVICE_IP}:8080/"
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
