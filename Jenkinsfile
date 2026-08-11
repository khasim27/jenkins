pipeline {
    agent any

    environment {
        REGION    = 'us-east-1'
        ACC_ID    = '677938781565'
        PROJECT   = 'roboshop'
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

                    echo "Package name: ${packageJson.name}"
                    echo "Package version: ${packageJson.version}"

                    if (!packageJson.version) {
                        error('package.json does not contain a valid version')
                    }

                    env.IMAGE_TAG = packageJson.version.toString()

                    echo "Docker Image Tag: ${env.IMAGE_TAG}"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    npm install
                '''
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    def imageName = "${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${PROJECT}/${COMPONENT}:${env.IMAGE_TAG}"

                    echo "Building Docker Image:"
                    echo "${imageName}"

                    withAWS(
                        credentials: 'aws-crds',
                        region: "${REGION}"
                    ) {
                        sh """
                            set -e

                            echo "Logging into Amazon ECR..."

                            aws ecr get-login-password --region ${REGION} | \
                            docker login \
                            --username AWS \
                            --password-stdin \
                            ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com

                            echo "Building Docker image..."

                            docker build \
                            -t ${imageName} \
                            .

                            echo "Pushing Docker image..."

                            docker push ${imageName}

                            echo "Docker image pushed successfully:"
                            echo "${imageName}"
                        """
                    }
                }
            }
        }

        stage('Cleanup Untagged ECR Images') {
            steps {
                script {
                    withAWS(
                        credentials: 'aws-crds',
                        region: "${REGION}"
                    ) {
                        sh '''
                            set -e

                            echo "Checking for untagged ECR images..."

                            UNTAGGED_IMAGES=$(aws ecr list-images \
                                --repository-name ${PROJECT}/${COMPONENT} \
                                --filter tagStatus=UNTAGGED \
                                --query 'imageIds[*].imageDigest' \
                                --output text \
                                --region ${REGION})

                            if [ -z "$UNTAGGED_IMAGES" ] || [ "$UNTAGGED_IMAGES" = "None" ]; then
                                echo "No untagged images found."
                            else
                                echo "Deleting untagged ECR images..."

                                for DIGEST in $UNTAGGED_IMAGES
                                do
                                    echo "Deleting: $DIGEST"

                                    aws ecr batch-delete-image \
                                        --repository-name ${PROJECT}/${COMPONENT} \
                                        --image-ids imageDigest=$DIGEST \
                                        --region ${REGION}
                                done

                                echo "Untagged image cleanup completed."
                            fi
                        '''
                    }
                }
            }
        }
    }

    post {

        always {
            echo 'Pipeline completed.'
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
