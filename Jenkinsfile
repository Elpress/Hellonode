pipeline {
    agent any

    environment {
        IMAGE_NAME = "houmeyra/hellonode"
        IMAGE_TAG = "latest"
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials' // ID de vos credentials Docker Hub
        EMAIL_RECIPIENTS = 'meweelvis.balo@orange-sonatel.com,aicha.ba1@orange-sonatel.com' // Remplacez par les adresses emails des destinataires
    }

    stages {
        stage('Clone repository') {
            steps {
                // Clone le dépôt Git
                checkout scm
            }
        }

        stage('Check disk space') {
            steps {
                script {
                    // Vérifiez l'espace disque disponible
                    def diskSpace = sh(script: 'df -h /home', returnStdout: true).trim()
                    echo "Disk Space: ${diskSpace}"

                    // Vérifiez si l'espace disque est suffisant
                    if (diskSpace.contains("100%") || diskSpace.contains("99%")) {
                        error "Insufficient disk space available to build the Docker image."
                    }
                }
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

        stage('Send email notification') {
            steps {
                script {
                    // Envoyer un email avec les résultats du scan
                    emailext (
                        to: "${EMAIL_RECIPIENTS}",
                        subject: "Résultats du Scan de Sécurité pour ${IMAGE_NAME}:${IMAGE_TAG}",
                        body: "Veuillez trouver ci-joint les résultats du scan de sécurité pour l'image Docker ${IMAGE_NAME}:${IMAGE_TAG}.\n\n${scanOutput}",
                        attachments: 'grype_scan_output.txt'
                    )
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
