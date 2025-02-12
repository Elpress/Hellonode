pipeline {
    agent any

    stages {
        stage('Clone repository') {
            steps {
                checkout scm
            }
        }

        stage('Build image') {
            steps {
                script {
                    app = docker.build("releaseworks/hellonode")
                }
            }
        }

        stage('Scan image') {
            steps {
                script {
                    // Exécuter le scan avec Grype et capturer la sortie
                    def scanOutput = sh(script: "grype releaseworks/hellonode", returnStdout: true).trim()

                    // Afficher la sortie du scan
                    echo "Scan Output: ${scanOutput}"

                    // Optionnel : Sauvegarder la sortie dans un fichier
                    writeFile file: 'grype_scan_output.txt', text: scanOutput
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}

