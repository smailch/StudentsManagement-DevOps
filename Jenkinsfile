pipeline {
  agent any

  tools {
    jdk 'JAVA_HOME'
    maven 'Maven3'
  }

  triggers { pollSCM('H/2 * * * *') }

  environment {
    SONAR_PROJECT_KEY  = 'StudentsManagement-DevOps'
    SONAR_PROJECT_NAME = 'StudentsManagement-DevOps'

    DOCKER_IMAGE = 'chemlali/studentsmgmt'
    DOCKER_TAG_LATEST = 'latest'

    DOCKERHUB_CREDENTIALS_ID = 'dockerhub-credentials'
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'master',
            url: 'https://github.com/smailch/StudentsManagement-DevOps.git'
      }
    }

    stage('Maven Build') {
      steps {
        sh 'mvn -B clean package -DskipTests'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('sonarqube') {
          sh """
            mvn -B sonar:sonar \
              -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
              -Dsonar.projectName=${SONAR_PROJECT_NAME}
          """
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }

    stage('Docker Build') {
      steps {
        sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG_LATEST} ."
        sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG_LATEST} ${DOCKER_IMAGE}:${BUILD_NUMBER}"
      }
    }

    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: "${DOCKERHUB_CREDENTIALS_ID}",
          usernameVariable: 'USER',
          passwordVariable: 'PASS'
        )]) {
          sh 'echo "$PASS" | docker login -u "$USER" --password-stdin'
          sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG_LATEST}"
          sh "docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}"
        }
      }
    }
  }

  post {
    success { echo 'Pipeline CI/CD complète terminée – Maven/Sonar/Docker OK' }
    failure { echo 'Échec de la pipeline' }
  }
}
