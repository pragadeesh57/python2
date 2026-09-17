pipeline {
    agent any

    stages {

        stage('Install Dependencies') {
            steps {
                sh '''
                    rm -rf venv
                    python3 -m venv venv
                    ./venv/bin/python -m pip install --upgrade pip
                    ./venv/bin/python -m pip install pytest pyinstaller
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

                    ./venv/bin/python -m pytest \
                    sources/test_calc.py \
                    -v \
                    --junit-xml=test-reports/results.xml
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

                    ./venv/bin/python -m PyInstaller \
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
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'r1',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {

                    sh '''
                        set -e

                        echo "Deploying to AWS EC2..."

                        chmod 600 "$SSH_KEY"

                        scp -o StrictHostKeyChecking=no \
                            -i "$SSH_KEY" \
                            dist/add2vals \
                            "$SSH_USER@54.253.129.59:/tmp/add2vals"

                        ssh -o StrictHostKeyChecking=no \
                            -i "$SSH_KEY" \
                            "$SSH_USER@54.253.129.59" \
                            'sudo mv /tmp/add2vals /usr/local/bin/add2vals && sudo chmod +x /usr/local/bin/add2vals'

                        echo "Deployment completed successfully!"
                    '''
                }
            }
        }
    }
}
