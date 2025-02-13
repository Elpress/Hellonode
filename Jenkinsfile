pipeline {
    agent any

    environment {
        IMAGE_NAME = "getintodevops/hellonode"
        IMAGE_TAG = "latest"
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials' // Remplacez par l'ID de vos credentials Docker Hub
    }

    stages {
        stage('Clone repository') {
            steps {
                // Clone le dépôt Git
                checkout scm
            }
        }

        stage('Build image') {
            steps {
                script {
                    // Construire l'image Docker
                    app = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push image to Docker Hub') {
            steps {
                script {
                    // Pousser l'image vers Docker Hub
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        app.push(IMAGE_TAG)
                        app.push('latest') // Optionnel : tagger l'image comme 'latest'
                    }
                }
            }
        }

        stage('Scan image') {
            steps {
                script {
                    // Scanner l'image Docker depuis Docker Hub avec Grype
                    def scanOutput = sh(script: "grype ${IMAGE_NAME}:${IMAGE_TAG}", returnStdout: true).trim()

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
            // Nettoyer l'espace de travail
            cleanWs()
        }
    }
}

