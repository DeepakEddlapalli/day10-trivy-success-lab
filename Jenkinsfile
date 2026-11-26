pipeline {
    agent any

    environment {
        IMAGE_NAME = "day10-trivy-success"
        TRIVY = "/usr/local/bin/trivy"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Generate Image Report') {
            steps {
                sh '''
                    mkdir -p reports

                    ${TRIVY} image \
                    --format table \
                    --output reports/image-report.txt \
                    ${IMAGE_NAME}:${BUILD_NUMBER}

                    ${TRIVY} image \
                    --format json \
                    --output reports/image-report.json \
                    ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Generate Config Report') {
            steps {
                sh '''
                    ${TRIVY} config \
                    --format table \
                    --output reports/config-report.txt \
                    .

                    ${TRIVY} config \
                    --format json \
                    --output reports/config-report.json \
                    .
                '''
            }
        }

        stage('Trivy Image Security Gate') {
            steps {
                sh '''
                    ${TRIVY} image \
                    --severity HIGH,CRITICAL \
                    --exit-code 1 \
                    ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Trivy Config Security Gate') {
            steps {
                sh '''
                    ${TRIVY} config \
                    --severity HIGH,CRITICAL \
                    --exit-code 1 \
                    .
                '''
            }
        }

        stage('Success - Security Validation') {
            steps {
                echo 'Security gates passed!'
                echo 'No HIGH or CRITICAL findings detected by the configured gates.'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'reports/*',
                           allowEmptyArchive: true
        }

        success {
            echo 'Day 10 Success Lab completed successfully!'
        }

        failure {
            echo 'Security gate failed or another pipeline stage failed.'
        }
    }
}
