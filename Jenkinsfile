pipeline {
    agent any

    stages {
        stage('GetCode') {
            steps {
                cleanWs()
                git branch: 'develop', url: 'https://github.com/mjazo/uni-helloworld.git'
                sh '''
                    ls -la
                    echo $WORKSPACE
                '''
                stash includes: '**/*', name: 'source'
            }
        }
        stage('Unit') {
            steps {
                cleanWs()
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    unstash 'source'
                    fileOperations([
                        fileDownloadOperation(userName: '', password: '', proxyHost: '', proxyPort: '', targetFileName: 'wiremock.jar', url: 'https://repo1.maven.org/maven2/org/wiremock/wiremock-standalone/3.10.0/wiremock-standalone-3.10.0.jar', targetLocation: '.',)
                    ])
                    sh '''
                        echo $WORKSPACE
                        export PYTHONPATH=$WORKSPACE
                        java -jar wiremock.jar --port 9090 --root-dir test/wiremock &
                        export FLASK_APP=app/api.py
                        flask run -p 5001 &
                        sleep 7
                        python3 -m pytest test/unit test/rest --junitxml=result.xml
                    '''
                    junit 'result.xml'
                    stash includes: '**/*', name: 'source'
                }
            }
        }
        stage('Coverage') {
            steps {
                cleanWs()
                unstash 'source'
                sh '''
                    python3 -m coverage run --source=app --omit=app/__init__.py,app/api.py -m pytest test/unit
                    python3 -m coverage xml
                '''
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    cobertura coberturaReportFile: 'coverage.xml', onlyStable: false, conditionalCoverageTargets: '100,0,80', lineCoverageTargets: '100,0,95'
                }
            }
        }
        stage('Static') {
            steps {
                cleanWs()
                unstash 'source'
                sh '''
                    python3 -m flake8 --format=pylint --exit-zero app >flake8.out
                '''
                recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 15, type: 'TOTAL', unstable: true], [threshold: 16, type: 'TOTAL', unstable: false]]
            }
        }
        stage('Security') {
            steps {
                cleanWs()
                unstash 'source'
                sh '''
                    python3 -m bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: {severity}: {test_id}: {msg}"
                '''
                recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]]
            }
        }
        stage('Performance'){
            steps {
                cleanWs()
                unstash 'source'
                sh '''
                    export FLASK_APP=app/api.py
                    flask run -p 5000 &
                    sleep 5
                    jmeter -n -t test/jmeter/testplan.jmx -f -l flask.jtl
                '''
                perfReport sourceDataFiles: 'flask.jtl'
            }
        }
    }
}