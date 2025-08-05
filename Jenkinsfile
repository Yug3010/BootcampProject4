pipeline {
    agent any

    environment {
        PATH = "/usr/local/bin:${env.PATH}"
        ACR = 'yugacr3010.azurecr.io'
        IMAGE = "${ACR}/spring-petclinic:latest"
        SONAR_TOKEN = credentials('sonar-token') // Make sure this ID matches Jenkins credentials
    }

    tools {
        maven 'Maven_3' // Match this with Jenkins Global Tool Configuration
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
                withSonarQubeEnv('SonarQubeServer') { // This must match your SonarQube configuration name in Jenkins
                    sh "mvn sonar:sonar -Dsonar.login=${SONAR_TOKEN}"
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'acr-creds', usernameVariable: 'ACR_USER', passwordVariable: 'ACR_PASS')]) {
                    sh """
                        docker build -t ${IMAGE} .
                        echo $ACR_PASS | docker login ${ACR} -u $ACR_USER --password-stdin
                        docker push ${IMAGE}
                    """
                }
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
