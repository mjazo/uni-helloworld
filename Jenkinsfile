pipeline {
    agent none

    stages {
        stage('GetCode') {
            agent { label 'agent1' }
            steps {
                git branch: 'develop', url: 'https://github.com/mjazo/uni-helloworld.git'
                sh '''
                    ls -la
                    echo $WORKSPACE
                '''
                stash includes: '**/*', name: 'files'
            }
        }
        stage('Build') {
            agent { label 'agent1' }
            steps {
                echo 'NO HAY QUE HACER, ESTO ES PYTHON'
                echo '----> USING AGENT 1 <----'
                sh '''
                    whoami
                    hostname
                    echo $WORKSPACE
                '''
            }
        }
        stage('Tests') {
            parallel {
                stage('Unit') {
                    agent { label 'agent2' }
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            unstash 'files'
                            echo '----> USING AGENT 2 <----'
                            sh '''
                                whoami
                                hostname
                                echo $WORKSPACE
                                export PYTHONPATH=$WORKSPACE
                                python3 -m pytest --junitxml=result-unit.xml test/unit
                            '''
                            stash includes: '**/*', name: 'files'
                        }
                    }
                }
                stage('Rest') {
                    agent { label 'agent2' }
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            unstash 'files'
                            sh '''
                                whoami
                                hostname
                                echo $WORKSPACE
                                export FLASK_APP=app/api.py
                                flask run -p 5001 &
                                sleep 2
                                python3 -m pytest --junitxml=result-rest.xml test/rest
                            '''
                            stash includes: '**/*', name: 'files'
                        }
                    }
                }
            }
        }
        stage('Results') {
            agent { label 'agent3' }
            steps {
                unstash 'files'
                echo '----> USING AGENT 3 <----'
                sh '''
                    whoami
                    hostname
                    echo $WORKSPACE
                '''
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    junit 'result*.xml'
                }
            }
        }
    }
}