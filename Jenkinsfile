pipeline {
    agent any

    environment {
        APP_NAME = "springboot-app"
        DOCKER_IMAGE = "sanskar0609/springboot-app"
        DOCKER_CREDENTIALS = "dockerhub-creds"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/your-username/your-repo.git'
            }
        }

        stage('Set Build Version') {
            steps {
                script {
                    COMMIT_ID = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()

                    BUILD_TAG = "v${env.BUILD_NUMBER}-${COMMIT_ID}"
                    env.DOCKER_TAG = BUILD_TAG
                }
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean test'
            }
        }

        stage('Package JAR') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t $DOCKER_IMAGE:$DOCKER_TAG .
                docker tag $DOCKER_IMAGE:$DOCKER_TAG $DOCKER_IMAGE:latest
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: DOCKER_CREDENTIALS,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKER_IMAGE:$DOCKER_TAG
                    docker push $DOCKER_IMAGE:latest
                    """
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh """
                docker stop $APP_NAME || true
                docker rm $APP_NAME || true
                docker run -d \
                  --name $APP_NAME \
                  -p 8080:8080 \
                  --restart unless-stopped \
                  $DOCKER_IMAGE:$DOCKER_TAG
                """
            }
        }

        stage('Health Check') {
            steps {
                sh """
                sleep 15
                curl -f http://localhost:8080/actuator/health
                """
            }
        }
    }

    post {
        success {
            echo "Deployment Successful"
        }

        failure {
            echo "Deployment Failed — Rolling back"
            sh """
            docker stop $APP_NAME || true
            docker rm $APP_NAME || true
            docker run -d \
              --name $APP_NAME \
              -p 8080:8080 \
              $DOCKER_IMAGE:latest
            """
        }

        always {
            sh "docker image prune -f"
        }
    }
}
