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
                DEFECTDOJO_API_KEY = credentials('defectdojo-api-key')
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

                // Import results to DefectDojo
                sh '''
                curl -X 'POST' \
                    'http://172.16.1.15:8080/api/v2/import-scan/' \
                    -H 'accept: application/json' \
                    -H "Authorization: Token $DEFECTDOJO_API_KEY" \
                    -H 'Content-Type: multipart/form-data' \
                    -F 'active=false' \
                    -F 'verified=true' \
                    -F 'close_old_findings=true' \
                    -F 'engagement_name=DAST pipeline' \
                    -F "build_id=$BUILD_ID" \
                    -F 'deduplication_on_engagement=true' \
                    -F 'minimum_severity=Info' \
                    -F 'create_finding_groups_for_all_findings=true' \
                    -F "commit_hash=$GIT_COMMIT" \
                    -F 'product_name=ConsoleReferenceServer' \
                    -F 'file=@opalopc-report.sarif' \
                    -F 'auto_create_context=true' \
                    -F 'scan_type=SARIF' \
                    -F "branch_tag=$GIT_BRANCH"
                '''
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