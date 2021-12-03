pipeline {
  agent any

  stages {
      stage('Build Artifact - Maven') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar'
            }
          }

      stage('SonarQube - SAST'){
        steps {
          sh "mvn sonar:sonar -Dsonar.projectKey=numeric-applicaiton -Dsonar.host.url=http://devsecops-demo-test.eastus.cloudapp.azure.com:9000 -Dsonar.login=08bb2c00895c02191c9df73d2545e6d4cb53865f"
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
        steps{
          sh "mvn org.pitest:pitest-maven:mutationCoverage"
        }
        post {
          always {
            pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
          }
        }
      }

      stage('Docker-Build and Push') {
          steps {
             withDockerRegistry([credentialsId:"docker-hub", url: ""]) {
             sh 'printenv'
             sh 'docker build -t jemstech/numeric-app:""$GIT_COMMIT"" .'
             sh 'docker push jemstech/numeric-app:""$GIT_COMMIT""'
          }
          }
      }
      stage('Kubernetes Deployment - DEV') {
        steps{
          withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#jemstech/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
          }

        }

      }
    }
}
