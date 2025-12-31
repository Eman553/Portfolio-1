pipeline { 
    agent any 
    environment { 
        REPO_URL      = 'https://github.com/Eman553/Portfolio-1' 
        SONARQUBE_ENV = 'SonarQube-Server' 
        DOCKER_SERVER = 'ubuntu@ip-172-31-17-210'
        // Limit JVM RAM to 256MB to prevent the instance from freezing
        SONAR_SCANNER_OPTS = '-Xmx256m'
    } 
    stages { 
        stage('Checkout Code') { 
            steps { 
                git branch: 'master', 
                url: "${REPO_URL}", 
                credentialsId: 'github-credentials' 
            } 
        } 
        stage('SonarQube Analysis') { 
            steps { 
                script { 
                    def scannerHome = tool 'SonarQube Scanner' 
                    withSonarQubeEnv("${SONARQUBE_ENV}") { 
                        // Added -Dsonar.scm.disabled=true to speed up and save RAM
                        sh "${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=portfolio-cloud \
                        -Dsonar.sources=. \
                        -Dsonar.scm.disabled=true"
                    } 
                } 
            } 
        } 
        stage('Docker Build & Deploy') { 
            steps { 
                sshagent(['docker-credentials']) { 
                    sh """ 
                    scp -o StrictHostKeyChecking=no index.html Dockerfile ${DOCKER_SERVER}:/home/ubuntu/ 
                    ssh -o StrictHostKeyChecking=no ${DOCKER_SERVER} ' 
                        cd /home/ubuntu 
                        docker build -t portfolio-app . 
                        docker stop portfolio-app || true 
                        docker rm portfolio-app || true 
                        docker run -d -p 80:80 --name portfolio-app portfolio-app 
                    ' 
                    """ 
                } 
            } 
        } 
    } 
    post { 
        success { 
            echo "Deployment Successful: http://172.31.26.188" 
        } 
        failure { 
            echo "Pipeline Failed" 
        } 
    } 
}