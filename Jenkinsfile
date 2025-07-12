pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds') // Set this in Jenkins credentials
        DOCKER_IMAGE = "<your-dockerhub-username>/bankpro-app"
    }
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build JAR') {
            steps { sh './mvnw clean package -DskipTests' }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    docker.withRegistry('', DOCKERHUB_CREDENTIALS) {
                        def app = docker.build(DOCKER_IMAGE)
                        app.push("latest")
                    }
                }
            }
        }
        stage('Deploy to Multi-Cloud') {
            steps {
                dir('ansible') {
                    sh '''
                    ansible-playbook -i inventory.ini deploy-docker-app.yml --limit aws
                    ansible-playbook -i inventory.ini deploy-docker-app.yml --limit azure
                    '''
                }
            }
        }
    }
}
