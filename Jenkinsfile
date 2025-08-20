pipeline {
    agent any
    environment {
        STAGING_SERVER = 'springuser@spring-docker'
        ARTIFACT_NAME = 'demo-0.0.1-SNAPSHOT.jar'
    }
    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/JLASOT/jenkins-Staging-spring.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Code Quality') {
            steps {
                //sh 'mvn checkstyle:check'
                sh 'hola world'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Code Coverage') {
            steps {
                sh 'mvn jacoco:report'
            }
        }
        stage('Deploy to Staging') {
            steps {
                sshagent(['spring-docker-key']) {
                    sh 'scp -o StrictHostKeyChecking=no target/${ARTIFACT_NAME} $STAGING_SERVER:/home/springuser/staging/'
                    sh 'ssh -o StrictHostKeyChecking=no $STAGING_SERVER "nohup java -jar /home/springuser/staging/${ARTIFACT_NAME} > /dev/null 2>&1 &"'
                }
            }
        }
        stage('Validate Deployment') {
            steps {
                sh 'sleep 10'
                sh 'curl --fail http://spring-docker:8080/health'
            }
        }
    }
}
