pipeline {
    agent any

    tools {
        // CORRECTION: Noms d'outils Jenkins (à vérifier dans Jenkins)
        jdk 'JAVA_HOME'           // Remplacez par le nom exact dans Jenkins
        maven 'M2_HOME'     // Remplacez par le nom exact dans Jenkins
    }

    environment {
        // Variables d'environnement
        PROJECT_NAME = 'TP-Projet-2025'
        GIT_REPO = 'https://github.com/Nourmahmoudd/TP-Projet-2026.git'
        SONAR_HOST_URL = 'http://192.168.33.10:9000'
        SONAR_PROJECT_KEY = 'tp-projet-2025'  // À créer dans SonarQube
    }

    stages {
        // ÉTAPE 1: Récupération du code
        stage('Checkout') {
            steps {
                echo "📥 Récupération du code depuis ${GIT_REPO}"
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        // VOTRE repo GitHub
                        url: 'https://github.com/Nourmahmoudd/TP-Projet-2026.git',
                        credentialsId: 'git-credentials'
                    ]]
                ])

                // Vérification
                sh '''
                    echo "=== VÉRIFICATION CHECKOUT ==="
                    echo "Dossier courant: $(pwd)"
                    echo "Contenu:"
                    ls -la
                    echo "Fichiers Java:"
                    find . -name "*.java" | wc -l
                '''
            }
        }

        // ÉTAPE 2: Nettoyage
        stage('Clean') {
            steps {
                echo "🧹 Nettoyage du projet..."
                // PAS besoin de dir() car checkout met les fichiers à la racine
                sh "mvn clean"
            }
        }

        // ÉTAPE 3: Compilation
        stage('Compile') {
            steps {
                echo "🔨 Compilation..."
                sh "mvn compile"

                post {
                    success {
                        sh '''
                            echo "✅ Compilation réussie"
                            echo "Fichiers .class générés:"
                            find target/classes -name "*.class" | wc -l
                        '''
                    }
                    failure {
                        echo "❌ Échec de compilation"
                        error("Arrêt du pipeline")
                    }
                }
            }
        }

        // ÉTAPE 4: Tests (AJOUT IMPORTANT!)
        stage('Test') {
            steps {
                echo "🧪 Exécution des tests..."
                sh "mvn test"
            }

            post {
                always {
                    // Publier les résultats JUnit
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        // ÉTAPE 5: Analyse SonarQube
        stage('SonarQube Analysis') {
            steps {
                echo "📊 Analyse SonarQube..."

                script {
                    // Deux méthodes au cas où
                    try {
                        // Méthode avec plugin Jenkins
                        withSonarQubeEnv('sonarqube') {
                            sh """
                                mvn sonar:sonar \
                                  -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                                  -Dsonar.projectName="${PROJECT_NAME}" \
                                  -Dsonar.host.url=${SONAR_HOST_URL}
                            """
                        }
                    } catch (Exception e) {
                        echo "⚠ Utilisation alternative avec credentials"
                        withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                            sh """
                                mvn sonar:sonar \
                                  -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                                  -Dsonar.projectName="${PROJECT_NAME}" \
                                  -Dsonar.host.url=${SONAR_HOST_URL} \
                                  -Dsonar.login=${SONAR_TOKEN}
                            """
                        }
                    }
                }
            }

            post {
                success {
                    echo "✅ Analyse SonarQube terminée"
                    echo "📊 Rapport disponible: ${SONAR_HOST_URL}/dashboard?id=${SONAR_PROJECT_KEY}"
                }
            }
        }

        // ÉTAPE 6: Génération du JAR
        stage('Package') {
            steps {
                echo "📦 Génération du JAR..."
                sh "mvn package -DskipTests"

                script {
                    // Vérifier le JAR généré
                    sh '''
                        echo "=== ARTÉFACTS GÉNÉRÉS ==="
                        if ls target/*.jar 1> /dev/null 2>&1; then
                            for jar in target/*.jar; do
                                echo "📁 $(basename $jar)"
                                echo "   Taille: $(du -h "$jar" | cut -f1)"
                            done
                        else
                            echo "❌ Aucun JAR généré!"
                        fi
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "🔚 Pipeline terminé - Build #${BUILD_NUMBER}"
        }

        success {
            echo "🎉 BUILD RÉUSSI!"

            // Archiver le JAR
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true

            // Afficher les informations
            sh '''
                echo "========================================="
                echo "         ✅ BUILD TERMINÉ ✅            "
                echo "========================================="
                echo "Artefacts disponibles dans Jenkins"
                echo "SonarQube: http://192.168.33.10:9000"
                echo "========================================="
            '''
        }

        failure {
            echo "💥 BUILD ÉCHOUÉ!"
            echo "Consultez les logs pour détails"
        }
    }
}
