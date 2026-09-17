pipeline {
    agent any

    stages {

        stage('Install Dependencies') {
            steps {
                sh '''
                    rm -rf venv
                    python3 -m venv venv
                    ./venv/bin/pip install --upgrade pip
                    ./venv/bin/pip install pytest pyinstaller
                '''
            }
        }

        stage('Check Files') {
            steps {
                sh '''
                    echo "Checking project files..."
                    ls -la
                    ls -la sources
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    ./venv/bin/python -m py_compile \
                    sources/add2vals.py \
                    sources/calc.py \
                    sources/test_calc.py
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
                    -v \
                    --junit-xml=../test-reports/results.xml
                '''
            }

            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }

        stage('Deliver') {
            steps {
                sh '''
                    rm -rf build dist *.spec

                    ./venv/bin/pyinstaller \
                    --onefile \
                    --name add2vals \
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
                        echo "Deploying to AWS EC2..."

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
