pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Start ReferenceServer') {
            steps {
                sh '''
                cd Applications/ConsoleReferenceServer/
                docker build -f Dockerfile -t consolerefserver ./../..
                docker run --rm -d -p 62541:62541 --name refserver consolerefserver:latest
                '''
            }
        }

        stage('Run OpalOPC') {
            environment {
                OPALOPC_LICENSE_KEY = credentials('opalopc-license-key')
            }
            agent {
                dockerfile {
                    filename 'Dockerfile.opalopc'
                }
            }
            steps {
                sh '''
                export HOME=`pwd`
                opalopc -vv localhost:62541 -o opalopc-report
                '''

                // Archive results
                archiveArtifacts artifacts: 'opalopc-report.*'
            }
        }
    }
    post {
        always {
            // Kill ReferenceServer if its running
            sh 'docker kill refserver || true'
        }
    }
}