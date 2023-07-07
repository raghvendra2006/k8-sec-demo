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
         post {
           always {
             junit 'target/surefire-reports/*.xml'
             jacoco(execPattern: 'target/jacoco.exec')
          // jacoco execPattern: 'target/jacoco.exec'
           }
         }
       }
       stage('Docker Build and Push') {
            steps {
              withDockerRegistry([credentialsId: "dockerhub", url: ""]) {
            //  docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
              sh 'printenv'
              sh 'docker build -t raghvendra2006/numeric-app:""GIT_COMMIT"" .'
              sh 'docker push raghvendra2006/numeric-app:""GIT_COMMIT""'
            }
         }
       }
    }
}
