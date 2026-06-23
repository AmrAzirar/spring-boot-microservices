pipeline {
    agent any

    stages {
        stage('Analyse des anti-patterns') {
            steps {
                checkout scm

                bat 'python --version'
                bat 'python -m pip --version'

                bat '''
                    python -m pip install --upgrade git+https://github.com/AmrAzirar/microservice-antipattern-detector.git
                '''

                script {
                    int detectorExitCode = bat(
                        returnStatus: true,
                        script: '''
                            @echo off
                            antipattern-detect --path . > detector-output.txt 2>&1
                            exit /b %ERRORLEVEL%
                        '''
                    )

                    String output = readFile('detector-output.txt')

                    echo '========== RESULTAT DU DETECTEUR =========='
                    echo output
                    echo '==========================================='

                    boolean criticalFound =
                        (output =~ /(?i)\bCRITICAL\b/).find()

                    if (criticalFound) {
                        error('QUALITY GATE FAILED : anti-pattern CRITICAL detecte.')
                    }

                    if (detectorExitCode != 0) {
                        error("Le detecteur a echoue avec le code ${detectorExitCode}.")
                    }

                    echo 'QUALITY GATE PASSED : aucun anti-pattern CRITICAL detecte.'
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'detector-output.txt, **/report.html, **/*.json, **/*.csv, **/*.sarif, **/*.png',
                             allowEmptyArchive: true
        }

        success {
            echo 'Analyse terminee : pipeline valide.'
        }

        failure {
            echo 'Pipeline bloque : consulte detector-output.txt et les artefacts.'
        }
    }
}