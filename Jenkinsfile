// Jenkinsfile - Pipeline de "Sistema Alumnos" (OCY1102)
//
// Esta es la versión BASE: solo compila y prueba.
// En la guía irás pegando etapas en los marcadores  >>> [ETIQUETA]  de más abajo.
pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    // >>> [A-TRIGGER] (Parte A, paso A4): aquí va el bloque "triggers"

    environment {
        // pipenv crea el entorno virtual (.venv) dentro del workspace del job
        PIPENV_VENV_IN_PROJECT    = '1'
        PIPENV_IGNORE_VIRTUALENVS = '1'
        PIPENV_NOSPIN             = '1'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    // Versión = 1.0.<nº de build>+<commit corto>. Sirve para la trazabilidad (Parte B)
                    def sha = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    env.GIT_SHA     = sha
                    env.APP_VERSION = "1.0.${env.BUILD_NUMBER}+${sha}"
                }
                echo "Versión de este build: ${env.APP_VERSION}"
                // Cada build parte limpio: sin reportes viejos y con un entorno virtual nuevo (reproducible)
                sh 'rm -rf .venv reports && mkdir -p reports'
                // --deploy: falla si Pipfile.lock no coincide con Pipfile (evita instalaciones "a ciegas")
                sh 'pipenv install --dev --deploy'
                sh 'pipenv run python -m compileall -q app.py'
            }
        }

        stage('Test') {
            steps {
                sh 'pipenv run pytest --junitxml=reports/junit.xml'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'reports/junit.xml'
                }
            }
        }

        // >>> [A-ETAPAS] (Parte A, paso A1): aquí van las etapas "Analyze" y "Dependency Management"

        // >>> [B-DOCS] (Parte B, paso B3): aquí va la etapa "Generate Documentation"
        stage('Generate Documentation') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                    sh '''#!/bin/bash
                        set -eu
                        mkdir -p reports
                        rm -rf docs/html reports/doxygen_warnings.log
                        doxygen Doxyfile
                        if [ -s reports/doxygen_warnings.log ]; then
                            echo "Doxygen encontró código sin documentar o mal documentado:"
                            cat reports/doxygen_warnings.log
                            exit 1
                        fi
                        echo "Documentación generada sin advertencias en docs/html"
                    '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'docs/html/**', allowEmptyArchive: true
                }
            }
        }
        // >>> [B-VERSION] (Parte B, paso B4): aquí va la etapa "Version Control"

        // >>> [B-DEPLOY] (Parte B, paso B5): aquí va la etapa "Deploy"
    }

    post {
        always {
            archiveArtifacts artifacts: 'reports/**', allowEmptyArchive: true
        }
        // >>> [A-POST] (Parte A, paso A4): aquí van las alertas "unstable" y "failure"
    }
}

// >>> [A-FUNCION] (Parte A, paso A4): aquí va la función notificar()
