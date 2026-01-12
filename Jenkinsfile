stage('Build') {
  steps {
    withEnv(["PATH=/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"]) {
      sh 'mvn -v'
      sh 'mvn -B -DskipTests clean package'
    }
  }
}