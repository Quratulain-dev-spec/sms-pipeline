pipeline {
    agent any

     tools {
        nodejs 'NodeJs'
    }
    environment {
            SONAR_SCANNER_HOME = tool 'SonarQube-Scanner'
            DOCKERHUB_USER = "anniesaeed"
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
                withSonarQubeEnv('SonarQube'){
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
                            bat 'docker build -t anniesaeed/my-nodejs-app:latest ./backend'
                        }
                      
                    }
                }

                stage('Frontend Image') {
                    steps {
                      
                        bat 'docker build -t anniesaeed/my-nodejs-app:latest ./frontend'
                    }
                }

            }
        }

        stage("Trivy") {
            parallel{
                stage("scan backend image") {
                    steps {
                        bat """
                        trivy.exe image ^
                        --scanners vuln ^
                        --severity CRITICAL ^
                        --exit-code 1 ^
                        --format json ^
                        -o backend-trivy-report.json ^
                        --timeout 30m ^
                        anniesaeed/my-nodejs-app:latest
                        """
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
                        anniesaeed/my-nodejs-app:latest
                        """
                    }
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
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
                        bat 'docker push anniesaeed/my-nodejs-app:latest'
                    }
                }
                stage('Push Frontend') {
                    steps {
                        bat 'docker push anniesaeed/my-nodejs-app:latest'
                    }
                }
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
