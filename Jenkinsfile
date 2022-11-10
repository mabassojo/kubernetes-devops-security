pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar'
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
        sh "mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=numeric"
        // sh "quality_status=$(curl -s -u 90a16c9b99ed201515835cc11ab168b34c549b37:https://sonarcloud.io/api/qualitygates/project_status?projectKey=numeric | jq -r '.projectStatus.status')"
        // sh echo 'SonarCloud analysistatus is $quality_status';
      }
    }
     stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
          sh 'printenv'
          sh 'docker build -t maryab/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push maryab/numeric-app:""$GIT_COMMIT""'
        }
      }
    }
    stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kubecfg']) {
          sh "sed -i 's#replace#maryab/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }
  }
}
