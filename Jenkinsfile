pipeline {
    agent any
    tools {
        jdk 'JAVA_HOME'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Dummyboy99/Capstone_Project.git'
                sh '''ls -la'''
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=Rohank-bms \
                    -Dsonar.sources=. \
                    -Dsonar.java.binaries=target/classes'''

                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh '''
                cd bookmyshow-app
                ls -la  # Verify package.json exists
                if [ -f package.json ]; then
                    rm -rf node_modules package-lock.json
                    npm i chokidar
                    npm install  # Install fresh dependencies
                else
                    echo "Error: package.json not found in bookmyshow-app!"
                    exit 1
                fi
                '''
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh ''' 
                        echo "Building Docker image..."
                        docker build --no-cache -t rohankanhai51/bms:latest -f bookmyshow-app/Dockerfile bookmyshow-app

                        echo "Pushing Docker image to registry..."
                        docker push rohankanhai51/bms:latest
                        '''
                    }
                }
            }
        }
        stage('Deploy to Container') {
            steps {
                sh ''' 
                echo "Stopping and removing old container..."
                docker stop bms || true
                docker rm bms || true

                echo "Running new container on port 3000..."
                docker run -d --restart=always --name bms -p 3069:3000 rohankanhai51/bms:latest

                echo "Checking running containers..."
                docker ps -a

                echo "Fetching logs..."
                sleep 5
                docker logs bms
                '''
            }
        }
    }
     post {
        success {
            emailext (
                subject: "SUCCESS: SonarQube Quality Gate Passed",
                body: """
                Hi Team,
                
                The SonarQube quality gate passed successfully.

                You can view the report here:
                http://3.138.102.215/:9000/dashboard?id=Rohank-bms
                username: admin, passwd: Rohan@200269

                If you want to download a PDF report, please use the SonarQube interface or a PDF plugin.

                Regards,
                Jenkins
                """,
                to: 'rohankanhai51@gmail.com'
            )
        }
        failure {
            emailext (
                subject: "FAILED: SonarQube Quality Gate",
                body: """
                Hi Team,
                
                The SonarQube quality gate has failed.
                
                Please check the details:
                http://3.138.102.215/:9000/dashboard?id=Rohank-bms
                username: admin, passwd: Rohan@200269
                
                Regards,
                Jenkins
                """,
                to: 'rohankanhai51@gmail.com'
            )
            
        }
    }
}