pipeline{
    agent any
    environment{
        SONAR_HOME= tool "sonar"
    }
    stages{
        stage("Code clone from GitHub"){
            steps{
                git url: "https://github.com/Sarvagya82/3-Tier-application-devops.git" , branch: "master"
            }
        }
        stage("SonarQube Quality Analysis"){
            steps{
                withSonarQubeEnv("sonar"){
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=3-tier-application -Dsonar.projectKey=3-tier-application"
                }
            }
        }
        stage("OWASP Dependecy Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./', odcInstallation:'OWAS_DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("sonar Quality Gate scan"){
            steps{
                timeout(time: 2 , unit: "MINUTES"){
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage("Trivy File System Scan"){
            steps{
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage("Deploy with Docker Compose"){
            steps{
                sh "docker-compose up -d"
            }
        }
    }
}
