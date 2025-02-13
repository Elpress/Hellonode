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

	stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
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

