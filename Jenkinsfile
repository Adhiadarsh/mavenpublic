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
  openshift.withProject("cicdjenkins") {
  
    def buildConfigExists = openshift.selector("bc", "otherbuild").exists() 
    
    if(!buildConfigExists){ 
      openshift.newBuild("--name=otherbuild", "--docker-image=registry.redhat.io/redhat-openjdk-18/openjdk18-openshift", "--binary") 
    } 
    
    openshift.selector("bc", "otherbuild").startBuild("--from-file=target/demo7-1.0-SNAPSHOT.jar", "--follow") } }

        }
      }
    }
    stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {

          openshift.withCluster() { 
  openshift.withProject("cicdjenkins") { 
    def deployment = openshift.selector("dc", "otherbuild") 
    
    if(!deployment.exists()){ 
      openshift.newApp('otherbuild', "--as-deployment-config").narrow('svc').expose() 
    } 
    
    timeout(5) { 
      openshift.selector("dc", "otherbuild").related('pods').untilEach(1) { 
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
