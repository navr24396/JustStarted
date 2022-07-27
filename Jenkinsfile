pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'jdk8'
    }
    stages {
        stage ('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage ('Copy Artifact') {
            steps {
                sh 'cp -r target/*.jar docker'                          #in docker folder dockerfile is present
            }
        }
        stage ('Build docker image and push to dockerhub') {
            steps {
                script {
                    def customImage = docker.build('ashok355/<image name>', "./docker")         #docker build -t ashok355/<image name> <dockerfile path>
                    docker.withRegistry('https://registry.hub.docker.com', '<dockerhub credentials ID>')        #docker hub credentials
                    customImage.push("${env.BUILD_NUMBER}")
                }
            }
        }
        stage ('Deploy to k8s using helm') {
            steps {
                sh 'cp -R helm/* .'                             #copies helm charts to current dir
                sh '/usr/local/bin/helm upgrade --install petclinic-app petclinic --set image.repository=ashok355/<image name> --set image.tag=1'       #optioanl --set
            }
        }
    }

}