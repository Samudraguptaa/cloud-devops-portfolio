
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
        disableConcurrentBuilds()
        timeout(time: 20, unit: 'MINUTES')
    }

    triggers {
        pollSCM('H/2 * * * *')
    }

    environment {
        IMAGE_NAME = 'cloud-devops-portfolio:local'
        COMPOSE_PROJECT = 'cloud-devops-portfolio'
        APP_URL = 'http://127.0.0.1:3000'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
                sh 'git rev-parse --short HEAD'
            }
        }

        stage('Validate') {
            steps {
                echo 'Validating Docker configuration...'
                sh '''
                    docker version
                    docker compose version
                    docker compose -p "$COMPOSE_PROJECT" \
                        -f compose.yaml config --quiet
                '''
            }
        }

        stage('Build') {
            steps {
                echo 'Building production Docker image...'
                sh '''
                    docker build \
                        -t "$IMAGE_NAME" \
                        .
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh '''
                    docker compose \
                        -p "$COMPOSE_PROJECT" \
                        -f compose.yaml \
                        up -d --force-recreate --remove-orphans
                '''
            }
        }

        stage('Verify') {
            steps {
                echo 'Checking application health...'
                sh '''
                    for attempt in $(seq 1 30); do
                        if curl -fsS "$APP_URL" > /dev/null; then
                            echo "Application is responding successfully!"
                            exit 0
                        fi

                        echo "Waiting for application... attempt $attempt"
                        sleep 2
                    done

                    echo "Application health check failed!"
                    docker compose -p "$COMPOSE_PROJECT" ps
                    docker compose -p "$COMPOSE_PROJECT" logs --tail=100 portfolio
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD PIPELINE SUCCESSFUL!'
            echo 'Application: http://localhost:3000'
        }

        failure {
            echo 'CI/CD PIPELINE FAILED!'
            echo 'Check the console output for details.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
