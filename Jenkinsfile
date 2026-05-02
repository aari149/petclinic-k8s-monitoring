pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        IMAGE_NAME = 'petclinic'
        IMAGE_TAG  = "${BUILD_NUMBER}"
        APP_NAME   = 'petclinic'
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
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=${APP_NAME} \
                        -Dsonar.login=${SONAR_TOKEN} \
                        -Dsonar.java.binaries=target/classes
                        """
                    }
                }
            }
        }
        stage('Sonar quality gate'){
              steps{
                 waitForQualityGate abortPipeline: true
              }
              }
        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Trivy Security Scan') {
            steps {
                // sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${IMAGE_NAME}:${IMAGE_TAG}"
                sh "trivy image --severity HIGH,CRITICAL ${IMAGE_NAME}:${IMAGE_TAG} || true"
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} \$DOCKER_USER/${IMAGE_NAME}:${IMAGE_TAG}
                    docker push \$DOCKER_USER/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
        stage('Deploy to k3s (Dev/Staging)') {
    when {
        branch 'master'
    }
    steps {
        withKubeConfig([credentialsId: 'k3s-kubeconfig']) {
            sh """
                kubectl get nodes 
                
                kubectl create namespace staging || true

                kubectl apply -f k8s/ -n staging

                kubectl set image deployment/${APP_NAME} \
                ${APP_NAME}=${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG} \
                -n staging

                kubectl rollout status deployment/${APP_NAME} -n staging
            """
        }
    }
}
    }
}
