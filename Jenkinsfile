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
                cobertura coberturaReportFile: ' **/target/site/cobertura/coverage.xml', conditionalCoverageTargets: '70, 0, 0', lineCoverageTargets: '80, 0, 0', maxNumberOfBuilds: 10, methodCoverageTargets: '80, 0, 0', packageCoverageTargets: '80, 0, 0', sourceEncoding: 'ASCII'
            }
        }
        stage('Sonarqube:- Static code Analyzer') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "mvn sonar:sonar"
                }
            }
        }
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }
        stage('ArchiveArtifact'){
            steps {
                    sh "mvn surefire-report:report"
                    archiveArtifacts 'target/*.war'   
            }
        }
        stage("Nexus Artifact Upload") {
            steps {
                 nexusArtifactUploader artifacts: [[artifactId: 'Spring3HibernateApp', classifier: '', file: 'target/Spring3HibernateApp.war', type: 'war']], credentialsId: 'nexus-credentials', groupId: 'Spring3HibernateApp', nexusUrl: 'localhost:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'repository-example', version: '6.0.0'
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
