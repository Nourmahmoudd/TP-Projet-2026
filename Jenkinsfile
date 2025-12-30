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

        stage('Build JAR') {
            steps { 
                sh 'mvn clean package -DskipTests' 
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withEnv(['KUBECONFIG=/var/lib/jenkins/.kube/config']) {
                    sh '''
                      # Déploiement MySQL
                      kubectl apply -f k8s/mysql-secret.yaml
                      kubectl apply -f k8s/mysql-deployment.yaml
                      kubectl apply -f k8s/mysql-service.yaml
        
                      # Déploiement de ton application TP-2025
                      kubectl apply -f k8s/deployment.yaml
                      kubectl apply -f k8s/service.yaml
                      kubectl apply -f k8s/metrics-service.yaml
        
                      # Déploiement Prometheus
                      kubectl apply -f k8s/prometheus-deployment.yaml
                      kubectl apply -f k8s/prometheus-service.yaml
                      kubectl apply -f k8s/prometheus-configmap.yaml
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
                        docker logout
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
            echo '✅ Pipeline terminé avec succès' 
        }
        failure { 
            echo '❌ Pipeline échoué' 
        }
    }
}
