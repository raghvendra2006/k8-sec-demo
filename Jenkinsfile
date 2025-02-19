pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later test
            }
        }

       stage('Unit test - JUnit & Jacoco ') {
            steps {
              sh "mvn test"
            }
//         post {
//           always {
//             junit 'target/surefire-reports/*.xml'
//             jacoco(execPattern: 'target/jacoco.exec')
            // jacoco execPattern: 'target/jacoco.exec'
//           }
//         }
       }
    
     stage('Mutation Tests - PIT') {
      steps {
        sh "mvn org.pitest:pitest-maven:mutationCoverage"
      }
//      post {
//        always {
//          pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
//        }
//      }
    }

    stage('SonarQube - SAST') {
      steps {
          sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.host.url=http://34.127.78.67:9000 -Dsonar.token=sqp_187831d85cedc97be671f1da99a0d59976f085de"
      }
    }
        
    //    stage('Vulnerability Scan - Docker ') {
    //      steps {
    //         sh "mvn dependency-check:check"   
    //        }
    // }

    stage('Vulnerability Scan - Docker') {
      steps {
        parallel(
          "Dependency Scan": {
            sh "mvn dependency-check:check"
          },
          "Trivy Scan": {
            sh "bash trivy-docker-image-scan.sh"
          }
        )
      }
    }

    stage('Docker Build and Push') {
            steps {
              withDockerRegistry([credentialsId: "dockerhub", url: ""]) {
            //  docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
              sh 'printenv'
              sh 'sudo docker build -t raghvendra2006/numeric-app:""$GIT_COMMIT"" .'
              sh 'docker push raghvendra2006/numeric-app:""$GIT_COMMIT""'
            }
         }
       }
        
       stage('Kubernets Deployment - DEV') {
            steps {
              withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "sed -i 's#replace#raghvendra2006/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
              sh 'kubectl apply -f k8s_deployment_service.yaml'
            }
         }
       }
        
    }
	  post {
        always {
			 junit 'target/surefire-reports/*.xml'
             jacoco(execPattern: 'target/jacoco.exec')
          // jacoco execPattern: 'target/jacoco.exec'
			pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
			dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
        }
    // success {

    // }

    // failure {

    // }
      }
	
}
