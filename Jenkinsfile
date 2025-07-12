pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'sarojnayak1983/bankpro:latest'
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'
        DOCKER = '/usr/local/bin/docker'
        ANSIBLE_PLAYBOOK = '/opt/homebrew/bin/ansible-playbook'
        CUSTOM_PATH = '/usr/local/bin:/opt/homebrew/bin:/usr/bin:/bin'
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

        stage('Docker Buildx Build+Push (Multi-Arch)') {
            steps {
                withEnv(["PATH=${env.CUSTOM_PATH}"]) {
                    // Setup buildx builder if needed, login, build & push multi-arch image
                    withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                        sh '''
                            ${DOCKER} buildx create --use || true
                            echo "$DOCKERHUB_PASS" | ${DOCKER} login -u "$DOCKERHUB_USER" --password-stdin
                            ${DOCKER} buildx build --platform linux/amd64,linux/arm64 -t $DOCKER_IMAGE --push .
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
            echo '✅ Success: App built, containerized (multi-arch), pushed, and deployed!'
        }
        failure {
            echo '❌ Pipeline failed. Please check the error logs.'
        }
    }
}
