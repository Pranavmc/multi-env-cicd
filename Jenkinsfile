pipeline {
    agent any

    environment {
        DEV_SERVER  = "ubuntu@13.205.29.147"
        QA_SERVER   = "ubuntu@13.235.220.79"
        PROD_SERVER = "ubuntu@3.6.97.178"

        APP_BASE_DIR = "/opt/app"
        APP_NAME     = "myapp"
        VERSION      = "${env.BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Pranavmc/multi-env-cicd.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Package Artifact') {
            steps {
                sh """
                    mkdir -p artifacts
                    cp target/*.jar artifacts/${APP_NAME}-${VERSION}.jar
                """
                archiveArtifacts artifacts: 'artifacts/*.jar', fingerprint: true
            }
        }

        stage('Deploy to Dev') {
            steps {
                sh """
                    ARTIFACT=artifacts/${APP_NAME}-${VERSION}.jar

                    scp -o StrictHostKeyChecking=no \$ARTIFACT ${DEV_SERVER}:${APP_BASE_DIR}/releases/
                    scp -o StrictHostKeyChecking=no config/dev.env ${DEV_SERVER}:${APP_BASE_DIR}/current/.env

                    ssh -o StrictHostKeyChecking=no ${DEV_SERVER} "
                        pkill -f app.jar || true
                        ln -sfn ${APP_BASE_DIR}/releases/${APP_NAME}-${VERSION}.jar ${APP_BASE_DIR}/current/app.jar
                        nohup java -jar ${APP_BASE_DIR}/current/app.jar > ${APP_BASE_DIR}/current/dev.log 2>&1 &
                    "
                """
            }
        }

        stage('Approve QA Deployment') {
            steps {
                script {
                    input message: "Proceed to QA?", ok: "Yes"
                }
            }
        }

        stage('Deploy to QA') {
            steps {
                sh """
                    ARTIFACT=artifacts/${APP_NAME}-${VERSION}.jar

                    scp -o StrictHostKeyChecking=no \$ARTIFACT ${QA_SERVER}:${APP_BASE_DIR}/releases/
                    scp -o StrictHostKeyChecking=no config/qa.env ${QA_SERVER}:${APP_BASE_DIR}/current/.env

                    ssh -o StrictHostKeyChecking=no ${QA_SERVER} "
                        pkill -f app.jar || true
                        ln -sfn ${APP_BASE_DIR}/releases/${APP_NAME}-${VERSION}.jar ${APP_BASE_DIR}/current/app.jar
                        nohup java -jar ${APP_BASE_DIR}/current/app.jar > ${APP_BASE_DIR}/current/qa.log 2>&1 &
                    "
                """
            }
        }

        stage('Approve Prod Deployment') {
            steps {
                script {
                    input message: "Proceed to PROD?", ok: "Yes"
                }
            }
        }

        stage('Deploy to Prod') {
            steps {
                sh """
                    ARTIFACT=artifacts/${APP_NAME}-${VERSION}.jar

                    scp -o StrictHostKeyChecking=no \$ARTIFACT ${PROD_SERVER}:${APP_BASE_DIR}/releases/
                    scp -o StrictHostKeyChecking=no config/prod.env ${PROD_SERVER}:${APP_BASE_DIR}/current/.env

                    ssh -o StrictHostKeyChecking=no ${PROD_SERVER} "
                        pkill -f app.jar || true
                        ln -sfn ${APP_BASE_DIR}/releases/${APP_NAME}-${VERSION}.jar ${APP_BASE_DIR}/current/app.jar
                        nohup java -jar ${APP_BASE_DIR}/current/app.jar > ${APP_BASE_DIR}/current/prod.log 2>&1 &
                    "
                """
            }
        }
    }
}
