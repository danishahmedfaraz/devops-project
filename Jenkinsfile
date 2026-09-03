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
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
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
    }
}
