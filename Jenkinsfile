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
                git branch: 'main', url: 'https://github.com/Ravitejadarla5/DevSecOps-CI-CD-Pipeline-YouTube-Clone.git'
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
        stage ('OWASP-Check'){
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'OWASP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage ('Trivy F-System Scan'){
            steps{
                sh 'trivy fs . > trivyFileSystemScanReport.txt'
            }
        }
        stage ('Docker Build-Push'){
            steps{
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                        sh 'docker build -t youtube-clone .'
                        sh 'docker tag youtube-clone ravitejadarla5/youtube-clone:latest'
                        sh 'docker push ravitejadarla5/youtube-clone:latest'
                    }
                }
            }
        }
        stage ('Trivy Image Scan'){
            steps{
                sh 'trivy image ravitejadarla5/youtube-clone:latest > trivyImageScanReport.txt'
            }
        }
    }






}











































