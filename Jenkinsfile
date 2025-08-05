pipeline {
    agent any

    environment {
        ACR = 'yugacr3010.azurecr.io'
        IMAGE = "${ACR}/spring-petclinic:latest"
        SONAR_TOKEN = credentials('sonar-token')
    }

    tools {
        maven 'Maven_3'  // This must match your Maven tool name in Jenkins Global Tool Configuration
    }

    stages {
        // No explicit checkout needed if Jenkins pipeline SCM checkout is enabled

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube') {
            steps {
                withSonarQubeEnv('MySonarQube') {
                    sh "mvn sonar:sonar -Dsonar.login=${SONAR_TOKEN}"
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh """
                   docker build -t ${IMAGE} .
                   az acr login --name yugacr3010
                   docker push ${IMAGE}
                   """
            }
        }

        stage('Trivy Scan') {
            steps {
                sh "trivy image ${IMAGE}"
            }
        }

        stage('Deploy to AKS') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
                sh 'kubectl apply -f k8s/service.yaml'
            }
        }
    }
}
