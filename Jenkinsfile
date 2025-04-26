pipeline {
    agent {
        label 'dind'
    }
    environment {
        ZAP_TARGET = 'https://s410-exam.cyber-ed.space:8084'
        DEP_TRACK_API = 'https://s410-exam.cyber-ed.space:8080'
        DEP_TRACK_KEY = 'odt_SfCq7Csub3peq7Y6lSlQy5Ngp9sSYpJl'
    }
    stages {
        stage('SAST') {
            steps {
                script {
                    sh '''
                        apt-get update && apt-get install -y python3 python3-pip python3-venv
                        python3 -m venv venv
                        . venv/bin/activate
                        pip install semgrep
                        venv/bin/semgrep --config=auto --verbose --json > report_semgrep.json
                        '''
                    archiveArtifacts artifacts: 'report_semgrep.json', allowEmptyArchive: true
                }
            }
        }
        stage('DAST') {
            steps {
                script {
                    sh '''
                        mkdir /tmp/reports
                        wget https://github.com/zaproxy/zaproxy/releases/download/v2.16.1/ZAP_2.16.1_Linux.tar.gz
                        tar -xvf ZAP_2.16.1_Linux.tar.gz
                    '''
                    sh '''
                        chmod 777 ZAP_2.16.1
                        cd ZAP_2.16.1/
                        ./zap.sh -cmd -quickurl ${ZAP_TARGET} -quickout zap-report.html
                    '''
                    archiveArtifacts artifacts: 'zap-report.html', allowEmptyArchive: true
                }
            }
        }
        stage('SCA') {
            steps {
                sh '''
                    npm install -g @cyclonedx/cdxgen
                    cdxgen -o bom.xml
                    curl -X POST -H "Content-Type: application/xml" -H "X-API-Key: ${DEP_TRACK_KEY}" --data-binary @bom.xml ${DEP_TRACK_API}/api/v1/bom
                '''
                script {
                    def vulnerabilities = sh(script: 'curl -H "X-API-Key: ${DEP_TRACK_KEY}" ${DEP_TRACK_API}/api/v1/project/{uuid}/vulnerabilities', returnStdout: true)
                    if (vulnerabilities.contains('"critical":')) {
                        error "FIND CRITICAL VULH!"
                    }
                }
            }
        }
        stage('Container Security') {
            steps {
                script {
                sh '''
                    curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin
                    docker build -t node-media-server:${BUILD_ID} .
                    trivy image --format cyclonedx --output sbom.json your-nodejs-app:${BUILD_ID}
                     curl -X POST -H "X-API-Key: ${API_KEY}" -H "Content-Type: application/json" --data-binary @sbom.json ${DEP_TRACK_API}/api/v1/bom
                '''
            }
        }
    }
}
