pipeline {
    agent any

    tools {
        maven 'M2_HOME'
        jdk 'JAVA_HOME'
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Build JAR') {
            steps { sh 'mvn clean package -DskipTests' }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=tp-projet-2025 \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=${SONAR_TOKEN}
                    '''
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        docker login -u $DOCKER_USER -p $DOCKER_PASS
                        docker build -t nourmahmoudd/tp-2025:1.0 .
                        docker push nourmahmoudd/tp-2025:1.0
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withEnv(['KUBECONFIG=/var/lib/jenkins/.kube/config']) {
                    sh '''
                      kubectl apply -f k8s/mysql-secret.yaml
                      kubectl apply -f k8s/mysql-deployment.yaml
                      kubectl apply -f k8s/mysql-service.yaml
        
                      kubectl apply -f k8s/deployment.yaml
                      kubectl apply -f k8s/service.yaml
                      kubectl apply -f k8s/metrics-service.yaml
                    '''
                }
            }
        }
    }

post {
        success {
            archiveArtifacts artifacts: 'target/*.jar'

            emailext (
                subject: "✅ SUCCESS : Pipeline Jenkins - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <h2>Pipeline exécuté avec succès 🎉</h2>
                    <p><b>Projet :</b> ${env.JOB_NAME}</p>
                    <p><b>Build :</b> #${env.BUILD_NUMBER}</p>
                    <p><b>Status :</b> SUCCESS ✅</p>
                    <p><b>URL :</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """,
                to: "nouurrmahmoudd@gmail.com",
                mimeType: 'text/html'
            )
        }

        failure {
            emailext (
                subject: "❌ FAILURE : Pipeline Jenkins - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <h2>Pipeline échoué ❌</h2>
                    <p><b>Projet :</b> ${env.JOB_NAME}</p>
                    <p><b>Build :</b> #${env.BUILD_NUMBER}</p>
                    <p><b>Status :</b> FAILURE ❌</p>
                    <p><b>Logs :</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """,
                to: "nouurrmahmoudd@gmail.com",
                mimeType: 'text/html'
            )
        }
    }
}
