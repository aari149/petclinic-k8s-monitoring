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
        sh 'mvn -Dcheckstyle.skip=true -Dtest=!PostgresIntegrationTests test'
    }
}
    
          
        stage('Docker Build'){
            steps{
                sh 'docker build -t petclinic:v2 .'
            }
        }
        stage('Trivy Security Scan'){
            steps{
            sh 'trivy image --severity HIGH,CRITICAL petclinic:v2'
    }
}
        stage('Docker push'){
            steps{
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                    )]){
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker tag petclinic:v1 $DOCKER_USER/petclinic:v2
                    docker push $DOCKER_USER/petclinic:v2
                    '''
                }
                   }
                      }
    }
}
