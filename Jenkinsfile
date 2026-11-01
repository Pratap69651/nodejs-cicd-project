
pipeline {
    agent any

    tools {
        nodejs 'NodeJS-16'
    }

    options {
        timestamps()
        ansiColor('xterm')
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        DOCKER_IMAGE = 'nodejs-cicd-project'
        CONTAINER_NAME = 'nodejs-cicd-test'
    }

    stages {

        stage('01 - Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('02 - Environment') {
            steps {
                sh 'node --version'
                sh 'npm --version'
                sh 'git --version'
                sh 'docker --version'
            }
        }

        stage('03 - Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('04 - Unit Testing') {
            steps {
                sh 'npm test'
            }
        }

        stage('05 - Build Application') {
            steps {
                sh 'npm run build'
            }
        }

        stage('06 - SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'

                    withSonarQubeEnv('SonarQube') {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }

        stage('07 - Quality Gate') {
    steps {
        timeout(time: 5, unit: 'MINUTES') {
            script {
                def qg = waitForQualityGate abortPipeline: false
                echo "SonarQube Quality Gate Status: ${qg.status}"
            }
        }
    }
}

        stage('08 - Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                    docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('09 - Run Docker Container') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p 127.0.0.1:3001:3000 \
                        ${DOCKER_IMAGE}:${BUILD_NUMBER}
                '''
            }
        }

        stage('10 - Smoke Test') {
            steps {
                sh '''
                    sleep 5
                    curl --fail http://127.0.0.1:3001
                '''
            }
        }

        stage('11 - Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'app.js, package.json',
                                 fingerprint: true
            }
        }
    }

    post {
        always {
            sh 'docker rm -f ${CONTAINER_NAME} || true'
            echo 'Pipeline execution completed'
        }

        success {
            echo 'CI PIPELINE + DOCKER SUCCESSFUL!'
        }

        failure {
            echo 'CI PIPELINE FAILED!'
        }
    }
}
