@Library("shared-library") _

node {
    try {
        notifyBuild('STARTED')

        stage('SCM') {
           git 'https://github.com/opstree/spring3hibernate.git'
        }

        stage('Build') {
            sh 'mvn clean package'
        }

        stage("Code Analyze and Coverage") {
           parallel(
              Cobertura: {
                    sh "mvn cobertura:cobertura"
            },
             SonarQube: {
                    withSonarQubeEnv('SonarQube') {
                    sh "mvn sonar:sonar"
                }
        }
    )
    }
       stage('Publish Reports') {
         parallel(
            Cobertura : {
                    cobertura coberturaReportFile: ' **/target/site/cobertura/coverage.xml',
                    conditionalCoverageTargets: '70, 0, 0',
                    lineCoverageTargets: '80, 0, 0',
                    maxNumberOfBuilds: 10,
                    methodCoverageTargets: '80, 0, 0',
                    packageCoverageTargets: '80, 0, 0',
                    sourceEncoding: 'ASCII'  
      },
           Surefire : {
                sh 'mvn surefire-report:report'
                     publishHTML(
                         [
                         allowMissing: true,
                         alwaysLinkToLastBuild: false,
                         includes: '**/*.html', keepAll: true,
                         reportDir: '/var/lib/jenkins/workspace/Spring3App-CI/target',
                         reportFiles: 'index.html',
                         reportName: 'HTML Report',
                         reportTitles: ''
                         ]
            )
      }
    )
}   
       stage(' Archiving Artifacts') {
        archiveArtifacts 'target/*.war'
   }
    stage("Artiafacts Upload to Nexus") {
        nexusArtifactUploader artifacts: [
                     [artifactId: 'Spring3HibernateApp',
                      classifier: '',
                      file: 'target/Spring3HibernateApp.war',
                      type: 'war']
                 ], 
                     credentialsId: 'nexusOSS',
                     groupId: 'Spring3HibernateApp',
                     nexusUrl: 'localhost:8081',
                     nexusVersion: 'nexus3',
                     protocol: 'http',
                     repository: 'repository-example',
                     version: '9.0.0'
    }
    stage('Shared-library-demo') {
            helloWorldSimple("Mikul","Sunday")
    }
}catch (e) {
    // If there was an exception thrown, the build failed
    currentBuild.result = "FAILED"
    throw e
  } finally {
    // Success or failure, always send notifications
    notifyBuild(currentBuild.result)
  }
}

def notifyBuild(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESSFUL'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESSFUL') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary)
}
