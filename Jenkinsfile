pipeline{
    agent any
    tools{
        nodejs 'nodejs23'
    }
    environment{
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages{
        stage("Git Checkout"){
            steps{
              git branch: 'docker', credentialsId: 'git-cred', url: 'https://github.com/pguleria52-devops/DevOps-Project.git'
            }
        }
        stage("Frontend Compilation"){
            steps{
                dir('client') {
                    sh 'find . -name "*.js" -exec node --check {} +'
                }
            }
        }
        stage("Backend Compilation"){
            steps{
                dir('api'){
                    sh ' find . -name "*.js" -exec node --check {} +'
                }
            }
        }
        stage("Gitleaks check"){
            steps{
                sh 'gitleaks detect --source ./client --exit-code 1'
                sh 'gitleaks detect --source ./api --exit-code 1'
            }
        }
        stage("SonarQube Analysis"){
            steps{
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=NodeJS-Project \
                            -Dsonar.projectKey=NodeJS-Project '''
                }
            }
        }
        stage("Quality Gate Check"){
            steps{
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage("Trivy FS Scan"){
            steps{
                 sh 'trivy fs --format table -o fs-report.html .'
            }
        }
        stage("Build and Push Backend Docker Image"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred'){
                        dir('api'){
                            sh 'docker build -t pguleria/backend:latest .'
                            sh 'trivy image --format table -o backend-image-report.html pguleria/backend:latest '
                            sh 'docker push pguleria/backend:latest'
                        }
                    }
                }
            }
        }
        stage("Build and Push Frontend Docker Image"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred'){
                        dir('client'){
                            sh 'docker build -t pguleria/frontend:latest .'
                            sh 'trivy image --format table -o backend-image-report.html pguleria/frontend:latest '
                            sh 'docker push pguleria/frontend:latest'
                        }
                    }
                }
            }
        }
        stage("Docker Deploy via Compose"){
            steps{
                script{
                    sh 'docker-compose up -d'
                }
            }
        }
    }
}