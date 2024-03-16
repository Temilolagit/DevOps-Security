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
   stage('Mutation Tests - PIT') {
      steps {
        sh "mvn org.pitest:pitest-maven:mutationCoverage"
      }
      post {
        always {
          pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
        }
      }
    }
  stage('SonarQube - SAST') {
      steps {
        withSonarQubeEnv('sonar-server') {
          sh "mvn sonar:sonar \
		              -Dsonar.projectKey=Numeric \
		              -Dsonar.host.url=http://34.197.199.7:9000"
        }
        timeout(time: 2, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true
          }
        }
      }
    }
  stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "Docker_hub", url: ""]) {
          sh 'printenv'
          sh 'docker build -t temiloladocker/numeric-app:v1 .'
          sh 'docker push temiloladocker/numeric-app:v1'
        }
      }
    }
  stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#temiloladocker/numeric-app:v1#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }
    
  }
}
  

