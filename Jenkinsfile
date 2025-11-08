pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo '📦 Checking out source code...'
                checkout scm
            }
        }

        stage('Build Application') {
            steps {
                echo '🏗️ Building the Java application...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SCA - Dependency Check') {
            steps {
                echo '🔍 Running OWASP Dependency Check...'
                sh '''
                docker run --rm \
                  -v "$PWD:/src" \
                  owasp/dependency-check:latest \
                  --scan /src \
                  --format "HTML" \
                  --project "demo" \
                  --out /src/reports
                '''
            }
        }

        stage('SAST - ShiftLeft Scan') {
            steps {
                echo '🧠 Running Static Application Security Testing (SAST)...'
                sh '''
                docker run --rm \
                  -v "$PWD:/app" \
                  shiftleft/sast-scan:v2.1.2 \
                  scan --type java --src /app --out_dir /app/reports
                '''
            }
        }

        stage('Archive Reports') {
            steps {
                echo '📄 Archiving security scan reports...'
                archiveArtifacts artifacts: 'reports/*.html', fingerprint: true
            }
        }
    }

    post {
        always {
            echo '✅ Pipeline completed (with or without vulnerabilities).'
        }
    }
}
