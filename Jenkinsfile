pipeline {
    agent any

    environment {
        JAR_NAME = 'spring-petclinic-2.7.0-SNAPSHOT.jar'
        APP_PORT = '8082'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                git url: 'https://github.com/Yemmmyc/spring-petclinic-docker.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                echo 'Building Maven project...'
                bat 'mvn clean package'
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Deploy') {
            steps {
                echo "Stopping any previous instance running on port ${APP_PORT}..."
                bat """
                    @echo off
                    for /f "tokens=5" %%a in ('netstat -ano ^| find ":${APP_PORT}" ^| find "LISTENING"') do (
                        echo Killing PID %%a on port ${APP_PORT}
                        taskkill /PID %%a /F
                    )
                """

                echo "Starting application on port ${APP_PORT}..."
                bat """
                    cd target
                    start java -jar %JAR_NAME% --server.port=${APP_PORT}
                """
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful!'
        }
        failure {
            echo '❌ Build or deployment failed.'
        }
    }
}
