pipeline {
    agent any
    
    tools {
        nodejs 'nodejs23'
    }

    environment {
       SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'deploy-to-dev-k8', url: 'https://github.com/sujayt-ghub/jaiswaladi246-3-Tier-DevSecOps-Mega-Project.git'
            }
        }
        
        stage('Frontend Compilation') {
            steps {
                dir('client') {
                    sh 'find . -name "*.js" -exec node --check {} +'
                }
            }
        }
        
        stage('Backend Compilation') {
            steps {
                dir('api') {
                    sh 'find . -name "*.js" -exec node --check {} +'
                }
            }
        }
        
        stage('GitLeaks Scan') {
            steps {
                //sh 'gitleaks detect --owner-path ./client'    ##v6.2 gitleaks working command
                //sh 'gitleaks detect --owner-path ./api'   ##v6.2 gitleaks working command
                //sh 'gitleaks detect --uncommitted'  ##v6.2 gitleaks working command
                sh 'gitleaks detect --source ./client --exit-code 1'
                sh 'gitleaks detect --source ./api --exit-code 1'
                sh 'gitleaks detect . -v'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=NodeJS-Project \
                            -Dsonar.projectKey=NodeJS-Project '''
                }
            }
        }
        stage('Quality Gate Check') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --format table -o fs-report.html .'
            }
        }
        stage('Build-Tag & Push Backend Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        dir('api') {
                            sh 'docker build -t sujaysuj/backend:latest .'
                            sh 'trivy image --format table -o backend-image-report.html sujaysuj/backend:latest '
                            sh 'docker push sujaysuj/backend:latest'
                           
                        }
                    }
                }
            }
        }  
            
        stage('Build-Tag & Push Frontend Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        dir('client') {
                            sh 'docker build -t sujaysuj/frontend:latest .'
                            sh 'trivy image --format table -o frontend-image-report.html sujaysuj/frontend:latest '
                            sh 'docker push sujaysuj/frontend:latest'
                        }
                    }
                }
            }
             
        }  
        stage('kubernetes deployment') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubeconfig', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                      //sh 'kubectl get pods -A'
                      sh 'kubectl apply -f k8s/mysql.yaml'
                      sh 'kubectl apply -f k8s/backend.yaml'
                      sh 'kubectl apply -f k8s/frontend.yaml'
                      sleep 30
                      sh 'kubectl get pods'
                      sh 'kubectl get svc'
                    }
                }
            }
        }
    }
}
