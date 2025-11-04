pipeline {
    agent any

    environment {
        MAVEN_CONTAINER = 'maven'
        LICENSE_CONTAINER = 'licensefinder'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                container("${MAVEN_CONTAINER}") {
                    sh 'mvn clean install -DskipTests'
                }
            }
        }

        stage('StaticAnalysis') {
            parallel {

                stage('SCA') {
                    steps {
                        container("${MAVEN_CONTAINER}") {
                            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                                sh 'mvn org.owasp:dependency-check-maven:check'
                            }
                        }
                    }
                    post {
                        always {
                            archiveArtifacts allowEmptyArchive: true,
                                             artifacts: 'target/dependency-check-report.html',
                                             fingerprint: true,
                                             onlyIfSuccessful: true
                        }
                    }
                }

                stage('OSSLicenseChecker') {
                    steps {
                        container("${LICENSE_CONTAINER}") {
                            // Optional: List directory to verify
                            sh 'ls -al'
                            // Install license_finder and run
                            sh '''
                            #!/bin/bash --login
                            gem install license_finder
                            license_finder
                            '''
                        }
                    }
                }

            } // end parallel
        }

        stage('UnitTests') {
            steps {
                container("${MAVEN_CONTAINER}") {
                    sh 'mvn test'
                }
            }
        }

        stage('Package') {
            steps {
                container("${MAVEN_CONTAINER}") {
                    sh 'mvn package'
                }
            }
        }

    } // end stages

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }

}
