pipeline {
    agent any

    environment {
        APP_VERSION =''
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
stage('Check package.json') {
    steps {
        sh '''
            echo "===== CURRENT DIRECTORY ====="
            pwd

            echo "===== FILES ====="
            ls -la

            echo "===== package.json ====="
            cat package.json

            echo "===== VERSION USING NODE ====="
            node -p "require('./package.json').version"
        '''

        script {
            def packageJson = readJSON file: 'package.json'

            echo "DEBUG packageJson: ${packageJson}"
            echo "DEBUG version: ${packageJson.version}"

            if (!packageJson.version) {
                error("package.json does not contain a valid version")
            }

            env.APP_VERSION = packageJson.version.toString()

            echo "Package version: ${env.APP_VERSION}"
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
                    withAWS(
                        credentials: 'aws-crds',
                        region: "${REGION}"
                    ) {
                        sh """
                            aws ecr get-login-password --region ${REGION} | \
                            docker login --username AWS --password-stdin \
                            ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com

                            docker build \
                            -t ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${PROJECT}/${COMPONENT}:${env.APP_VERSION} \
                            .

                            docker push \
                            ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${PROJECT}/${COMPONENT}:${env.APP_VERSION}
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
