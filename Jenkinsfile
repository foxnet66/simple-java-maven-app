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
    SONAR_TOKEN = credentials('sonar-token')   // ✅ 从 Jenkins Credentials 取
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        sh 'java -version'
        sh 'mvn -v'
        sh 'mvn -B -DskipTests clean package'
      }
      post {
        success {
          archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
      }
    }

    stage('Test') {
      when {
        expression { fileExists('src/test/java') }
      }
      steps {
        sh 'mvn -B test'
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
        }
      }
    }

    stage('SAST (SonarQube)') {
      steps {
        withSonarQubeEnv(env.SONARQUBE_SERVER) {   // ✅ 用 env.
sh '''
  mvn -B org.sonarsource.scanner.maven:sonar-maven-plugin:3.11.0.3922:sonar \
    -Dsonar.login=$SONAR_TOKEN \
    -Dsonar.projectKey=my-project \
    -Dsonar.projectName=my-project
'''
        }
      }
    }

    // （可选但很建议）加质量门禁
    stage('Quality Gate') {
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
    always {
      cleanWs()
    }
  }
}