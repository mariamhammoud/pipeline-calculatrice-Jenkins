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
            environment {
                IMAGE = 'cdrx/pyinstaller-linux'
            }
            steps {
                dir(env.BUILD_ID) {

                    sh "ls -R ."

                    unstash 'compiled-results'

                    sh '''
                        VOLUME="$(pwd)/sources:/src"
                        echo "Using VOLUME=$VOLUME"

                        docker run --rm \
                            -v "$VOLUME" \
                            -e PYINSTALLER_OPTS="-F" \
                            cdrx/pyinstaller-linux "prog.py"
                    '''

                    sh "chown -R \$(id -u):\$(id -g) sources"
                }
            }
            post {
                success {
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/prog"
                    sh "rm -rf ${env.BUILD_ID}/sources/build ${env.BUILD_ID}/sources/dist"
                }
            }
        }



    }
}