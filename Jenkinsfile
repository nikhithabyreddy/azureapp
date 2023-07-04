import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

pipeline {
  agent any
  
  environment {
    AZURE_SUBSCRIPTION_ID = 'f36e20ed-7535-418f-b7e3-3c26c6d85da9'
    AZURE_TENANT_ID = '15fae2f6-4d4b-4382-a2a5-89478e38a20e'
  }
  
  stages {
    stage('init') {
      steps {
        checkout scm
      }
    }
    
    stage('build') {
      steps {
        sh 'mvn clean package'
      }
    }
    
    stage('deploy') {
      steps {
        script {
          def resourceGroup = 'firstwebapp2899_group'
          def webAppName = 'nikhithaapp'
          
          // login Azure
          withCredentials([usernamePassword(credentialsId: 'awsjenkinsserver', passwordVariable: 'tjB8Q~aTPpzfuKTzRAKn~jCzTX.ZKzai54PKHdCo', usernameVariable: '0b2725bf-519c-44c1-978b-fe4e6178a623')]) {
            sh '''
              az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
              az account set -s $AZURE_SUBSCRIPTION_ID
            '''
          }
          
          // get publish settings
          def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
          def ftpProfile = getFtpPublishProfile(pubProfilesJson)
          
          // upload package
          sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '${ftpProfile.username}:${ftpProfile.password}'"
          
          // log out
          sh 'az logout'
        }
      }
    }
  }
}
