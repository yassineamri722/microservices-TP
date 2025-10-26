pipeline {
    agent any

    tools {
        maven 'M2_HOME'
        jdk 'JDK21'
    }

    stages {
        stage('Checkout code') {
            steps {
                git branch: 'master', url: 'https://github.com/tligmohamed02/microservices-TP.git'
            }
        }

        stage('Compile, test code and package' ) {
            steps {
                sh 'mvn clean install'
            }
            post {
                success {
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    sh "mvn sonar:sonar -Dsonar.projectKey=country-service"
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-credentials', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASSWORD')]) {
                    configFileProvider([configFile(fileId: 'nexus-maven-settings', variable: 'MAVEN_SETTINGS')]) {
                        sh "mvn deploy -s \"${env.MAVEN_SETTINGS}\""
                    }
                }
            }
        }

        stage('Deploy from Nexus') {
            steps {
                configFileProvider([configFile(fileId: 'nexus-maven-settings', variable: 'MAVEN_SETTINGS')]) {
                    script {
                        echo 'Déploiement de lartefact depuis Nexus...'

                        def pom = readMavenPom file: 'pom.xml'
                        def groupId = pom.groupId
                        def artifactId = pom.artifactId
                        def version = pom.version
                        def packaging = pom.packaging
                        def downloadDir = "target/dependency"

                        sh """
                            rm -rf ${downloadDir}
                            mkdir -p ${downloadDir}

                            mvn org.apache.maven.plugins:maven-dependency-plugin:3.1.2:copy \\
                                -Dartifact=${groupId}:${artifactId}:${version}:${packaging} \\
                                -DoutputDirectory=${downloadDir} \\
                                -s "${env.MAVEN_SETTINGS}"
                        """

                        sh """
                            sudo mkdir -p /var/www/my-app
                            sudo chown -R jenkins:jenkins /var/www/my-app

                            echo "Copie du fichier trouvé dans ${downloadDir}..."
                            cp \$(find ${downloadDir} -name '*.jar') /var/www/my-app/app.jar

                            # --- CHOIX DE LA MÉTHODE DE DÉPLOIEMENT ---
                            # Méthode 1: Non fiable, mais ne nécessite pas de config systemd
                            JENKINS_NODE_COOKIE=dontKillMe nohup java -jar /var/www/my-app/app.jar > /var/www/my-app/app.log 2>&1 &

                            # Méthode 2: Robuste et recommandée
                            #echo "Redémarrage du service my-app via systemd..."
                            #sudo systemctl restart my-app.service
                        """

                        echo "Application redémarrée avec la version ${version} de Nexus."
                    }
                }
            }
        }
    } // <<< Fin du bloc 'stages'. Une seule accolade ici.

    post {
        always {
            echo 'Pipeline finished.'
            cleanWs()
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
