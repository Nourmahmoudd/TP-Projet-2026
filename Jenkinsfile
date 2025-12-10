pipeline {
    agent any
    
    tools {
        maven 'M2_HOME'
        jdk 'JAVA_HOME'
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
                        -DskipTests
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo "🔚 Pipeline terminé - Résultat: ${currentBuild.result}"
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

        failure {
            echo '❌ Pipeline en échec'
        }
    }
}
