pipeline {
    agent any

    tools{
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
                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'       
    }
    stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('sonar') {
            sh '''
            mvn sonar:sonar \
              -Dsonar.projectKey=petclinic \
              -Dsonar.host.url=http://13.233.186.89:9000 \
              -Dsonar.login=$SONAR_TOKEN
            '''
        }
    }
}
}
