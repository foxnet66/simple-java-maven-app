pipeline {
  agent any

  stages {
    stage('Env') {
      steps {
        sh 'echo "PATH=$PATH"'
        sh 'which mvn || true'
        sh 'mvn -v || true'
      }
    }

    stage('Build') {
      steps {
        withEnv(["PATH=/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"]) {
          sh 'mvn -v'
          sh 'mvn -B -DskipTests clean package'
        }
      }
    }
  }
}