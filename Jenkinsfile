pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/aari149/petclinic-k8s-monitoring.git'
            }
        }

        stage('Build') {
            steps {
                sh 'java -version'
                sh 'mvn -B -DskipTests -Dcheckstyle.skip=true clean package'
            }
        }

       
           
               stage('Test') {
    steps {
        sh 'mvn -Dcheckstyle.skip=true -Dspring.docker.compose.skip.in-tests=true test'
    }
}
          
        stage('Docker Build'){
            steps{
                sh 'docker build -t petclinic:v1 .'
            }
        }
        stage('Trivy Security Scan'){
            steps{
            sh 'trivy image --severity HIGH,CRITICAL petclinic:v1'
    }
}
}
}
