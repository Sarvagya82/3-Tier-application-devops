pipeline {
    agent any

    environment {
        SONAR_HOME = tool "sonar"
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials')  // Jenkins Credential ID
    }

    stages {
        stage("Code clone from GitHub") {
            steps {
                git url: "https://github.com/Sarvagya82/3-Tier-application-devops.git", branch: "master"
            }
        }

        stage("SonarQube Quality Analysis") {
            steps {
                withSonarQubeEnv("sonar") {
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=3-tier-application -Dsonar.projectKey=3-tier-application"
                }
            }
        }

        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'OWAS_DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage("Sonar Quality Gate Scan") {
            steps {
                timeout(time: 2, unit: "MINUTES") {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage("Trivy File System Scan") {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }

        stage("Build Docker Images") {
            steps {
                sh """
                    docker build -t ${DOCKER_HUB_CREDENTIALS_USR}/3-tier-backend:latest -f backend/Dockerfile .
                    docker build -t ${DOCKER_HUB_CREDENTIALS_USR}/3-tier-frontend:latest -f frontend/Dockerfile .
                """
            }
        }

        stage("Push Docker Images to DockerHub") {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                    sh """
                        docker push ${DOCKER_HUB_CREDENTIALS_USR}/3-tier-backend:latest
                        docker push ${DOCKER_HUB_CREDENTIALS_USR}/3-tier-frontend:latest
                    """
                }
            }
        }

        stage("Deploy with Docker Compose") {
            steps {
                sh "docker-compose up -d"
                sh "sleep 30"  // Wait for containers to start
                sh "docker ps" // Check running containers
            }
        }

        stage("Stop Docker Compose after 1 minute") {
            steps {
                sh "sleep 60"  // Wait for 1 minute
                sh "docker-compose down"  // Stop all containers
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed! Check logs."
        }
    }
}
