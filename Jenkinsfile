node {
    properties([
        pipelineTriggers([
            [$class: 'SCMTrigger', scmpoll_spec: '*/2 * * * *'] // Menjalankan polling setiap 2 menit
        ])
    ])

    try {
        stage('Checkout') {
            checkout scm
        }

        stage('Build') {
            docker.image('python:3.11.4-alpine3.18').inside {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }

        stage('Test') {
            docker.image('qnib/pytest').inside {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
        }

         stage('Manual Approval') {
            input(id: 'approval', message: 'Lanjutkan ke tahap Deploy?', parameters: [
                string(defaultValue: 'Proceed', description: 'Pilih Proceed untuk melanjutkan atau Abort untuk menghentikan', name: 'action')
            ])
        }

        stage('Deploy') {
            if (approval == 'Proceed') {
                echo 'Aplikasi berhasil di-deploy.'
                sleep time: 60, unit: 'SECONDS' // Menjeda eksekusi selama 1 menit.
            } else {
                echo 'Eksekusi pipeline dihentikan oleh pengguna.'
                currentBuild.result = 'ABORTED'
                error('Pipeline dihentikan oleh pengguna')
            }
        }

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
