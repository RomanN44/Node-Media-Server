pipeline {
    agent {
        label 'dind'
    }
    environment {
        DOJO_API = "https://s410-exam.cyber-ed.space:8083/api/v2"
        API_TOKEN = '5c45847565eea7c9c5551f49ad8d72c64a72fa36'
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
                    sh """
                        apk add --no-cache openjdk11-jre-headless python3 py3-pip curl git
                        pip3 install python-owasp-zap-v2.4 --break-system-packages
                        ZAP_VERSION=\$(curl -s https://api.github.com/repos/zaproxy/zaproxy/releases/latest | sed -n 's/.*"tag_name": "\([^"]*\)".*/\1/p')
                        curl -sL "https://github.com/zaproxy/zaproxy/releases/download/\${ZAP_VERSION}/ZAP_\${ZAP_VERSION#v}_Linux.tar.gz" | tar -xz -C /opt
                        ln -s /opt/ZAP_*/zap.sh /usr/local/bin/zap
                        mkdir -p "./zap-reports/"
                        python3 /opt/ZAP_*/zap-full-scan.py \
                            -I -j -m 10 -T 60 \
                            -t "https://s410-exam.cyber-ed.space:8084" \
                            -x ./zap-reports/report_site.xml
                        cp ./zap-reports/report_site.xml .
                    """
                    archiveArtifacts artifacts: "report_site.xml", allowEmptyArchive: true
                    stash(name: 'zap-report', includes: "report_site.xml")
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
