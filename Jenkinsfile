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
                    sh "docker build -t aakashhandibar/zomato:v1 ."
                    }
                }
            }
        }
        stage('Trivy image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html aakashhandibar/zomato:v1"
            }
        }
        stage('Docker Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred'){
                    sh "docker push aakashhandibar/zomato:v1"
                    }
                }
            }
        }
        stage('Deploy to kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'zomato-sfm', contextName: '', credentialsId: 'kube-secret', namespace: 'zomato', restrictKubeConfigAccess: false, serverUrl: 'https://9D6B2BA7A8966ACFE312ECE75277B58B.gr7.ap-south-1.eks.amazonaws.com') {
                    dir('/var/lib/jenkins/workspace/k8s-manifest/k8smanifests') {
                        sh "kubectl apply -f deployment.yaml"
                        sh "kubectl apply -f service.yaml"
                        sh "kubectl get pods"
                        sh "kubectl get svc"
                    }
                }
            }
        }
    }
}

