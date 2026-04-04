pipeline {
    agent none
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:3.8-alpine3.16'
                }
            }
            steps {
                sh 'python3.8 -m py_compile sources/prog.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        
        stage('Test') {
            agent {
                docker {
                    image 'grihabor/pytest'
                }
            }
            steps {
                sh 'pytest  -v --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit "test-reports/results.xml"
                }
            }
        }

        stage('Deliver') {
            agent any
            steps {
                unstash 'compiled-results'

                script {
                    docker.image('cdrx/pyinstaller-linux').inside("-v ${env.WORKSPACE}/sources:/src") {

                        sh "sed -i 's/\\r\$//' /src/*.py"
                        sh "sed -i '1s/^\\xEF\\xBB\\xBF//' /src/*.py"

                        sh "sed -i '1i #!/usr/bin/env python3' /src/prog.py"
                        sh "chmod +x /src/prog.py"

                        sh 'pyinstaller -F /src/prog.py'
                    }
                }
            }
            post {
                success {
                    archiveArtifacts 'sources/dist/prog'
                    sh 'rm -rf sources/build sources/dist || true'
                }
            }
        }

    }
}