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

        stage('Deliver') {
            node {
                def VOLUME = "${pwd()}/sources:/src"
                def IMAGE = "cdrx/pyinstaller-linux:python2"

                try {
                    // Set up the environment and create a directory for the build
                    def BUILD_ID = UUID.randomUUID().toString()
                    dir(BUILD_ID) {
                        // Restore the stashed compiled results
                        unstash(name: 'compiled-results')

                        // Run PyInstaller inside the Docker container
                        sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                    }

                    // Archive the built executable
                    archiveArtifacts artifacts: "${BUILD_ID}/sources/dist/add2vals", allowEmptyArchive: true

                } catch (Exception e) {
                    currentBuild.result = 'FAILURE'
                    throw e

                } finally {
                    // Clean up: remove build artifacts
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf ${BUILD_ID}'"
                }
            }
        }

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
