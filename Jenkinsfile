pipeline {
    agent any

    environment {
        // Add BOTH Docker and Maven paths
        CONTAINER_NAME = "myfirstmaven-container"
        PATH = "/opt/homebrew/bin:/usr/local/bin:${env.PATH}"
        DOCKER_IMAGE = 'myfirstmaven-app'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/samyak-0027/Pract-5.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn -v'
                sh 'mvn clean package'
                archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: false
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def tag = "${env.DOCKER_IMAGE}:${env.BUILD_NUMBER}"
                    sh "docker build -t ${tag} ."
                    sh "docker tag ${tag} ${env.DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    sh "docker rm -f $`{CONTAINER_NAME}` || true"
                    sh "docker run -d --name $`{CONTAINER_NAME}` -p 9090:8080 myfirstmaven-app:latest"
                    sh "docker ps"
                }
            }
        }


        stage('Verify Container') {
            steps {
                sh 'docker ps --filter "status=running" | grep ${DOCKER_IMAGE}'
            }
        }
    }

    post {
        always {
            sh '''
                docker stop ${DOCKER_IMAGE}-container-${BUILD_NUMBER} || true
                docker rm -f ${DOCKER_IMAGE}-container-${BUILD_NUMBER} || true
                docker image prune -f
            '''
        }
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
