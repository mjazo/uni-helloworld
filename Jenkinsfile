pipeline {
    agent any

    stages {
        stage('GetCode') {
            steps {
                git branch: 'develop', url: 'https://github.com/mjazo/uni-helloworld.git'
                sh '''
                    ls -la
                    echo $WORKSPACE
                '''
            }
        }
        stage('Unit') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '''
                        echo $WORKSPACE
                        export PYTHONPATH=$WORKSPACE
                        python3 -m pytest --junitxml=result-unit.xml test/unit
                    '''
                    junit 'result*.xml'
                }
            }
        }
         stage('Coverage') {
            steps {
                sh '''
                    python3 -m coverage run --source=app --omit=app/__init__.py,app/api.py -m pytest test/unit
                    python3 -m coverage xml
                '''
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,0,80', lineCoverageTargets: '100,0,90'
                }
            }
        }
        stage('Static') {
            steps {
                sh '''
                    python3 -m flake8 --format=pylint --exit-zero app >flake8.out
                '''
                recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 15, type: 'TOTAL', unstable: true], [threshold: 16, type: 'TOTAL', unstable: false]]
            }
        }
        stage('Security') {
            steps {
                sh '''
                    python3 -m bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: {severity}: {test_id}: {msg}"
                '''
                recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold: 1, type: 'TOTAL', unstable: true], [threshold: 2, type: 'TOTAL', unstable: false]]
            }
        }
        stage('Performance'){
            steps {
                sh '''
                    export FLASK_APP=app/api.py
                    flask run -p 5000 &
                    sleep 5
                    jmeter -n -t test/jmeter/flask.jmx -f -l flask.jtl
                '''
                perfReport sourceDataFiles: 'flask.jtl'
            }
        }
    }
}