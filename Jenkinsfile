pipeline {
    agent any
   
    stages {

        stage('Run Selenium Tests with pytest') {
            steps {
                echo "Running Selenium Tests using pytest"

                // Install Python dependencies
                bat 'pip install -r requirements.txt'

                // Start Flask app in background and log output
                bat 'start /MIN cmd /C "python app.py > flask_log.txt 2>&1"'

                // Wait until Flask server responds
                bat '''
                @echo off
                setlocal enabledelayedexpansion
                echo Waiting for Flask server to start...
                set server_up=0
                for /L %%i in (1,1,30) do (
                    powershell -Command "try { $r=(Invoke-WebRequest -Uri http://127.0.0.1:5000 -UseBasicParsing).StatusCode; exit 0 } catch { exit 1 }"
                    if !errorlevel! == 0 (
                        echo Flask server is up!
                        set server_up=1
                        goto :breakLoop
                    )
                    timeout /t 2 >nul
                )
                :breakLoop
                if !server_up! == 0 (
                    echo Flask server failed to start after 60 seconds
                    exit /b 1
                )
                endlocal
                '''

                // Run pytest Selenium tests
                bat 'pytest -v --disable-warnings --maxfail=1'

                // Kill the Flask app only
                bat 'taskkill /IM python.exe /F /T'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Build Docker Image"
                bat "docker build -t seleniumdemoapp:v1 ."
            }
        }

        stage('Docker Login') {
            steps {
                bat 'docker login -u varshap25 -p Varsha@2503'
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                echo "Push Docker Image to Docker Hub"
                bat "docker tag seleniumdemoapp:v1 varshap25/sample:seleniumtestimage"               
                bat "docker push varshap25/sample:seleniumtestimage"
            }
        }

        stage('Deploy to Kubernetes') { 
            steps { 
                bat 'kubectl apply -f deployment.yaml --validate=false' 
                bat 'kubectl apply -f service.yaml' 
            } 
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
