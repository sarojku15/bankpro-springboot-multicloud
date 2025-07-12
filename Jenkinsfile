pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'sarojku15/bankpro:latest'
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'
    }

    tools {
        maven 'MAVEN_HOME' // This should now match your Jenkins tool name!
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
