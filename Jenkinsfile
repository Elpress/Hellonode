pipeline {
    agent any

    environment {
        IMAGE_NAME = "getintodevops/hellonode"
        IMAGE_TAG = "latest"
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials' // ID de vos credentials Docker Hub
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

        stage('Push image') {
            steps {
                script {
                    // Pousser l'image avec deux tags :
                    // 1. Le numéro de build incrémental de Jenkins
                    // 2. Le tag 'latest'
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
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

                    // Sauvegarder la sortie dans un fichier
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

