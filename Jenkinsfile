pipeline {
  agent any

  options {
    timestamps()
    disableConcurrentBuilds()
    skipDefaultCheckout(false)
  }

  tools {
    jdk 'jdk-17'
    maven 'maven-3'
  }

  environment {
    MAVEN_OPTS = '-Dmaven.repo.local=.m2/repository'
    SONARQUBE_SERVER = 'sonar-server'
    SONAR_TOKEN = credentials('sonar-token')
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build & Test') {
      steps {
        sh 'echo "JAVA_HOME=$JAVA_HOME"'
        sh 'which java && java -version'
        sh 'mvn -v'
        sh 'mvn -B clean verify'
      }
      post {
        always { junit 'target/surefire-reports/*.xml' }
        success { archiveArtifacts artifacts: 'target/*.jar', fingerprint: true }
      }
    }

    stage('SAST (SonarQube)') {
      steps {
        withSonarQubeEnv(env.SONARQUBE_SERVER) {
          sh '''
            mvn -B org.sonarsource.scanner.maven:sonar-maven-plugin:3.11.0.3922:sonar \
              -Dsonar.login=$SONAR_TOKEN \
              -Dsonar.projectKey=my-project \
              -Dsonar.projectName=my-project
          '''
        }
      }
    }

    stage('Quality Gate') {
      when { anyOf { branch 'master'; branch 'main' } }
      steps {
        timeout(time: 10, unit: 'MINUTES') {
          script {
            def qg = waitForQualityGate()
            if (qg.status != 'OK') {
              error "Quality Gate failed: ${qg.status}"
            }
          }
        }
      }
    }
  }

  post {
    always { cleanWs() }
  }
}