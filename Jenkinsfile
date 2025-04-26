#!groovy
pipeline {
    agent any
    stages {
        stage('sast-semgrep') {
            when {
                beforeAgent true
                anyOf {
                    expression { params.STAGE == 'all' }
                    expression { params.STAGE == 'sast-semgrep' }
                }
            }
            options {
                timeout(time: 1, unit: 'HOURS')  // Таймаут для stage
                retry(0)  // Отключаем повторные попытки
            }
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                    script {
                        sh '''
                            apk add --update python3 py3-pip py3-virtualenv
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
}
