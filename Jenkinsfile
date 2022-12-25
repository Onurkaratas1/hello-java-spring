pipeline {
  agent {
    label 'maven'
  }
  environment {
    PROJECT = 'onurkaratas-crt-dev'
  }
  stages {
    stage('Preamble') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("${env.PROJECT}") {
              echo "Using project: ${openshift.project()}"
            }
          }
        }
      }
    }
    stage('Maven Build') {
      steps {
        echo 'Build jar file'
        script {
          maven {
            goals('clean')
            goals('install')
            properties skipTests: true
          }
        }
      }
    }
    stage('Run Unit Tests') {
      steps {
        echo 'Run unit tests'
        script {
          maven {
            goals('test')
          }
        }
      }
    }
    stage('Delete') {
      steps {
        echo "Delete application"
        script {
          openshift.withCluster() {
            openshift.withProject("${env.PROJECT}") {
              openshift.selector("all", [  'app' : 'springclient' ]).delete()
            }
          }
        }
      }
    }
    stage('Deploy') {
      steps {
        echo "Deploy application"
        script {
          openshift.withCluster() {
            openshift.withProject("${env.PROJECT}") {
              openshift.newApp('registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift:1.6~https://github.com/Onurkaratas1/hello-java-spring.git', "--name=springhellotest", "--strategy=source", "--allow-missing-images", "--build-env=\'JAVA_APP_JAR=hello-java-spring-boot-0.0.1-SNAPSHOT.jar\'").narrow('svc').expose()
            }
          }
        }
      }
    }
  }
}