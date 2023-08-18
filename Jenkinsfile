node {
    stage('Build') {
        docker.image('python:3.11.4-alpine3.18').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }
}
