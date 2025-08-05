pipeline {
    agent any

    environment {
        PATH = "/usr/local/bin:${env.PATH}"
        ACR = 'yugacr3010.azurecr.io'
        IMAGE = "${ACR}/spring-petclinic:latest"
        SONAR_TOKEN = credentials('sonar-token')
    }

    tools {
        maven 'Maven_3'  // Make sure this matches your Maven tool name in Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Yug3010/BootcampProject4'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {  // Updated to match your Jenkins config name
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
