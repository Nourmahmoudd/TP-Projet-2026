pipeline {
    agent any

    tools {
        jdk 'JAVA_HOME'      // à adapter si tu utilises une autre version dans Jenkins
        maven 'M2_HOME'      // le nom que tu as donné à Maven dans Jenkins
    }

    environment {
        SONAR_TOKEN = credentials('sonar-token')  // ton token SonarQube
    }

    stages {

        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/Nourmahmoudd/TP-Projet-2026.git',
                        credentialsId: 'git-credentials'
                    ]]
                ])
            }
        }

        stage('Clean') {
            steps {
                dir('TP-Projet-2026') {   // chemin corrigé
                    sh "mvn clean"
                }
            }
        }

        stage('Compile') {
            steps {
                dir('TP-Projet-2025') {
                    sh "mvn compile"
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('TP-Projet-2026') {
                    withSonarQubeEnv('sonarqube') {
                        sh """
                        mvn sonar:sonar \
                          -Dsonar.projectKey=tp-projet-2026 \
                          -Dsonar.host.url=http://192.168.33.10:9000 \
                          -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Package') {
            steps {
                dir('TP-Projet-2025') {
                    sh "mvn package -DskipTests"
                }
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'TP-Projet-2025/target/*.jar'
        }
    }
}