#! /usr/bin/env groovy

pipeline {

  agent {
    label 'maven'
  }

  stages {
    stage('Build') {
      steps {
        echo 'Building..'

        // Add steps here
        sh 'mvn clean package'
      }
    }
    stage('Create Container Image') {
      steps {
        echo 'Create Container Image..'

        script {

          // Add steps here
          openshift.withCluster() {
            openshift.withProject("onurkaratas-crt-dev") {

              def buildConfigExists = openshift.selector("bc", "springhellotest").exists()

              if(!buildConfigExists){
                openshift.newBuild("--name=springhellotest", "--docker-image=registry.redhat.io/jboss-eap-7/eap74-openjdk8-openshift-rhel7", "--binary")
              }

              openshift.selector("bc", "springhellotest").startBuild("--from-file=target/hello-java-spring-boot-0.0.1-SNAPSHOT.jar", "--follow") } }

        }
      }
    }
    stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {

          // Add steps here
          openshift.withCluster() {
            openshift.withProject("onurkaratas-crt-dev") {
              def deployment = openshift.selector("dc", "springhellotest")

              if(!deployment.exists()){
                openshift.newApp('springhellotest', "--as-deployment-config").narrow('svc').expose()
              }

              timeout(5) {
                openshift.selector("dc", "springhellotest").related('pods').untilEach(1) {
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