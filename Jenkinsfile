pipeline {
    agent any
    tools {
        maven 'maven-3.8.5'
    }
    environment {
        DOCKER_TAG = "getVersion"
    }
    stages {
        stage ('Git Checkout') {
             git 'https://github.com/navr24396/hello-world.git'
        }
        stage ("Maven Build") {
            steps {
                sh "mvn clean package"
            }
        }
        stage ("Email Notification") {
            steps {
                emailext body: 'BuILD IS SUCCESSFULL', recipientProviders: [developers()], subject: 'Build Success', to: 'navr243@gmail.com'
            }
        }
        stage ('SonarQube analysis') {
           steps {
                withSonarQubeEnv('sonar6')
                sh 'mvn sonar:sonar'
           }
        }
        stage ("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS')
                waitForQualityGate abortPipeline: true

            }
        }
        stage ("Artifactory configurtion") {
            steps {
                rtMavenDeployer (
                    id: 'maven_deployer',
                    serverId: 'jfrog',
                    releaseRepo: 'libs-release-local',
                    snapshotRepo: 'libs-snapshot-local'
                )
            }
        }
        stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: 'jfrog'
                )
            }
        }
        stage ("Deploy war/ear to a container - tomcat") {
            steps {
               deploy adapters: [tomcat9(credentialsId: 'tomcat-server', path: '', url: 'http://52.206.189.28:8080')], contextPath: null, war: '**/*.war'
            }
        }
        stage ("Docker Build") {
            steps {
                sh 'docker image build -t ashok355/***:${DOCKER_TAG}'
            }
        }
        stage ("Docker Push") {
            steps {
                withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerHubPwd')]) {
                sh 'docker login -u ashok355 -p ${dockerHubPwd}'
                }
                sh 'docker push ashok355/***:${DOCKER_TAG}'
            }
        }
        stage ('Docker Deploy') {
            ansiblePlaybook credentialsId: 'dev-server', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
            }
        }
        stage ('k8s deploy') {
            kubernetesDeploy(
                configs: 'MyAwesomeApp/springboot-lb.yaml',
                    kubeconfigId: 'K8S',
                    enableConfigSubstitution: true
            )
                    
        }
    }
}

def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}

        }
    }
}   

def getVersion() {
    def commitHash = sh returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}    




Plugins used
git = git: Git
maven = directly wrote 
mail = emailtext: Extended email
sonar = search for sonarqube jenkinsfile, official documentation
artifactory = search for jfrog artifactory jenkinsfile, official documentation
tomcat = deploy: Deploy war/ear to a container
docker build = directly wrote 
docker push = withCredentials: bind credentials to variables
docker deploy = ansiblePlaybook: Invoke an ansible playbook
tools = click Generate Declarative Directive and select tools
Environment  = click Generate Declarative Directive and select tools
k8s = kubernetesDeploy: Deploy to kubernetes