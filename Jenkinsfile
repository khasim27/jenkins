pipeline {
    agent any

    environment
    {
        COURSE= 'Jenkins'
    }

    stages {

        stage('Build') {
            steps {
              script{
               sh  """
               echo "Hi buuil"
               env 
               """
               

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


