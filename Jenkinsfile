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
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withEnv(['KUBECONFIG=/var/lib/jenkins/.kube/config']) {
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    sh 'kubectl apply -f k8s/service.yaml'
                }
            }
        }
    }

post {
    success {
        echo '✅ Pipeline terminé avec succès'
        mail to: 'ton.email@exemple.com',
             subject: "Pipeline SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
             body: "Le build ${env.JOB_NAME} #${env.BUILD_NUMBER} s'est terminé avec succès.\nVoir Jenkins: ${env.BUILD_URL}"
    }
    failure {
        echo '❌ Pipeline échoué'
        mail to: 'nour.mahmoud@esprit.tn',
             subject: "Pipeline FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
             body: "Le build ${env.JOB_NAME} #${env.BUILD_NUMBER} a échoué.\nVoir Jenkins: ${env.BUILD_URL}"
    }
}
}
