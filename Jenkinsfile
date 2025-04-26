pipeline {
    agent {
        label 'dind'
    }
    stages {
        // stage('Build') {
        //     steps {
        //         nodejs(nodeJSInstallationName: 'Node 6.x', configId: '<config-file-provider-id>') {
        //             sh 'npm config ls'
        //         }
        //     }
        // }
        stage('Checkout') {
            steps {
                checkout scm 
            }
        }
        stage('Semgrep') {
            steps {
                script {
                    docker.image('returntocorp/semgrep').inside {
                    sh 'semgrep --config=auto .'
                }
            }
        }
        // stage('SCA with Dependency-Track') {
        //     steps {
        //         // Генерация BOM файла
        //         sh 'npm install -g @cyclonedx/cyclonedx-npm'
        //         sh 'cyclonedx-npm --output bom.xml'
                
        //         // Загрузка BOM через API
        //         sh """
        //         curl -X POST -H "X-API-Key: odt_SfCq7Csub3peq7Y6lSlQy5Ngp9sSYpJl" \
        //         -H "Content-Type: application/xml" \
        //         --data-binary @bom.xml \
        //         "https://s410-exam.cyber-ed.space:8081/api/v1/bom"
        //         """
        //     }
        // }

        // // 4. Сборка Docker-образа
        // stage('Build Docker Image') {
        //     steps {
        //         sh 'docker build -t node-media-server:latest .'
        //     }
        // }

        // // 5.(Trivy + Dependency-Track)
        // stage('Container Security with Trivy') {
        //     steps {
        //         sh 'trivy image --format cyclonedx --output trivy-results.json node-media-server:latest'
        //         sh 'dependency-track-ctl.sh upload-scan --project-name "Node-Media-Server" --scan trivy-results.json'
        //     }
        // }

        // // 6. DAST
        // stage('DAST with OWASP ZAP') {
        //     steps {
        //         sh 'zap-baseline.py -t http://localhost:8000 -r zap-report.html'
        //     }
        // }
    }
    // post {
    //     always {
    //         // Публикация отчетов
    //         publishHTML(target: [reportDir: '.', reportFiles: 'zap-report.html', reportName: 'ZAP Report'])
    //         dependencyTrackPublisher()
    //     }
    // }
}
