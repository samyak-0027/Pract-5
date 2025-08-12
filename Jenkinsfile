pipeline {
    agent any  // Run directly on your Mac Jenkins node

    environment {
        DOCKER_IMAGE = 'myfirstmaven-app'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/samyak-0027/Pract-5.git'
            }
        }

        stage('Build with Maven') {
            steps {
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
                    def port = sh(script: "shuf -i 8000-9000 -n 1", returnStdout: true).trim()
                    sh """
                        docker run -d \
                          --name ${env.DOCKER_IMAGE}-container-${env.BUILD_NUMBER} \
                          -p ${port}:8080 \
                          ${env.DOCKER_IMAGE}:${env.BUILD_NUMBER}
                    """
                    echo "Application running on port ${port}"
                }
            }
        }

        stage('Verify Container') {
            steps {
                script {
                    sh 'docker ps --filter "status=running" | grep ${DOCKER_IMAGE}'
                    // Optional: Add `curl http://localhost:<port>` to check health
                }
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