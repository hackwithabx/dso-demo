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
                        echo "Static Analysis (Dependency-Check) is configured in pom.xml,"
                        echo "but skipped in Jenkins due to NVD API/heap limitations."
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
                    sh '''
                        echo "SAST with SCAN (slscan) is configured in this stage."
                        echo "In this lab environment, the full scan was executed manually via Docker:"
                        echo "  docker run --rm -e WORKSPACE=\\${PWD} -v \\$PWD:/app shiftleft/sast-scan:v2.1.2 scan --type java,depscan --build"
                    '''
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
                        echo "Dependency-Track upload stage (optional) – not used in this lab."
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
