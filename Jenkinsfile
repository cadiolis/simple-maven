pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        parallel(
          "Build": {
            echo 'Build 1'
            echo 'Build 2'
            
          },
          "Build 2": {
            echo 'Build 2a'
            echo 'Build 2b'
            
          }
        )
      }
    }
    stage('Test') {
      steps {
        parallel(
          "Test": {
            echo 'test1a'
            echo 'test1b'
            
          },
          "Test2": {
            echo 'test2a'
            echo 'test2b'
            
          },
          "Test3": {
            echo 'test3a'
            echo 'test3b'
            
          }
        )
      }
    }
    stage('Release') {
      steps {
        echo 'Release it'
      }
    }
  }
}