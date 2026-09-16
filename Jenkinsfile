pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    withCredentials([string(
                        credentialsId: 'sonarqube-token',
                        variable: 'SONAR_TOKEN'
                    )]) {
                        sh '''
                            mvn sonar:sonar \
                              -Dsonar.projectKey=devops-project \
                              -Dsonar.projectName=devops-project \
                              -Dsonar.host.url=http://localhost:9000 \
                              -Dsonar.token=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-credentials',
                    usernameVariable: 'NEXUS_USERNAME',
                    passwordVariable: 'NEXUS_PASSWORD'
                )]) {
                    sh '''
                        mvn deploy \
                          -DskipTests \
                          -s /var/lib/jenkins/.m2/settings.xml
                    '''
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                deploy adapters: [
                    tomcat9(
                        credentialsId: 'tomcat-credentials',
                        path: '',
                        url: 'http://localhost:8082'
                    )
                ],
                contextPath: 'devops-project',
                war: 'target/devops-project.war'
            }
        }

        stage('Docker Deploy') {
            steps {
                sh '''
                    docker stop devops-project-container || true
                    docker rm devops-project-container || true

                    docker build -t devops-project:${BUILD_NUMBER} .

                    docker run -d \
                        --name devops-project-container \
                        --restart unless-stopped \
                        -p 8083:8080 \
                        devops-project:${BUILD_NUMBER}
                '''
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                sh '''
                    minikube image load devops-project:${BUILD_NUMBER}

                    kubectl apply -f k8s-deployment.yaml
                    kubectl apply -f k8s-service.yaml
                    kubectl apply -f k8s-ingress.yaml

                    kubectl set image deployment/devops-project \
                        devops-project=devops-project:${BUILD_NUMBER} \
                        -n dev

                    kubectl rollout status deployment/devops-project -n dev
                '''
            }
        }
    }
}
