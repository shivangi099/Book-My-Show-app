pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node23'  
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
                git branch: 'feature/BMS-jenkins', url: 'https://github.com/shivangi099/Book-My-Show-app.git'
                sh 'ls -la'  
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' 
                    $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BMS \
                    -Dsonar.projectKey=BMS 
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    try {
                        sh '''
                        cd bookmyshow-app
                        if [ -f package.json ]; then
                            [ -d node_modules ] && rm -rf node_modules
                            npm install --legacy-peer-deps
                        else
                            echo "Error: package.json not found!"
                            exit 1
                        fi
                        '''
                    } catch (err) {
                        echo "npm install failed: ${err}"
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh '''
                            docker build -t bms:latest bookmyshow-app
                            docker tag bms:latest 81392756/bms:latest
                            echo "Pushing Docker image to registry..."
                            docker push 81392756/bms:latest
                        '''
                    }
                }
            }
        }
        stage('Deploy to Container') {
            steps {
                sh '''
                    docker rm -f bms || true
                    docker run -d --name bms -p 3000:3000 81392756/bms:latest
                '''
            }
        }
    }
    post {
        always {
            emailext attachLog: true,
                subject: "'${currentBuild.result}'",
                body: "Project: ${env.JOB_NAME}<br/>" +
                      "Build Number: ${env.BUILD_NUMBER}<br/>" +
                      "URL: ${env.BUILD_URL}<br/>",
                to: 'shivangigarg678@gmail.com'
        }
    }
}
