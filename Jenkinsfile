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

        stage('Deploy') {
            steps {
                script {
                    // Lakukan deploy aplikasi ke tempat yang sesuai, misalnya: docker-compose, Kubernetes, atau server langsung.
                    // Gantilah dengan perintah atau skrip yang sesuai dengan lingkungan deploy Anda.
                    echo 'Aplikasi berhasil di-deploy.'
                    sleep time: 60, unit: 'SECONDS' // Menjeda eksekusi selama 1 menit.
                }
            }
        }

        stage('Manual Approval') {
            steps {
                script {
                    def userInput = input(
                        id: 'userInput',
                        message: 'Lanjutkan ke tahap Deploy?',
                        parameters: [
                            [$class: 'ChoiceParameterDefinition', choices: 'Proceed\nAbort', description: 'Pilih opsi', name: 'ACTION']
                        ]
                    )

                    if (userInput == 'Abort') {
                        error('Eksekusi pipeline dihentikan oleh pengguna.')
                    }
                }
            }
        }
        
        stage('Monitoring') {
            steps {
                // Lakukan konfigurasi Prometheus dan Grafana sesuai kriteria 5.
                // Gantilah dengan perintah atau skrip yang sesuai dengan konfigurasi Anda.
                echo 'Konfigurasi Prometheus dan Grafana'
            }
        }
        
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}