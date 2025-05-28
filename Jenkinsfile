pipeline {
    agent any

    stages {
        stage('Declarative: Checkout SCM') {
            steps {
                checkout scm
            }
        }
        
        stage('Checkout') {
            steps {
                // Additional git checkout if needed (from your logs)
                bat 'git fetch --tags --force --progress'
                bat 'git checkout main'
            }
        }

        stage('Format') {
            steps {
                // Auto-fix code formatting violations
                bat 'mvn spring-javaformat:apply'
            }
        }

        stage('Build') {
            steps {
                bat 'mvn clean package'
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def status = bat(returnStatus: true, script: '''
                        @echo off
                        setlocal enabledelayedexpansion
                        REM Kill any process listening on port 8082, ignore if none found
                        set KILLED=0
                        for /F "tokens=5" %%a in ('netstat -ano ^| find ":8082" ^| find "LISTENING"') do (
                            taskkill /PID %%a /F
                            set KILLED=1
                        )
                        if !KILLED! == 0 (
                            echo No process listening on port 8082 found.
                        )
                        REM Start the Spring Boot application (adjust path if needed)
                        echo Starting Spring Boot app...
                        start "" java -jar target/spring-petclinic-3.4.0-SNAPSHOT.jar --server.port=8082
                        REM Wait 10 seconds to allow app startup
                        timeout /t 10 /nobreak > nul

                        REM Basic health check using PowerShell Invoke-WebRequest
                        powershell -Command "try { Invoke-WebRequest -Uri http://localhost:8082/ -UseBasicParsing -TimeoutSec 5; exit 0 } catch { exit 1 }"
                        exit /b %ERRORLEVEL%
                    ''')

                    if (status != 0) {
                        echo "Warning: Health check failed or deploy script exited with status ${status}, but continuing pipeline."
                    } else {
                        echo "Deploy stage completed successfully."
                    }
                }
            }
        }
    }
}
