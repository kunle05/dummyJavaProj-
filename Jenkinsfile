pipeline {
  agent any
 
  tools {
    maven 'Maven3'
  }
  stages {
    stage ('Build') {
      steps {
        sh 'mvn clean install -f MyWebApp/pom.xml'
      }
    }
    stage ('Code Quality Scan') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh 'mvn -f MyWebApp/pom.xml sonar:sonar'
        }
      }
    }
    stage ('Quality Gate') {
      steps {
        waitForQualityGate abortPipeline: true
      }
    }
    stage ('JaCoCo') {
      steps {
        jacoco()
      }
    }
    stage ('Nexus Upload') {
      steps {
        nexusArtifactUploader(
          artifacts: [
            [artifactId: 'MyWebApp', 
            classifier: '', 
            file: 'MyWebApp/target/MyWebApp.war', 
            type: 'war']
          ], 
          credentialsId: '65d7d3da-02b2-48e3-99db-6d3fd742f87c', 
          groupId: 'com.dept.app', 
          nexusUrl: 'ec2-3-145-151-9.us-east-2.compute.amazonaws.com:8081', 
          nexusVersion: 'nexus3', 
          protocol: 'http', 
          repository: 'maven-snapshots', 
          version: '1.0-SNAPSHOT'
        )
      }
    }
    stage ('Dev Deploy') {
      steps {
        echo "Deploying to dev environment"
        deploy (
          adapters: [
            tomcat9(
              credentialsId: '68386b99-d778-4fe8-8223-1e18b0f82720', 
              path: '', 
              url: 'http://ec2-13-59-147-194.us-east-2.compute.amazonaws.com:8080'
            )
          ], 
          contextPath: null, 
          war: '**/*.war'
        )
      }
    }
    stage ('Slack Notofocation') { 
      steps {
        slackSend (
          channel: 'python', 
          message: "${env.JOB_NAME} project successfully deployed to dev env"
        )
      }
    }
    stage ('Dev Approve') {
      steps {
        echo 'Waiting for Dev manager approval'
        timeout(time: 7, unit: 'DAYS') {
          input message: 'Do you approve to deploy to QA environ?'
        }
      }
    }
    
  }
}
