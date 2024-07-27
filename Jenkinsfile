pipeline {
    agent any
    stages {

        stage('Gitleaks-Scan') {
            agent {
                docker {
                    image 'zricethezav/gitleaks'
                    args '--entrypoint="" -u root -v ${WORKSPACE}:/src'
                }
            }
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    script {
                        sh "gitleaks detect --verbose --source . -f json -r report_gitleaks.json"
                        sh "ls -la"
                        archiveArtifacts artifacts: "report_gitleaks.json"
                        //stash includes: 'report_gitleaks.json', name: 'report_gitleaks.json'
                    }
                }
            }
        }

        stage('build'){
            steps {
                echo 'Compilando el codigo..'
            }
        }

        stage('SonarQube'){
            environment {
                scannerHome = tool 'SonarScanner'
            }
            steps {
                withSonarQubeEnv(credentialsId: 'TokenSonarQube	', installationName: 'sonarqube'){
                    sh "${scannerHome}/bin/SonarScanner"
                }
            }
        }

        stage('Pruebas'){
            steps{
                echo 'Ejecutando pruebas '
            }
        }

        stage('Despliegue'){
            steps{
                echo 'Desplegando aplicacion '
            }
        }
    }
}