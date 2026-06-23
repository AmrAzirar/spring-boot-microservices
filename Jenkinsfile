pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        PYTHON = 'C:/Users/Aorus PC/AppData/Local/Programs/Python/Python312/python.exe'
        PYTHON_SCRIPTS = 'C:/Users/Aorus PC/AppData/Local/Programs/Python/Python312/Scripts'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Analyse des anti-patterns') {
            steps {
                script {
                    int setupExitCode = bat(
                        returnStatus: true,
                        script: '''
@echo off
echo === Python utilise par Jenkins ===
"%PYTHON%" --version
if errorlevel 1 exit /b 1

echo === Pip utilise par Jenkins ===
"%PYTHON%" -m pip --version
if errorlevel 1 exit /b 1

echo === Installation du detecteur ===
"%PYTHON%" -m pip install --upgrade git+https://github.com/AmrAzirar/microservice-antipattern-detector.git
if errorlevel 1 exit /b 1

exit /b 0
'''
                    )

                    if (setupExitCode != 0) {
                        error("Preparation du detecteur echouee (code ${setupExitCode}).")
                    }

                    int detectorExitCode = bat(
                        returnStatus: true,
                        script: '''
@echo off
echo === Lancement de l analyse architecturale ===
"%PYTHON_SCRIPTS%/antipattern-detect.exe" --path . > detector-output.txt 2>&1
exit /b %ERRORLEVEL%
'''
                    )

                    String output = fileExists('detector-output.txt')
                        ? readFile('detector-output.txt')
                        : ''

                    echo '========== RESULTAT DU DETECTEUR =========='
                    echo output
                    echo '==========================================='

                    if (output.toUpperCase().contains('CRITICAL')) {
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
