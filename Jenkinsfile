pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node23'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'  // Ensure proper spacing around the '='
    }
    stages {
        stage("Clean Workspace") {
            steps {
                cleanWs()
            }
        }
        stage("Git Checkout") {
            steps {
                git 'https://github.com/Rohinigundala2019/Zomato-clone-App-deployment.git'
            }
        }
        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' 
                    $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=zomato \
                    -Dsonar.projectKey=zomato 
                    '''  // Multiline shell script for better readability
                }
            }
        }
        stage("Code Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage("Install NPM Dependencies") {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit -n', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }  // Missing closing bracket for the 'OWASP FS SCAN' stage
        }
        stage("Trivy File Scan") {
            steps {
                sh "trivy fs . > trivy.txt"
            }
        }
        stage("Build Docker Image") {
            steps {
                sh "docker build -t zomato ."
            }
        }
        stage("Tag & Push to DockerHub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker') {
                        sh "docker tag zomato rohinigundala2024/zomato:latest "
                        sh "docker push rohinigundala2024/zomato:latest "
                    }
                }
            }
        }
        stage('Docker Scout Image') {
            steps {
                script {
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                       sh 'docker-scout quickview rohinigundala2024/zomato:latest'
                       sh 'docker-scout cves rohinigundala2024/zomato:latest'
                       sh 'docker-scout recommendations rohinigundala2024/zomato:latest'
                   }
                }
            }
        }
        stage("Deploy to Container") {
            steps {
                sh 'docker run -d --name zomato -p 3000:3000 rohinigundala2024/zomato:latest'
            }
        }
    }
}
