pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/devopsenthusiastH/Zomato-DevSecOps-Project.git'
            }
        }
        stage('Trivy File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=zomato \
                    -Dsonar.projectKey=zomato '''
                }
            }
        }
        stage('Qualtiy Gates') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP Filesystem Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'owasp-dp-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Docker Build') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred') {
                    sh "docker build -t aakashhandibar/zomato:v2 ."
                    }
                }
            }
        }
        stage('Trivy image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html aakashhandibar/zomato:v2"
            }
        }
        stage('Docker Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred'){
                    sh "docker push aakashhandibar/zomato:v2"
                    }
                }
            }
        }
        stage('Deploy to container') {
            steps {
                sh "docker run -d --name zomato -p 3000:3000 aakashhandibar/zomato:v2"
            }
        }
    }
}

