pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-cred'
        DOCKER_IMAGE          = '8089278330/fast-api-chatbot'
        SONAR_SERVER_NAME     = 'sonarqube_env' 
        SONAR_SCANNER_TOOL    = 'sonar-scanner'     
        GIT_CREDENTIALS_ID    = 'github-cred'
        MANIFEST_REPO         = 'https://github.com/suhaaail2004/fast-api-chatbot.git'
        MANIFEST_BRANCH       = 'main'
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                echo "passed"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool "${SONAR_SCANNER_TOOL}"
                    withSonarQubeEnv("${SONAR_SERVER_NAME}") {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=fast-api-chatbot-key \
                            -Dsonar.projectName='fast-api-chatbot-project' \
                            -Dsonar.sources=. \
                            -Dsonar.exclusions=*/venv/,/.venv/,/_pycache_/,/.pytest_cache/,/static/*"
                    }
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin
                            
                            docker build -t ${DOCKER_IMAGE}:${env.BUILD_NUMBER} -t${DOCKER_IMAGE}:latest .
                            
                            docker push ${DOCKER_IMAGE}:${env.BUILD_NUMBER}
                            docker push ${DOCKER_IMAGE}:latest
                        """
                    }
                }
            }
        }

        stage('Update Manifest Repo') {
            steps {
                dir('manifest-workspace') {
                    sh "git clone ${MANIFEST_REPO} . || true"
                    withCredentials([usernamePassword(credentialsId: "${GIT_CREDENTIALS_ID}", usernameVariable: 'GH_USER', passwordVariable: 'GH_TOKEN')]) {
                        sh """
                            git remote set-url origin https://\${GH_USER}:\${GH_TOKEN}@github.com/suhaaail2004/fast-api-chatbot.git
                            git pull --rebase origin ${MANIFEST_BRANCH}
                            
                            sed -i '/name: fast-api-chatbot/{n;s|image:.*|image: ${DOCKER_IMAGE}:${env.BUILD_NUMBER}|}' k8s/deployment.yaml
                            
                            git config user.name "suhaaail2004"
                            git config user.email "suhailshihab444@gmail.com"
                            git commit -am "chore(manifest): update fast-api-chatbot image tag to ${env.BUILD_NUMBER} [skip ci]"
                            git push origin ${MANIFEST_BRANCH}
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline executed successfully! fast-api-chatbot image built, pushed, and manifest repo updated.'
        }
        failure {
            echo 'Pipeline failed. Please review the stage logs above.'
        }
    }
}
