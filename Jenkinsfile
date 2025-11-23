pipeline {
    agent {
        kubernetes {
            yamlFile 'build-agent.yaml'
        }
    }

    options {
        skipDefaultCheckout()
    }

    stages {

        stage('Checkout') {
            steps {
                container('maven') {
                    checkout scm
                }
            }
        }

        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn -DskipTests clean package'
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage('Static Analysis') {
            steps {
                container('maven') {
                    sh '''
                        echo "Static Analysis placeholder - Dependency-Check skipped in this lab environment"
                    '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: '**/dependency-check-report.*', allowEmptyArchive: true
                }
            }
        }

        stage('SAST') {
            steps {
                container('slscan') {
                    sh 'scan --type java,depscan --build'
                }
            }
            post {
                always {
                    archiveArtifacts allowEmptyArchive: true, artifacts: 'reports/**', fingerprint: true
                }
            }
        }

        stage('Dependency-Track Upload') {
            steps {
                container('kubectl') {
                    sh '''
                        echo "Dependency Track stage (optional)"
                    '''
                }
            }
        }

    }

    post {
        always {
            echo "Pipeline completed."
        }
    }
}
