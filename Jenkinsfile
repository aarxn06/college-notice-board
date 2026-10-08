
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
    }

    
    environment {
        PATH = "C:\\Users\\Aaron Patric Johnson\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin;${env.PATH}"
        IMAGE_NAME = 'aarxn06/college-notice-board'
        DEPLOYMENT_NAME = 'college-notice-board'
    }


    stages {
        stage('Clone Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
                    @echo off
                    docker build -t %IMAGE_NAME%:%BUILD_NUMBER% .
                '''
            }
        }

        stage('Test Application') {
            steps {
                bat '''
                    @echo off
                    docker run --rm --entrypoint sh %IMAGE_NAME%:%BUILD_NUMBER% -c "test -f /usr/share/nginx/html/index.html && grep -q 'College Notice Board' /usr/share/nginx/html/index.html"
                    if errorlevel 1 exit /b 1
                    echo Application content test passed.
                '''
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    bat '''
                        @echo off
                        powershell -NoProfile -Command "$env:DOCKER_PASSWORD | docker login -u $env:DOCKER_USER --password-stdin"
                        if errorlevel 1 exit /b 1

                        docker push %IMAGE_NAME%:%BUILD_NUMBER%
                        if errorlevel 1 exit /b 1

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig-creds',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    bat '''
                        @echo off
                        kubectl apply -f deployment.yaml
                        if errorlevel 1 exit /b 1

                        kubectl set image deployment/%DEPLOYMENT_NAME% notice-board=%IMAGE_NAME%:%BUILD_NUMBER%
                        if errorlevel 1 exit /b 1

                        kubectl rollout status deployment/%DEPLOYMENT_NAME% --timeout=180s
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig-creds',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    bat '''
                        @echo off
                        kubectl get deployments
                        if errorlevel 1 exit /b 1

                        kubectl get pods -l app=college-notice-board
                        if errorlevel 1 exit /b 1

                        kubectl wait --for=condition=Ready pod -l app=college-notice-board --timeout=120s
                        if errorlevel 1 exit /b 1

                        kubectl get services
                        if errorlevel 1 exit /b 1
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Q1 CI/CD Pipeline Completed Successfully!'
        }
        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
