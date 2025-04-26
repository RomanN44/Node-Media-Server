pipeline {
    agent {
        label 'dind'
    }
    environment {
        DOJO_API = "https://s410-exam.cyber-ed.space:8083/api/v2"
        API_TOKEN = '5c45847565eea7c9c5551f49ad8d72c64a72fa36'
        ZAP_TARGET = 'https://s410-exam.cyber-ed.space:8084'
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
                    archiveArtifacts artifacts: 'zap-report.*', allowEmptyArchive: true
                }
            }
        }
        stage('DefectDojo') {
            steps {
                script {
                    sh '''
                    curl -X POST "{$URL_DOJO}/import-scan" \
                    -H "Authorization: Token ${API_TOKEN}" \
                    -H "Content-Type: multipart/form-data" \
                    -F "scan-type=Semgrep JSON Report" \
                    -F "file=report_semgrep.json" \
                    -F "engagement=1"
                    -F "verified=true"
                    '''
                }
            }
        }
    }
}
