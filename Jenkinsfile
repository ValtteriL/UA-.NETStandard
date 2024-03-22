pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Create Docker Network') {
            steps {
                sh 'docker network create opalopc-network'
            }
        }

        stage('Build & Start ReferenceServer') {
            steps {
                sh '''
                cd Applications/ConsoleReferenceServer/
                docker build -f Dockerfile -t consolerefserver ./../..
                docker run --rm -d --name refserver --network opalopc-network consolerefserver:latest
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
                    args '--network=opalopc-network'
                }
            }
            steps {
                sh '''
                export HOME=`pwd`
                opalopc -vv opc.tcp://refserver:62541 -o opalopc-report
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

            // Remove Docker Network if it exists
            sh 'docker network rm opalopc-network || true'
        }
    }
}