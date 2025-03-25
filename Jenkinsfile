import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles) {
    if (p['publishMethod'] == 'FTP') {
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
    }
  }
  return null
}

node {
  withEnv([
    'AZURE_SUBSCRIPTION_ID=58935b87-b198-477c-be29-c88989b1d9a0',
    'AZURE_TENANT_ID=0a298482-3c81-4841-82f4-c30a790cc0f5'
  ]) {
    stage('init') {
      checkout scm
    }

    stage('build') {
      sh 'mvn clean package'
    }

    stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName = 'myjenkinswebappkexu'

      // Azure login
      withCredentials([usernamePassword(
        credentialsId: 'AzureServicePrincipal',
        passwordVariable: 'AZURE_CLIENT_SECRET',
        usernameVariable: 'AZURE_CLIENT_ID'
      )]) {
        sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }

      // Get publish profile JSON
      def pubProfilesJson = sh(
        script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName",
        returnStdout: true
      ).trim()

      def ftpProfile = getFtpPublishProfile(pubProfilesJson)

      // Extract base URL for WAR deploy
      def deployUrl = ftpProfile.url.replaceFirst(/:.*$/, '').replace('ftp://', 'https://') + '/api/wardeploy'

      // Upload WAR using WAR Deploy API
      sh """
        curl -X POST \\
             -u '${ftpProfile.username}:${ftpProfile.password}' \\
             --data-binary @target/calculator-1.0.war \\
             '${deployUrl}'
      """

      // Azure logout
      sh 'az logout'
    }
  }
}
