pipeline {
    agent any

    environment {
        ACR = 'yugacr3010.azurecr.io'
        IMAGE = "${ACR}/spring-petclinic:latest"
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {
        stage('Checkout') {
            steps {
               git branch: 'main', url: 'https://github.com/spring-projects/spring-petclinic.git'

            }
        }

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
                sh 'docker build -t $IMAGE .'
                sh 'az acr login --name myacrname123'
                sh 'docker push $IMAGE'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy image $IMAGE'
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
