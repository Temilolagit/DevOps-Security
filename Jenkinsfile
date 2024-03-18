pipeline {
  agent any
  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "temiloladocker/numeric-app:v1"
    applicationURI = "/increment/99"
  }

  stages{
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
    }
   stage('Mutation Tests - PIT') {
      steps {
        sh "mvn org.pitest:pitest-maven:mutationCoverage"
      }
    }
  stage('SonarQube - SAST') {
      steps {
        withSonarQubeEnv('sonar-server') {
          sh "mvn sonar:sonar \
		              -Dsonar.projectKey=numeric \
		              -Dsonar.host.url=http://34.197.199.7:9000"
        }
        timeout(time: 2, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true
          }
        }
      }
    }
  stage('Vulnerability Scan - Docker ') {
      steps {
            sh "mvn dependency-check:check"
          }
      }
    

  stage('Docker Build and Push') {
      steps {
          sh 'docker build -t temiloladocker/numeric-app:v1 .'
        }
      }
    
  stage('Trivy Image Scan') {
      steps {
            sh "/usr/local/bin/trivy image temiloladocker/numeric-app:v1 > trivy_image_result.txt"
         }
     }
  stage('OPA') {
      steps {
            sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
         }
     }

  stage('Docker Push') {
      steps {
        withDockerRegistry([credentialsId: "Docker_hub", url: ""]) {
          script {
              sh 'docker push temiloladocker/numeric-app:v1'
               echo "Push Image to Registry"
          }
      }
    }
  }

  //stage('Kubernetes Deployment - DEV') {
     // steps {
      //  withKubeConfig([credentialsId: 'kubeconfig']) {
        //  sh "sed -i 's#replace#temiloladocker/numeric-app:v1#g' k8s_deployment_service.yaml"
        //  sh "kubectl apply -f k8s_deployment_service.yaml"
  //}
     // }
 // }
  
  stage('K8S Deployment - DEV') {
      steps {
        parallel(
          "Deployment": {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "bash k8s-deployment.sh"
            }
          },
          "Rollout Status": {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "bash k8s-deployment-rollout-status.sh"
            }
          }
        )
      }
    }
  }
  post {
    always {
      junit 'target/surefire-reports/*.xml'
      jacoco execPattern: 'target/jacoco.exec'
      pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
      dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
    }

    // success {

    // }

    // failure {

    // }
  }
}

  

