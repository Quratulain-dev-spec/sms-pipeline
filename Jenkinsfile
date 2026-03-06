pipeline {
    agent any

     tools {
        nodejs 'NodeJs'
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

        

        stage('Build Docker Images') {
            parallel {
                stage('Backend Image') {
                    steps{
                        script {
                            bat 'docker build -t anniesaeed/my-nodejs-app ./backend'
                        }
                      
                    }
                }

                stage('Frontend Image') {
                    steps {
                      
                        bat 'docker build -t anniesaeed/my-nodejs-app ./frontend'
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
