pipeline {
    agent any
    stages {
        // stage('Build') {
        //     steps {
        //         nodejs(nodeJSInstallationName: 'Node 6.x', configId: '<config-file-provider-id>') {
        //             sh 'npm config ls'
        //         }
        //     }
        // }
        // 2. SAST
        stage('SAST with Semgrep') {
            steps {
                sh 'semgrep --config=auto .'
                sh 'semgrep --config=auto --sarif . > semgrep-results.sarif'
            }
        }
        // 3. SCA
        stage('SCA with Dependency-Track') {
            steps {
                sh 'dependency-track-ctl.sh upload-bom --project-name "Node-Media-Server" --project-version "1.0" --bom bom.xml'
            }
        }

        // 4. Сборка Docker-образа
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t node-media-server:latest .'
            }
        }

        // 5.(Trivy + Dependency-Track)
        stage('Container Security with Trivy') {
            steps {
                sh 'trivy image --format cyclonedx --output trivy-results.json node-media-server:latest'
                sh 'dependency-track-ctl.sh upload-scan --project-name "Node-Media-Server" --scan trivy-results.json'
            }
        }

        // 6. DAST
        stage('DAST with OWASP ZAP') {
            steps {
                sh 'zap-baseline.py -t http://localhost:8000 -r zap-report.html'
            }
        }
    }
    // post {
    //     always {
    //         // Публикация отчетов
    //         publishHTML(target: [reportDir: '.', reportFiles: 'zap-report.html', reportName: 'ZAP Report'])
    //         dependencyTrackPublisher()
    //     }
    // }
}
