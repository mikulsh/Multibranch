pipeline {
    agent any

    stages {
        stage('Artifacts Downoad') {
            steps {
              sh 'wget --user=admin --password=1234 http://localhost:8081/repository/repository-example/Spring3HibernateApp/Spring3HibernateApp/4.0.0/Spring3HibernateApp-4.0.0.war'
              sh 'mv Spring3HibernateApp-4.0.0.war Spring3Appp.war'
            }
        }
        stage('Copying Artifacts to server') {
            steps {
                sh '/var/lib/jenkins/deployScript.sh'
            }
        }
    }
    post {
        success {
            notifySuccessful()
        }
        failure{
            notifyFailed()
        }
    }
}

def notifyFailed() {
    slackSend (color: '#00FF00', message: "FAILED TO DEPLOY ARTIFACTS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
}
def notifySuccessful() {
    slackSend (color: '#00FF00', message: "SUCCESSFULLY DEPLOYED ONTO SERVER: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
}
