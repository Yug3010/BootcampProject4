pipeline {
    agent any

    environment {
        PATH = "/usr/local/bin:${env.PATH}"
        ACR = 'yugacr3010.azurecr.io'
        IMAGE = "${ACR}/spring-petclinic:latest"
        SONAR_TOKEN = credentials('sonar-token')
    }

    tools {
        maven 'Maven_3'
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
                withSonarQubeEnv('SonarQubeServer') {
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
                sh """
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                    aquasec/trivy:latest image ${IMAGE}
                """
            }
        }

        stage('Deploy to AKS') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
                sh 'kubectl apply -f k8s/service.yaml'
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
