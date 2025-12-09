//!groovy
pipeline {
    agent any
    
    tools {
        maven 'M2_HOME'
        jdk 'JAVA_HOME'
    }
    
    environment {
        // Configuration spécifique aux tests
        SPRING_PROFILES_ACTIVE = 'test'
        MAVEN_TEST_OPTS = '-Dspring.profiles.active=test'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Clean') {
            steps {
                sh 'mvn clean'
            }
        }
        
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('Tests Unitaires') {
            steps {
                script {
                    echo "🔧 Configuration des tests Spring Boot..."
                    
                    // Option 1: Exécuter avec profile test
                    try {
                        sh """
                            mvn test \
                            -Dspring.profiles.active=test \
                            -Dtest=!*IntegrationTest \
                            -DfailIfNoTests=false
                        """
                    } catch (Exception e) {
                        echo "⚠ Tests échoués, mais on continue le pipeline..."
                        // Marquer le build comme unstable au lieu de failed
                        currentBuild.result = 'UNSTABLE'
                    }
                    
                    // Option 2: Lire les rapports même en cas d'échec
                    sh '''
                        echo "=== RAPPORT DES TESTS ==="
                        if [ -d "target/surefire-reports" ]; then
                            echo "Fichiers de rapport trouvés:"
                            ls -la target/surefire-reports/
                            echo ""
                            echo "Détails de l'erreur:"
                            cat target/surefire-reports/*.txt 2>/dev/null | grep -A 20 "ERROR" || echo "Aucun détail d'erreur trouvé"
                        fi
                    '''
                }
            }
            
            post {
                always {
                    // Toujours publier les rapports JUnit
                    junit 'target/surefire-reports/*.xml'
                    
                    // Archiver les logs de test pour analyse
                    archiveArtifacts artifacts: 'target/surefire-reports/*.txt, target/surefire-reports/*.dump*', allowEmptyArchive: true
                }
            }
        }
        
        stage('Debug Tests') {
            when {
                expression { currentBuild.result == 'UNSTABLE' || currentBuild.result == 'FAILURE' }
            }
            steps {
                echo "🐛 DEBUG: Analyse des erreurs de test..."
                sh '''
                    echo "=== DÉTAILS DE L'ERREUR DE TEST ==="
                    
                    # Chercher les fichiers dump Java
                    if ls target/*.dump* 1> /dev/null 2>&1; then
                        echo "Fichiers dump trouvés:"
                        ls -la target/*.dump*
                        echo ""
                        echo "Extrait du premier dump:"
                        head -100 target/*.dump 2>/dev/null || echo "Impossible de lire le dump"
                    fi
                    
                    # Vérifier les logs de test
                    if [ -d "target/surefire-reports" ]; then
                        for file in target/surefire-reports/*.txt; do
                            if [ -f "$file" ]; then
                                echo ""
                                echo "=== Rapport: $(basename $file) ==="
                                grep -A 50 "Stacktrace:" "$file" || grep -A 50 "Caused by:" "$file" || tail -50 "$file"
                            fi
                        done
                    fi
                    
                    echo "=== CONFIGURATION SPRING ==="
                    find . -name "application*.properties" -o -name "application*.yml" | xargs ls -la
                '''
            }
        }
        
        stage('Build JAR (skip tests)') {
            steps {
                echo "📦 Construction du JAR (tests ignorés)..."
                sh 'mvn package -DskipTests'
            }
            
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                    sh 'ls -lh target/*.jar'
                }
            }
        }
        
        stage('SonarQube') {
            steps {
                echo "📊 Analyse SonarQube..."
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=tp-projet-2025 \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=${SONAR_TOKEN} \
                        -Dsonar.tests.exclusions=**/*Test.java \
                        -Dsonar.coverage.exclusions=**/*Test.java,**/test/** \
                        -DskipTests
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo "🔚 Pipeline terminé - Résultat: ${currentBuild.result}"
            
            // Nettoyage
            sh '''
                echo "=== ESPACE DISQUE ==="
                df -h .
                echo ""
                echo "=== FICHIERS GÉNÉRÉS ==="
                find target -type f -name "*.jar" -o -name "*.class" | wc -l
            '''
        }
        
        success {
            echo '✅ Pipeline terminé avec succès'
        }
        
        unstable {
            echo '⚠️ Pipeline instable (tests échoués)'
            script {
                // Notification pour tests instables
                emailext(
                    subject: "⚠️ Tests échoués: Build ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: "Les tests ont échoué mais le build a continué.\n\nConsultez: ${env.BUILD_URL}testReport/",
                    to: 'dev@example.com'
                )
            }
        }
        
        failure {
            echo '❌ Pipeline en échec'
        }
    }
}
