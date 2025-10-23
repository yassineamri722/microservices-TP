pipeline {
    agent any
    tools {
        maven 'M2_HOME'
    }
    stages {
        stage('Checkout code') {
            steps {
                git branch: 'master', url: 'https://github.com/tligmohamed02/microservices-TP.git'
            }
        }

        stage('Build maven' ) {
            steps {
                // Cette étape crée le fichier .jar nécessaire pour le Dockerfile
                sh 'mvn clean install'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Construit l'image Docker localement en utilisant le numéro de build comme tag
                    sh "docker build -t my-country-service:$BUILD_NUMBER ."

                    // Utilise les credentials Jenkins pour se connecter à Docker Hub de manière sécurisée
                    withCredentials([string(credentialsId: 'dockerhub-pwd', variable: 'DOCKERHUB_PASSWORD')]) {
                        sh "docker login -u mohamed357 -p ${DOCKERHUB_PASSWORD}"
                    }

                    // Tague l'image avec le nom d'utilisateur Docker Hub pour la publication
                    sh "docker tag my-country-service:$BUILD_NUMBER mohamed357/my-country-service:$BUILD_NUMBER"

                    // Pousse l'image vers le registre Docker Hub
                    sh "docker push mohamed357/my-country-service:$BUILD_NUMBER"
                }
            }
        }

        stage('Deploy micro-service') {
            steps {
                // Arrête et supprime les anciens conteneurs pour éviter les conflits de port
                sh 'docker rm -f $(docker ps -aq --filter "ancestor=mohamed357/my-country-service") || true'

                // Lance un nouveau conteneur avec la nouvelle image
                sh "docker run -d -p 8082:8082 my-country-service:$BUILD_NUMBER"
            }
        }
    }
}
