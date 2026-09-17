pipeline {
    agent any

    stages {

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    ./venv/bin/pip install --upgrade pip
                    ./venv/bin/pip install pytest pyinstaller
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    ./venv/bin/python -m py_compile \
                    sources/add2vals.py \
                    sources/calc.py
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    mkdir -p test-reports

                    cd sources

                    ../venv/bin/python -m pytest \
                    test_calc.py \
                    --verbose \
                    --junit-xml=../test-reports/results.xml
                '''
            }

            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }

        stage('Run Application') {
            steps {
                sh '''
                    cd sources
                    ../venv/bin/python add2vals.py
                '''
            }
        }

        stage('Deliver') {
            steps {
                sh '''
                    ./venv/bin/python -m PyInstaller \
                    --onefile \
                    sources/add2vals.py
                '''
            }

            post {
                success {
                    archiveArtifacts artifacts: 'dist/add2vals',
                                     fingerprint: true
                }
            }
        }

        stage('Deploy to AWS') {
            steps {
                sshagent(credentials: ['r1']) {
                    sh '''
                        echo "Copying application to AWS EC2..."

                        scp -o StrictHostKeyChecking=no \
                        dist/add2vals \
                        ubuntu@54.253.129.59:/tmp/add2vals

                        ssh -o StrictHostKeyChecking=no \
                        ubuntu@54.253.129.59 \
                        'sudo mv /tmp/add2vals /usr/local/bin/add2vals && \
                         sudo chmod +x /usr/local/bin/add2vals'

                        echo "Deployment completed successfully!"
                    '''
                }
            }
        }
    }
}
