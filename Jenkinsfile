:::writing{variant="standard" id="35184"}
pipeline {
    agent any

    stages {
        stage('Analyse des anti-patterns') {
            steps {
                script {
                    int detectorExitCode = bat(
                        returnStatus: true,
                        script: '''
                            @echo off

                            set "PYTHON=C:\Users\Aorus PC\AppData\Local\Programs\Python\Python312\python.exe"
                            set "PYTHON_SCRIPTS=C:\Users\Aorus PC\AppData\Local\Programs\Python\Python312\Scripts"

                            echo === Python utilise par Jenkins ===
                            "%PYTHON%" --version
                            if errorlevel 1 exit /b %ERRORLEVEL%

                            echo === Pip utilise par Jenkins ===
                            "%PYTHON%" -m pip --version
                            if errorlevel 1 exit /b %ERRORLEVEL%

                            echo === Installation du detecteur ===
                            "%PYTHON%" -m pip install --upgrade git+https://github.com/AmrAzirar/microservice-antipattern-detector.git
                            if errorlevel 1 exit /b %ERRORLEVEL%

                            echo === Lancement de l analyse ===
                            "%PYTHON_SCRIPTS%\antipattern-detect.exe" --path . > detector-output.txt 2>&1
                            exit /b %ERRORLEVEL%
                        '''
                    )

                    String output = fileExists('detector-output.txt')
                        ? readFile('detector-output.txt')
                        : ''

                    echo '========== RESULTAT DU DETECTEUR =========='
                    echo output
                    echo '==========================================='

                    boolean criticalFound = (output =~ /(?i)\bCRITICAL\b/).find()

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
:::