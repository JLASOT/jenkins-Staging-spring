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
        // Mata el proceso anterior si está corriendo la app para evitar conflictos
        stage('Prepare Staging Server') {
            steps {
                sshagent(['spring-docker-key']) {
                    script {
                        def status = sh(
                            script: """
                                ssh -o StrictHostKeyChecking=no $STAGING_SERVER '
                                pid=\$(pgrep -f ${ARTIFACT_NAME} || true)
                                if [ -n "\$pid" ]; then
                                    echo "Matando proceso \$pid"
                                    kill -9 \$pid || true
                                else
                                    echo "No hay procesos corriendo"
                                fi
                                '
                            """,
                            returnStatus: true
                        )
                        echo "Código de salida del stage: ${status}"
                    }
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                sshagent(['spring-docker-key']) {
   
                     // Copiar el artefacto al servidor
                    sh 'scp -o StrictHostKeyChecking=no target/${ARTIFACT_NAME} $STAGING_SERVER:/home/springuser/staging/'

                     // ejecutar la aplicación con Java en segundo plano
                    sh 'ssh -o StrictHostKeyChecking=no $STAGING_SERVER "nohup /opt/java/openjdk/bin/java -jar /home/springuser/staging/${ARTIFACT_NAME} > /home/springuser/staging/spring.log 2>&1 &"'

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
    post {
        always {
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
    }
}
