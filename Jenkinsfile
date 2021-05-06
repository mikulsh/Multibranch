pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                git 'https://github.com/opstree/spring3hibernate.git'
                sh "mvn clean"
                sh "mvn compile"
            }
        }
        stage('Test') {
            steps {
                sh "mvn test"
                sh "mvn package"
            }
        }
        stage('Cobertura Report') {
            steps {
                sh "mvn cobertura:cobertura"
            }
        }
        stage('Sonarqube:- Static code Analyzer') {
            steps {
                withSonarQubeEnv('sonarQube') {
                    sh "mvn sonar:sonar"
                }
            }
        }
        stage('ArchiveArtifact'){
            steps {
                    sh "mvn surefire-report:report"
                    archiveArtifacts 'target/*.war'   
            }
        }
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }
        stage("Nexus Artifact Upload") {
            steps {
                 nexusArtifactUploader artifacts: [[artifactId: 'Spring3HibernateApp', classifier: '', file: 'target/Spring3HibernateApp.war', type: 'war']], credentialsId: 'nexus-credentials', groupId: 'Spring3HibernateApp', nexusUrl: 'localhost:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'repository-example', version: '4.0.0'
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
        always{
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, includes: '**/*.html', keepAll: false, reportDir: '/var/lib/jenkins/workspace/Final/target/site', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: ''])
        }
    }
}
def notifyFailed() {
    slackSend (color: '#00FF00', message: "Failed: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
}
def notifySuccessful() {
    slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
}
