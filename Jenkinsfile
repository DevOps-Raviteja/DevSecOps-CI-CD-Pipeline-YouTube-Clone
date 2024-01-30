pipeline {
    agent any
    tools {
        jdk 'JDK-17'
        nodejs 'Nodejs-16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages{
        stage ('Clean Work-Space'){
            steps {
                cleanWs()
            }
        }
        stage ('Git Check-Out'){
            steps{
                git branch: 'main', url: 'https://github.com/DevOps-Raviteja/DevSecOps-CI-CD-Pipeline-YouTube-Clone.git'
            }
        }
        stage ('SonarQube-Analysis'){
            steps{
                withSonarQubeEnv('sonar-server'){
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=YouTube-Clone \
                    -Dsonar.projectKey=YouTube-Clone'''
                }
            }
        }
        stage ('Quality-Gate'){
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
            }
        }
        stage ('Install-Dependency'){
            steps {
                sh 'npm install'
            }
        }
        stage ('Trivy F-System Scan'){
            steps{
                sh 'trivy fs . > trivyFileSystemScanReport.txt'
            }
        }
        stage ('Docker-Build'){
            steps{
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                        sh "docker build -t ravitejadarla5/youtube-clone:version-${BUILD_NUMBER} ."
                    }
                }
            }
        }
        stage ('Trivy Image Scan'){
            steps{
                sh "trivy image ravitejadarla5/youtube-clone:version-${BUILD_NUMBER} > trivyImageScanReport.txt"
            }
        }
        stage ('Docker-Push'){
            steps{
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                        sh "docker tag ravitejadarla5/youtube-clone:version-${BUILD_NUMBER} ravitejadarla5/youtube-clone:latest"
                        sh "docker push ravitejadarla5/youtube-clone:version-${BUILD_NUMBER}"
                        sh 'docker push ravitejadarla5/youtube-clone:latest'
                    }
                }
            }
        }
        stage ('Clear-Artifacts'){
            steps {
                sh "docker image rmi ravitejadarla5/youtube-clone:latest ravitejadarla5/youtube-clone:version-${BUILD_NUMBER}"
            }
        }
        stage ('Deploy-Kubernets'){
            steps {
                script {
                    dir ('Kubernetes'){
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'K8S', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                            sh 'kubectl apply -f deployment.yml'
                            sh 'kubectl apply -f service.yml'
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            emailext attachLog: true, 
            subject: " '${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME} <br/>" + "Build Number: ${env.BUILD_NUMBER}<br/>" + "URL: ${env.BUILD_URL}<br/>",
            to: 'ravitejadarla5@gmail.com'
            attachmentsPattern: 'trivyFileSystemScanReport.txt, trivyImageScanReport.txt' 
        }
    }
}