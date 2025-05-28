pipeline {
    agent any

    environment {
        JAR_NAME = 'spring-petclinic-2.7.0-SNAPSHOT.jar'
        APP_PORT = '8082'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Yemmmyc/my-spring-petclinic.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
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
                bat """
                    for /f "tokens=5" %%a in ('netstat -ano ^| find ":${APP_PORT}" ^| find "LISTENING"') do taskkill /PID %%a /F
                """
                bat """
                    cd target
                    start java -jar %JAR_NAME% --server.port=${APP_PORT}
                """
            }
        }
    }
}
