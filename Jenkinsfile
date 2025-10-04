pipeline {
    agent any
    stages {
        stage('Checkout code') {
            steps {
                echo 'Checking out code'
                git branch: 'main', url: 'https://github.com/BenNiuX/aws-elastic-beanstalk-express-js-sample.git'
                stash name: 'sourcecode', includes: '**'
            }
        }
        stage('Snyk Scan') {
            steps {
                snykSecurity(
                    snykInstallation: 'Snyk Scanner',
                    snykTokenId: 'snyk_api_token',
                    failOnIssues: true,
                    severity: 'high',
                    monitorProjectOnBuild: false
                )
            }
        }
        stage('Build, Run and Test') {
            agent {
                docker {
                    image 'node:16'
                    args '-p 8080:8080'
                }
            }
            steps {
                echo 'Building, running and testing the app'
                unstash 'sourcecode'
                sh 'npm install --save'
                sh 'npm start &'
                sh 'sleep 5'
                script {
                    // if contains Hello World, then the app is running
                    if (sh(returnStdout: true, script: 'curl http://localhost:8080').contains('Hello World!')) {
                        echo 'App is running'
                        // finish the parallel stage when the app is running
                        currentBuild.result = 'SUCCESS'
                    } else {
                        error 'App is not running'
                        // finish the parallel stage when the app is not running
                        currentBuild.result = 'FAILURE'
                    }
                }
            }
        }
        stage('Build & Push Docker Image') {
            when {
                expression {
                    currentBuild.result == 'SUCCESS'
                }
            }
            environment {
                DOCKER_IMAGE = 'niubenxy/secdevops'
                DOCKER_TAG = 'assignment2'
            }
            steps {
                echo 'Building and pushing the docker image'
                // Create a Dockerfile as below
                // Create Dockerfile using writeFile (cleanest approach)
                writeFile file: 'Dockerfile', text: '''FROM node:16
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 8080
CMD ["npm", "start"]'''
                script {
                    docker.withRegistry('', 'credential_docker') {
                        def dockerImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}", ".")
                        dockerImage.push()
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs(
                deleteDirs: true, // Delete directories as well
                notFailBuild: true, // Do not fail the build if cleanup fails
                patterns: [
                    [pattern: '.git/**', type: 'INCLUDE'], // Include .git directory
                ]
            )
            echo 'Workspace cleaned.'
        }
    }
}
