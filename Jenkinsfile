pipeline {
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment{
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/KollaChandini/Task-Master-Pro.git'
            }
        }
        
        stage('compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('Unit-Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=taskmaster \
                    -Dsonar.projectName=taskmaster \
                    -Dsonar.java.binaries=target '''
                }
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --format table -o fs.html .'
            }
        }
        
        stage('Build Application') {
            steps {
                sh 'mvn package'
            }
        }
        
        stage('Publish Artifact') {
            steps {
                withMaven(globalMavenSettingsConfig: 'setting-xml', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy'
                }
            }
        }
        
        stage('Docker Build & Tag') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker build -t kollachandini/taskmaster:latest .'
                    }
                }
            }
        }
        
        stage('Trivy image Scan') {
            steps {
                sh 'trivy image --format table -o image.html kollachandini/taskmaster:latest'
            }
        }
        
        stage('Docker Push Image') {
            steps {
                 script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker push kollachandini/taskmaster:latest'
                    }
                }
            }
        }
        
        stage('Deploy To Kubernetes') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: ' devopsshack-cluster', contextName: '', credentialsId: 'k8', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://xxxxxx') {
                    sh 'kubectl apply -f deployment-service.yml'
                    sleep 30
                }
            }
        }
        
        stage('Verify K8 Deployment') {
            steps {
                 withKubeConfig(caCertificate: '', clusterName: ' devopsshack-cluster', contextName: '', credentialsId: 'k8', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://xxxxxx') {
                    sh 'kubectl get pods -n webapps'
                    sh 'kubectl get svc -n webapps'
                }
            }
        }
    }
}
