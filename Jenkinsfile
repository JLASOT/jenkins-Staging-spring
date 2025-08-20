pipeline {
    agent any
    environment {
        STAGING_SERVER = 'springuser@spring-docker'
        ARTIFACT_NAME = 'demo-0.0.1-SNAPSHOT.jar'
    }
    stages {
        stage('Clone Repository') {
            steps {
                //git 'https://github.com/JLASOT/jenkins-Staging-spring.git'
                git branch: 'main', url: 'https://github.com/JLASOT/jenkins-Staging-spring.git'
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
                echo 'Skipping code quality check'
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
                    // sh 'scp -o StrictHostKeyChecking=no target/${ARTIFACT_NAME} $STAGING_SERVER:/home/springuser/staging/'
                    // sh 'ssh -o StrictHostKeyChecking=no $STAGING_SERVER "nohup java -jar /home/springuser/staging/${ARTIFACT_NAME} > /dev/null 2>&1 &"'
                     // Copiar artefacto
                    // sh 'scp -o StrictHostKeyChecking=no target/${ARTIFACT_NAME} $STAGING_SERVER:/home/springuser/staging/'

                    // // Arrancar la app en background con logs
                    // sh '''
                    // ssh -o StrictHostKeyChecking=no $STAGING_SERVER "
                    // pkill -f '${ARTIFACT_NAME}' || true
                    // nohup java -jar /home/springuser/staging/${ARTIFACT_NAME} > /home/springuser/staging/spring.log 2>&1 &
                    // "
                    // '''
                     // Copiar el artefacto al servidor
                    sh 'scp -o StrictHostKeyChecking=no target/${ARTIFACT_NAME} $STAGING_SERVER:/home/springuser/staging/'

                    // Matar proceso anterior si existe
                    sh 'ssh -o StrictHostKeyChecking=no $STAGING_SERVER "pkill -f ${ARTIFACT_NAME} || true"'

                    // Arrancar la aplicación en background con logs
                    // sh 'ssh -o StrictHostKeyChecking=no $STAGING_SERVER "nohup java -jar /home/springuser/staging/${ARTIFACT_NAME} > /home/springuser/staging/spring.log 2>&1 &"'
                    sh 'ssh -o StrictHostKeyChecking=no $STAGING_SERVER "nohup /opt/java/openjdk/bin/java -jar /home/springuser/staging/${ARTIFACT_NAME} > /home/springuser/staging/spring.log 2>&1 &"'
                     // Matar proceso anterior y arrancar la app en background con logs
                    // sh """
                    //     ssh -o StrictHostKeyChecking=no $STAGING_SERVER '
                    //     pkill -f ${ARTIFACT_NAME} || true
                    //     nohup /opt/java/openjdk/bin/java -jar /home/springuser/staging/${ARTIFACT_NAME} > /home/springuser/staging/spring.log 2>&1 &
                    //     exit 0
                    // '
                    // """


                }
            }
        }
        stage('Validate Deployment') {
            steps {
                sh 'sleep 20'
                sh 'curl --fail http://spring-docker:8080/health'
            }
        }
    }
}
