import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles) {
    if (p['publishMethod'] == 'FTP') {
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
    }
  }
}

node {
  withEnv([
    'AZURE_SUBSCRIPTION_ID=85b99612-f850-4ddb-85d8-b58aaddd6d0e',
    'AZURE_TENANT_ID=24a70762-5b21-480a-8ce6-7b0ae258d399'
  ]) {
    stage('init') {
      checkout scm
    }

    stage('build') {
      sh 'mvn clean package'
    }

    stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName = 'yuhangou'

      withCredentials([
        usernamePassword(
          credentialsId: 'AzureServicePrincipal',
          usernameVariable: 'AZURE_CLIENT_ID',
          passwordVariable: 'AZURE_CLIENT_SECRET'
        )
      ]) {
        sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }

      def pubProfilesJson = sh(
        script: "az webapp deployment list-publishing-profiles -g ${resourceGroup} -n ${webAppName}",
        returnStdout: true
      )
      def ftpProfile = getFtpPublishProfile(pubProfilesJson)

      sh "az webapp deploy --resource-group ${resourceGroup} --name ${webAppName} --src-path target/calculator-1.0.war --type war"
      sh 'az logout'
    }
  }
}
