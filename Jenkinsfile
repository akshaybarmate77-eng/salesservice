pipeline {
    agent any

    environment {
        COMPOSE_FILE = "docker-compose.sellerservice.yml"
        REGISTRY_CONTAINER_NAME = "serviceregistry"
        TARGET_SERVICE = "sellerservice"
        TARGET_CONTAINER_NAME = "sellerservice"
        TARGET_IMAGE_NAME = "sellerservice:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Version') {
            steps {
                sh '''
                    set -e
                    docker --version
                    docker compose version
                '''
            }
        }

        stage('Validate Configuration') {
            steps {
                sh '''
                    set -e

                    if [ ! -f "$COMPOSE_FILE" ]; then
                        echo "ERROR: Compose file not found: $COMPOSE_FILE"
                        exit 1
                    fi

                    docker compose -f "$COMPOSE_FILE" config -q
                    echo "Docker Compose configuration is valid."
                '''
            }
        }

        stage('Check Registry and Run Service') {
            steps {
                script {
                    def isRegistryRunning = sh(
                        script: "docker ps -q -f name=^/${REGISTRY_CONTAINER_NAME}$",
                        returnStdout: true
                    ).trim()

                    if (isRegistryRunning) {
                        echo "${REGISTRY_CONTAINER_NAME} is running. Deploying ${TARGET_SERVICE}..."

                        // Build the service image
                        sh "docker compose -f ${COMPOSE_FILE} build ${TARGET_SERVICE}"

                        // Create or recreate the service container
                        sh "docker compose -f ${COMPOSE_FILE} up -d --no-deps --force-recreate ${TARGET_SERVICE}"

                        // Verify the service container is running
                        sh '''
                            set -e
                            docker ps --filter "name=^/sellerservice$" \
                                --filter "status=running" \
                                --format '{{.Names}}' | grep -Fx sellerservice
                        '''

                        echo "${TARGET_SERVICE} deployment completed successfully."
                    } else {
                        error "${REGISTRY_CONTAINER_NAME} is not running. Deployment aborted."
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully.'
        }

        failure {
            echo 'Deployment failed. Check the preceding Jenkins logs.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
