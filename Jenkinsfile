pipeline {
    agent any
    
    
    environment {
        SCANNER_HOME= tool 'sonarqube-scanner'
    }

    stages {
        stage('Git') {
            steps {
                git 'https://github.com/adikesavanaidug2404/boardgame.git'
            }
        }
        stage('Maven') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Trivy File Scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        stage('S3 Bucket Backup') {
            steps {
                sh 'aws s3 cp /var/lib/jenkins/workspace/boardgame/ s3://myprojectsbucker/boardgamefiles/ --recursive'
            }
        }
        stage('Docker Build&Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker build -t adikesavanaidug2404/boardgame:latest .'
                        
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --format table -o trivy-image-report.html adikesavanaidug2404/boardgame:latest '
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker push adikesavanaidug2404/boardgame:latest'
                        
                    }
                }
            }
        }
        stage('Deploy To Docker Container') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker run -d -p 8083:8080 adikesavanaidug2404/boardgame:latest'
                        
                    }
                }
            }
        }
        stage('Deploy To Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://10.0.12.59:6443') {
                    sh 'kubectl apply -f deployment-service.yaml -n webapps'
                    sh 'kubectl get svc -n webapps'
                }
            }
        }
    }
}

