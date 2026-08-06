pipeline {
    agent {
        label 'agent-1'
    }

    stages {

        stage('Build') {
            steps {
              script{
                 echo 'Build'
              }
            }
        }

        stage('Test') {
            steps {
               script{
                 echo 'Test'
              }
            }
        }

        stage('Deploy') {
            steps {
                script{
                 echo 'Deploy'
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
            echo 'Build succeeded.'
        }
        failure {
            echo 'Build failed.'
        }
    }
}


