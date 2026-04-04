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
            agent {
                docker {
                    image 'python:3.10-slim'
                }
            }
            steps {
                unstash 'compiled-results'

                // Install required system packages
                sh 'apt-get update && apt-get install -y binutils'

                // Install PyInstaller
                sh 'pip install --no-cache-dir pyinstaller'

                // Normalize Python files
                sh "sed -i 's/\\r\$//' sources/*.py"
                sh "sed -i '1s/^\\xEF\\xBB\\xBF//' sources/*.py"

                // Ensure executable
                sh "sed -i '1i #!/usr/bin/env python3' sources/prog.py"
                sh "chmod +x sources/prog.py"

                // Build binary
                sh 'pyinstaller -F sources/prog.py'

                // Archive result
                archiveArtifacts 'dist/prog'
            }
        }



    }
}