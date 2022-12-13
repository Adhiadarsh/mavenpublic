#! /usr/bin/env groovy

pipeline {

  agent {
    label 'maven'
  }

  stages {
    stage('Build') {
      steps {
        echo 'Building..'
        
       sh 'mvn clean package'
      }
    }
    stage('Create Container Image') {
      steps {
        echo 'Create Container Image..'
        
        script {

          openshift.withCluster() { 
  openshift.withProject("demo2") {
  
    def buildConfigExists = openshift.selector("bc", "mavenpublic").exists() 
    
    if(!buildConfigExists){ 
      openshift.newBuild("--name=mavenpublic", "--docker-image=registry.redhat.io/redhat-openjdk-18/openjdk18-openshift", "--binary") 
    } 
    
    openshift.selector("bc", "mavenpublic").startBuild("--from-file=target/demo7-1.0-SNAPSHOT.jar", "--follow") } }

        }
      }
    }
    stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {

          openshift.withCluster() { 
  openshift.withProject("demo2") { 
    def deployment = openshift.selector("dc", "mavenpublic") 
    
    if(!deployment.exists()){ 
      openshift.newApp('mavenpublic', "--as-deployment-config").narrow('svc').expose() 
    } 
    
    timeout(5) { 
      openshift.selector("dc", "mavenpublic").related('pods').untilEach(1) { 
        return (it.object().status.phase == "Running") 
      } 
    } 
  } 
}

        }
      }
    }
  }
}
