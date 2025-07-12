pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'sarojnayak1983/bankpro:latest'     // <-- New repo/tag
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'          // <-- Your Jenkins credentials ID
        DOCKER = '/usr/local/bin/docker'                   // <-- Path to Docker
        ANSIBLE_PLAYBOOK = '/opt/homebrew/bin/ansible-playbook' // <-- Path to ansible-playbook
        CUSTOM_PATH = '/usr/local/bin:/opt/homebrew/bin:/usr/bin:/bin' // For all shells
    }

    tools {
        maven 'MAVEN_HOME'
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
                withEnv(["PATH=${env.CUSTOM_PATH}"]) {
                    sh '${DOCKER} build -t $DOCKER_IMAGE .'
                }
            }
        }

        stage('Docker Push') {
            steps {
                withEnv(["PATH=${env.CUSTOM_PATH}"]) {
                    withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                        sh '''
                            echo "$DOCKERHUB_PASS" | ${DOCKER} login -u "$DOCKERHUB_USER" --password-stdin
                            ${DOCKER} push $DOCKER_IMAGE
                            ${DOCKER} logout
                        '''
                    }
                }
            }
        }

        stage('Deploy via Ansible') {
            steps {
                withEnv(["PATH=${env.CUSTOM_PATH}"]) {
                    dir('ansible') {
                        sh '${ANSIBLE_PLAYBOOK} -i inventory.ini deploy-docker-app.yml'
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Success: App built, containerized, pushed, and deployed!'
        }
        failure {
            echo '❌ Pipeline failed. Please check the error logs.'
        }
    }
}
