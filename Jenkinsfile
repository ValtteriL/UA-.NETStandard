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
                sh 'docker network create $BUILD_NUMBER-opalopc-network'
            }
        }

        stage('Build & Start ReferenceServer') {
            steps {
                sh '''
                cd Applications/ConsoleReferenceServer/
                docker build -f Dockerfile -t consolerefserver ./../..
                docker run --rm -d --name $BUILD_NUMBER-refserver --network $BUILD_NUMBER-opalopc-network consolerefserver:latest
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
                    args '--network=$BUILD_NUMBER-opalopc-network'
                }
            }
            steps {
                sh '''
                export HOME=`pwd`
                opalopc -vv opc.tcp://$BUILD_NUMBER-refserver:62541 -o opalopc-report
                '''

                // Archive results
                archiveArtifacts artifacts: 'opalopc-report.*'
            }
        }
    }
    post {
        always {
            // Kill ReferenceServer if its running
            sh 'docker kill $BUILD_NUMBER-refserver || true'

            // Remove Docker Network if it exists
            sh 'docker network rm $BUILD_NUMBER-opalopc-network || true'
        }
    }
}