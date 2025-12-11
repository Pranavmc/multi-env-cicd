pipeline {
    agent any

    environment {
        DEV_SERVER  = "ubuntu@13.126.34.112"
        QA_SERVER   = "ubuntu@13.200.153.190"
        PROD_SERVER = "ubuntu@13.204.97.226"

        APP_BASE_DIR = "/opt/app"
        APP_NAME     = "myapp"
        VERSION      = "${env.BUILD_NUMBER}"

        DEV_PORT  = "8080"
        QA_PORT   = "8080"
        PROD_PORT = "8080"
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
                script {
                    def DEV_IP = DEV_SERVER.split("@")[1]

                    sh """
                        ARTIFACT=artifacts/${APP_NAME}-${VERSION}.jar

                        scp -o StrictHostKeyChecking=no \$ARTIFACT ${DEV_SERVER}:${APP_BASE_DIR}/releases/
                        scp -o StrictHostKeyChecking=no config/dev.env ${DEV_SERVER}:${APP_BASE_DIR}/current/.env

                        ssh -o StrictHostKeyChecking=no ${DEV_SERVER} "
                            pkill -f app.jar || true
                            ln -sfn ${APP_BASE_DIR}/releases/${APP_NAME}-${VERSION}.jar ${APP_BASE_DIR}/current/app.jar
                            nohup java -jar ${APP_BASE_DIR}/current/app.jar --server.port=${DEV_PORT} > ${APP_BASE_DIR}/current/dev.log 2>&1 &
                        "

                        sleep 5
                        curl -f http://${DEV_IP}:${DEV_PORT}/health
                    """
                }
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
                script {
                    def QA_IP = QA_SERVER.split("@")[1]

                    sh """
                        ARTIFACT=artifacts/${APP_NAME}-${VERSION}.jar

                        scp -o StrictHostKeyChecking=no \$ARTIFACT ${QA_SERVER}:${APP_BASE_DIR}/releases/
                        scp -o StrictHostKeyChecking=no config/qa.env ${QA_SERVER}:${APP_BASE_DIR}/current/.env

                        ssh -o StrictHostKeyChecking=no ${QA_SERVER} "
                            pkill -f app.jar || true
                            ln -sfn ${APP_BASE_DIR}/releases/${APP_NAME}-${VERSION}.jar ${APP_BASE_DIR}/current/app.jar
                            nohup java -jar ${APP_BASE_DIR}/current/app.jar --server.port=${QA_PORT} > ${APP_BASE_DIR}/current/qa.log 2>&1 &
                        "

                        sleep 5
                        curl -f http://${QA_IP}:${QA_PORT}/health
                    """
                }
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
                script {
                    def PROD_IP = PROD_SERVER.split("@")[1]

                    sh """
                        ARTIFACT=artifacts/${APP_NAME}-${VERSION}.jar

                        scp -o StrictHostKeyChecking=no \$ARTIFACT ${PROD_SERVER}:${APP_BASE_DIR}/releases/
                        scp -o StrictHostKeyChecking=no config/prod.env ${PROD_SERVER}:${APP_BASE_DIR}/current/.env

                        ssh -o StrictHostKeyChecking=no ${PROD_SERVER} "
                            pkill -f app.jar || true
                            ln -sfn ${APP_BASE_DIR}/releases/${APP_NAME}-${VERSION}.jar ${APP_BASE_DIR}/current/app.jar
                            nohup java -jar ${APP_BASE_DIR}/current/app.jar --server.port=${PROD_PORT} > ${APP_BASE_DIR}/current/prod.log 2>&1 &
                        "

                        sleep 5
                        curl -f http://${PROD_IP}:${PROD_PORT}/health
                    """
                }
            }
        }
    }
}
