pipeline {
    agent any

    environment {
        VERSION_TAG = "3.0.0"
        IMAGE_NAME = "demo-curso"
        DOCKER_HUB_REGISTRY = credentials('registro-hub')
    }

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
                sh "docker build -t $DOCKER_HUB_REGISTRY/$IMAGE_NAME:$VERSION_TAG ."
            }
        }

        stage('SonarQube'){
            environment {
                scannerHome = tool 'SonarScanner'
            }
            steps {
                withSonarQubeEnv(credentialsId: 'SonarID', installationName: 'SonarServer'){
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Trivy-Scan') {
            agent {
                docker {
                    image 'aquasec/trivy:0.48.1'
                    args '--entrypoint="" -u root -v /var/run/docker.sock:/var/run/docker.sock -v ${WORKSPACE}:/src'
                }
            }
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    script {
                        sh "trivy image --format json --output report_trivy.json $DOCKER_HUB_REGISTRY/$IMAGE_NAME:$VERSION_TAG"
                        archiveArtifacts artifacts: "report_trivy.json"
                    }
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