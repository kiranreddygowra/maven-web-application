node{
def mavenhome = tool name: "maven-3.8.6"

echo "Jenkins url is: ${env.JENKINS_URL}"
echo "Node Name is: ${env.NODE_NAME}"
echo "Job name is: ${env.JOB_NAME}"


properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')), pipelineTriggers([pollSCM('* * * * * ')])])
try{
slackNotifications('STARTED')
stage('CheckOutCode'){
git branch: 'development', credentialsId: '88ac761b-eefd-4a3d-94b0-c0469dddd23f', url: 'https://github.com/kiranreddygowra/maven-web-application.git'
}

stage('Build'){
sh "$mavenhome/bin/mvn clean package"
}

stage('ExecuteSonarQubeReport'){
sh "$mavenhome/bin/mvn clean sonar:sonar"
}

stage('UploadArtifactorytoArtifactRepo'){
sh "$mavenhome/bin/mvn clean deploy"
}

stage('DeployAppIntoTomcatServer'){
sshagent(['c190d424-ea9c-48e7-9487-3d348a07a503']) {
sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@172.31.39.114:/opt/apache-tomcat-9.0.65/webapps/"
}
}

}//try block closing

catch (e)
{
slackNotifications(currentBuild.result)
throw e
}
finally{
slackNotifications(currentBuild.result)
}
} //Node closing


//Code Snippet for sending slack notifications.

def slackNotifications(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESS'
  
  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    colorName = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESS') {
    colorName = 'GREEN'
    colorCode = '#00FF00'
  } else {
    colorName = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary)
}
