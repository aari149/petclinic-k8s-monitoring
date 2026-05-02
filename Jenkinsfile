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
        DOCKER_USER = 'ashwiniethiraj'   // ✅ ADD THIS
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

        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Trivy Security Scan') {
            steps {
                sh "trivy image --severity HIGH,CRITICAL ${IMAGE_NAME}:${IMAGE_TAG} || true"
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} $USER/${IMAGE_NAME}:${IMAGE_TAG}
                    docker push $USER/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy to k3s') {
            when {
                branch 'develop'
            }
            steps {
                withKubeConfig([credentialsId: 'k3s-kubeconfig']) {
                    sh '''
                    set -e

                    export IMAGE="$DOCKER_USER/$IMAGE_NAME:$IMAGE_TAG"

                    echo "Checking cluster..."
                    kubectl get nodes

                    echo "Creating namespace..."
                    kubectl create namespace staging || true

                    echo "Applying manifests..."
                    kubectl apply -f k8s/ -n staging

                    echo "Updating image..."
                    kubectl set image deployment/$APP_NAME $APP_NAME=$IMAGE -n staging || true

                    echo "Rollout status..."
                    kubectl rollout status deployment/$APP_NAME -n staging
                    '''
                }
            }
        }

        stage('Cleanup Docker Images') {
            steps {
                sh '''
                echo "Cleaning old images..."
                docker image prune -f
                docker builder prune -f
                '''
            }
        }
    }
}
