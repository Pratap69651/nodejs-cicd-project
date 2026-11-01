
pipeline {
    agent any

    tools {
        nodejs 'NodeJS-16'
    }

    options {
        timestamps()
        ansiColor('xterm')
        disableConcurrentBuilds()
        buildDiscarder(logRotator(
            numToKeepStr: '10'
        ))
    }

    stages {

        stage('01 - Checkout') {
            steps {
                echo '\033[34mChecking out source code...\033[0m'
                checkout scm
            }
        }

        stage('02 - Environment') {
            steps {
                echo 'Checking environment'
                sh 'node --version'
                sh 'npm --version'
                sh 'git --version'
            }
        }

        stage('03 - Install Dependencies') {
            steps {
                echo 'Installing Node.js dependencies'
                sh 'npm install'
            }
        }

        stage('04 - Unit Testing') {
            steps {
                echo 'Running unit tests'
                sh 'npm test'
            }
        }

        stage('05 - Build Application') {
            steps {
                echo 'Building application'
                sh 'npm run build'
            }
        }

        stage('06 - SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'

                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner
                        """
                    }
                }
            }
        }
        stage('07 - Quality Gate') {
    steps {
        timeout(time: 5, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }
}
        stage('08 - Archive Artifacts') {
            steps {
                echo 'Archiving application files'
                archiveArtifacts artifacts: 'app.js, package.json',
                                 fingerprint: true
            }
        }
    }

    post {
        success {
            echo '\033[32mCI PIPELINE SUCCESSFUL!\033[0m'
        }

        failure {
            echo '\033[31mCI PIPELINE FAILED!\033[0m'
        }

        always {
            echo 'Pipeline execution completed'
        }
    }
}
