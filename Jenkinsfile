pipeline {
    agent {
        label 'dind'
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
    }
}
