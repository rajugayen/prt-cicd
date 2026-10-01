pipeline {
    agent { label 'ci-agent' }

    environment {
        IMAGE = "${DOCKERHUB_USER}/prt-cicd:${BUILD_NUMBER}"
        LATEST_IMAGE = "${DOCKERHUB_USER}/prt-cicd:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE -t $LATEST_IMAGE .'
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_TOKEN'
                )]) {
                    sh '''
                        echo "$DH_TOKEN" | docker login -u "$DH_USER" --password-stdin
                        docker push "$IMAGE"
                        docker push "$LATEST_IMAGE"
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sshagent(credentials: ['k8s-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@$K8S_HOST                           "kubectl set image deployment/prt-web prt-web=$IMAGE &&                            kubectl rollout status deployment/prt-web --timeout=120s"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'PRT – CI/CD Completed Successfully'
        }
        always {
            sh 'docker image prune -f || true'
        }
    }
}
