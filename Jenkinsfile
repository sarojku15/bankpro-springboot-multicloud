pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'sarojku15/bankpro:latest'
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials-id'
    }

    tools {
        maven 'Maven 3.9.6'   // Set to the Maven version installed in Jenkins (or use 'maven' if default)
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
                // If you use the Maven wrapper, use: sh './mvnw clean package -DskipTests'
            }
        }
        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh '''
                        echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                        docker push $DOCKER_IMAGE
                        docker logout
                    '''
                }
            }
        }
        stage('Deploy via Ansible') {
            steps {
                dir('ansible') {
                    sh 'ansible-playbook -i inventory.ini deploy-docker-app.yml'
                }
            }
        }
    }
    post {
        success {
            echo '✅ Success: App built with Maven, containerized, pushed, and deployed!'
        }
        failure {
            echo '❌ Pipeline failed. Please check the error logs.'
        }
    }
}
