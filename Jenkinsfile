pipeline {
    agent any

    tools {
        nodejs 'NodeJs'
    }

    environment {
            SONAR_SCANNER_HOME = tool 'SonarQube-Scanner-600'
            BACKEND_IMAGE = "pipeline-sms-backend"
            FRONTEND_IMAGE = "pipeline-sms-frontend"
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
                      -D"sonar.projectKey=sms-pipeline" ^
                      -Dsonar.sources="backend, frontend" ^
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
                    steps{
                        bat """
                        --severity CRITICAL ^ 
                        --exit-code 1 ^
                        --format json ^
                        -o backend-trivy-report.json ^
                        %BACKEND_IMAGE%
                        """
                    }
                }

                stage("scan frontend image") {
                    steps {
                        bat """ 
                        --severity CRITICAL ^
                        --exit-code 1 ^
                        --format json ^
                        -o frontend-trivy-report.json ^
                        %FRONTEND_IMAGE%
                        """
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
            echo 'pipeline finished.'
        }
        success {
            echo 'pipeline suceess.'
        }
        failure {
            echo 'pipeline failed.'
        }
    }
}