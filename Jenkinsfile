pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }   
    
  stage('Unit Tests - JUnit and Jacoco') {
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }
  stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "Docker_hub", url: ""]) {
          sh 'printenv'
          sh 'docker build -t Temiloladocker/numeric-app:v1 .'
          sh 'docker push Temiloladocker/numeric-app:v1'
        }
      }
    }
  }
}
  

