pipeline {
    // agent {
    //     label 'docker-agent' // Используйте выделенный агент, не master
    // }
    stages {
        // 1. Сборка приложения
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        // 2. SAST (Static Application Security Testing)
        stage('SAST with Semgrep') {
            steps {
                sh 'semgrep --config=auto .'
                // Сохранение отчета в формате SARIF
                sh 'semgrep --config=auto --sarif . > semgrep-results.sarif'
            }
        }

        // 3. SCA (Software Composition Analysis)
        stage('SCA with Dependency-Track') {
            steps {
                // Сканирование зависимостей и отправка в Dependency-Track
                sh 'dependency-track-ctl.sh upload-bom --project-name "Node-Media-Server" --project-version "1.0" --bom bom.xml'
            }
        }

        // 4. Сборка Docker-образа
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t node-media-server:latest .'
            }
        }

        // 5. Контейнерная безопасность (Trivy + Dependency-Track)
        stage('Container Security with Trivy') {
            steps {
                // Сканирование образа Trivy и экспорт в Dependency-Track
                sh 'trivy image --format cyclonedx --output trivy-results.json node-media-server:latest'
                sh 'dependency-track-ctl.sh upload-scan --project-name "Node-Media-Server" --scan trivy-results.json'
            }
        }

        // 6. DAST (Dynamic Application Security Testing)
        stage('DAST with OWASP ZAP') {
            steps {
                // Запуск ZAP и сканирование приложения
                sh 'zap-baseline.py -t http://localhost:8000 -r zap-report.html'
            }
        }
    }
    post {
        always {
            // Публикация отчетов
            publishHTML(target: [reportDir: '.', reportFiles: 'zap-report.html', reportName: 'ZAP Report'])
            dependencyTrackPublisher() // Интеграция с Dependency-Track
        }
    }
}
