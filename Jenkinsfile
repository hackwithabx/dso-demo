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
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, allowEmptyArchive: true
                }
            }
        }

        stage('Static Analysis (SCA)') {
            steps {
                container('maven') {
                    echo "Static Analysis (Dependency-Check) is configured in pom.xml"
                    echo "But skipped in Jenkins due to NVD API + Java Heap limitations in this lab."
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
                    // Just print how SAST is run, no tricky quotes
                    sh '''
                        echo SAST with SCAN slscan is configured in this stage.
                        echo In this lab environment, the actual scan was executed manually via Docker:
                        echo docker run --rm -e WORKSPACE=${PWD} -v $PWD:/app shiftleft/sast-scan:v2.1.2 scan --type java,depscan --build
                    '''
                }
            }
            post {
                always {
                    // OK if reports/ does not exist
                    archiveArtifacts allowEmptyArchive: true, artifacts: 'reports/**', fingerprint: true
                }
            }
        }

        stage('Dependency-Track Upload') {
            steps {
                container('kubectl') {
                    echo "Dependency-Track stage placeholder (optional)"
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
