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
        // 将 SonarQube 环境变量注入（如 SONAR_HOST_URL）
        withSonarQubeEnv("${SONARQUBE_SERVER}") {
          sh """
            mvn -B sonar:sonar \
              -Dsonar.login=${SONAR_TOKEN} \
              -Dsonar.projectKey=my-project \
              -Dsonar.projectName=my-project \
              -Dsonar.branch.name=${BRANCH_NAME}
          """
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