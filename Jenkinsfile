pipeline {
    agent any

    tools {
        nodejs 'NodeJs'
    }

    environment {
            SONAR_SCANNER_HOME = tool 'SonarQube-Scanner-600'
            DOCKERHUB_USER = "fahad813"
            BACKEND_IMAGE = "${DOCKERHUB_USER}/pipeline-sms-backend:${BUILD_NUMBER}"
            FRONTEND_IMAGE = "${DOCKERHUB_USER}/pipeline-sms-frontend:${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps{
                checkout scm
            }
        }

        stage('Install Dependency') {
            parallel {
                stage('Backend Dependency Install') {
                    steps {
                        dir('backend'){
                            bat 'npm install'
                        }
                    }
                }
                stage('Frontend Dependency install'){
                    steps {
                        dir('frontend'){
                            bat 'npm install'
                        }
                    }
                    
                }
            }
        }

        stage('Run Test') {
            steps{
                dir('backend'){
                    echo 'Backend Tests'
                }

                dir('frontend'){
                    echo 'Frontend Tests'
                }
            }
        }

        stage('Security Audit') {
            steps {
                dir('backend'){
                    bat 'npm audit --audit-level=critical'
                }
                dir('frontend'){
                    bat 'npm audit --audit-level=critical'
                }
            }
        }

        stage('SAST') {
            steps {
                withSonarQubeEnv('MySonarQubeServer'){
                    bat """
                    ${SONAR_SCANNER_HOME}/bin/sonar-scanner ^
                      -Dsonar.projectKey=sms-pipeline ^
                      -Dsonar.sources=. ^
                      -Dsonar.host.url=%SONAR_HOST_URL%
                    """
                }
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Backend Image') {
                    steps{
                        script {
                            bat 'docker build -t %BACKEND_IMAGE% ./backend'
                        }
                      
                    }
                }

                stage('Frontend Image') {
                    steps {
                      
                        bat 'docker build -t %FRONTEND_IMAGE% ./frontend'
                    }
                }

            }
        }

        stage("Trivy") {
            parallel{
                stage("scan backend image") {
                    steps {
                        bat "trivy.exe image --severity CRITICAL --exit-code 1 --format json -o backend-trivy-report.json --timeout 30m %BACKEND_IMAGE%"
                    }
                }
                stage("scan frontend image") {
                    steps {
                        bat """ 
                        trivy.exe image ^
                        --severity CRITICAL ^
                        --exit-code 1 ^
                        --format json ^
                        -o frontend-trivy-report.json ^
                        --timeout 30m ^
                        %FRONTEND_IMAGE%
                        """
                    }
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerHub_creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
                }
            }
        }

        stage('Push Images to DockerHub') {
            parallel {
                stage('Push Backend') {
                    steps {
                        bat 'docker push %BACKEND_IMAGE%'
                    }
                }
                stage('Push Frontend') {
                    steps {
                        bat 'docker push %FRONTEND_IMAGE%'
                    }
                }
            }
        }

        stage('Docker compose') {
            steps {
                bat 'docker compose up -d'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '*.json', allowEmptyArchive: true
        }
        success {
            echo 'pipeline suceess.'
        }
        failure {
            echo 'pipeline failed.'
        }
    }
}
