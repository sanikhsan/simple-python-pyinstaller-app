node {
    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }  
    }
    
    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --junit-xml test-reports/results.xml ./sources/test_calc.py' 
            junit 'test-reports/results.xml'
        }
    }

    stage('Manual Approval') {
        input message: 'Lanjutkan ke tahap Deploy? (Klik "Proceed" untuk melanjutkan) dan (Klik "Abort" untuk mengakhiri)'
        sh 'echo "Melanjutkan ke Tahap Deploy"'
    }

    withEnv(['VOLUME=$(pwd)/sources:/src',
             'IMAGE=cdrx/pyinstaller-linux:python2']) {
                stage('Deploy') {
                    unstash(name: 'compiled-results')
                        sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                        archiveArtifacts "sources/dist/add2vals"
                        sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
                        sleep(time:60,unit:"SECONDS")
                }
            }
}