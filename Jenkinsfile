pipeline {
    agent any
    tools {
        jdk 'jdk-17'
        nodejs 'nodejs-16'
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
                        sh "docker build -t youtube-clone:version-${BUILD_NUMBER} ."
                        sh 'docker tag youtube-clone ravitejadarla5/youtube-clone:latest'
                    }
                }
            }
        }
        stage ('Trivy Image Scan'){
            steps{
                sh "trivy image youtube-clone:version-${BUILD_NUMBER} > trivyImageScanReport.txt"
            }
        }
        stage ('Docker-Push'){
            steps{
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                        sh 'docker push ravitejadarla5/youtube-clone:latest'
                        sh "docker push youtube-clone:version-${BUILD_NUMBER} "
                    }
                }
            }
        }
        stage ('Clear-Artifacts'){
            steps {
                sh 'docker image rmi ravitejadarla5/youtube-clone youtube-clone'
            }
        }
    }
}