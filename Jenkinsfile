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
                    // Construire l'image Docker
                    app = docker.build("releaseworks/hellonode")
                }
            }
        }

        stage('Save image locally') {
            steps {
                script {
                    // Enregistrer l'image localement
                    sh "docker save -o hellonode.tar releaseworks/hellonode"
                }
            }
        }

        stage('Scan image') {
            steps {
                script {
                    // Scanner l'image Docker locale avec Grype
                    def scanOutput = sh(script: "grype oci-archive:hellonode.tar", returnStdout: true).trim()

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

