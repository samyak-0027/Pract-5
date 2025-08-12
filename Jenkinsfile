pipeline {
    agent {
        docker {
            image 'maven:3.9.6-eclipse-temurin-17'  // Maven with Java 17
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock -v $HOME/.m2:/root/.m2'
        }
    }

    environment {
        // Good practice to name/tag your images with the build number
        DOCKER_IMAGE = 'myfirstmaven-app'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/samyak-0027/Pract-5.git',
                    branch: 'main'  // Explicitly specify branch
            }
        }

        stage('Install Docker CLI') {
            steps {
                sh '''
                    apt-get update -qq && \
                    apt-get install -y --no-install-recommends docker.io
                '''
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
                    // Tag with build number for traceability
                    def tag = "${env.DOCKER_IMAGE}:${env.BUILD_NUMBER}"
                    sh "docker build -t ${tag} ."
                    sh "docker tag ${tag} ${env.DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    // Use random available port to avoid conflicts
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
                    // Simple health check
                    sh 'docker ps --filter "status=running" | grep ${env.DOCKER_IMAGE}'
                    // You could add curl/wget to verify the endpoint
                }
            }
        }
    }

    post {
        always {
            script {
                // Cleanup all containers from this build
                sh '''
                    docker stop ${env.DOCKER_IMAGE}-container-${env.BUILD_NUMBER} || true
                    docker rm -f ${env.DOCKER_IMAGE}-container-${env.BUILD_NUMBER} || true
                    docker image prune -f  // Clean up unused images
                '''
            }
        }
        success {
            echo '✅ Pipeline completed successfully!'
            // Optionally push the image to a registry here
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
