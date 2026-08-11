pipeline {
    agent any

    environment {
        REGION = 'us-east-1'
        ACC_ID = '677938781565'
        PROJECT = 'roboshop'
        COMPONENT = 'catalogue'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    stages {

        stage('Read package.json') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    def appVersion = packageJson.version.toString()

                    if (!appVersion || appVersion == 'null') {
                        error("Invalid version in package.json")
                    }

                    echo "Package version: ${appVersion}"

                    // Store it in the current build
                    currentBuild.description = "Version: ${appVersion}"

                    // Save for later stages
                    env.IMAGE_TAG = appVersion
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Docker Build') {
            steps {
                script {

                    if (!env.IMAGE_TAG) {
                        error("IMAGE_TAG is empty. Cannot build Docker image.")
                    }

                    echo "Docker image tag: ${env.IMAGE_TAG}"

                    def imageName = "${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${PROJECT}/${COMPONENT}:${env.IMAGE_TAG}"

                    echo "Docker image: ${imageName}"

                    withAWS(
                        credentials: 'aws-crds',
                        region: "${REGION}"
                    ) {
                        sh """
                            set -e

                            aws ecr get-login-password --region ${REGION} | \
                            docker login \
                            --username AWS \
                            --password-stdin \
                            ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com

                            docker build \
                            -t ${imageName} \
                            .

                            docker push ${imageName}
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'I will always say Hello again!'
            deleteDir()
        }

        success {
            echo 'Hello Success'
        }

        failure {
            echo 'Hello Failure'
        }
    }
}
