pipeline {

    agent any

    environment {
        DOCKER_IMAGE = "shivam3294/student-event"
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/shivamsaini124/student-event.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build --pull=false -t $DOCKER_IMAGE:latest .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )
                ]) {
                    sh 'echo "$PASS" | docker login -u "$USER" --password-stdin'
                    sh 'docker push $DOCKER_IMAGE:latest'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kuberconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    sh '''
                        export KUBECONFIG="$KUBECONFIG"

                        kubectl apply -f deployment.yaml

                        kubectl rollout status deployment/student-event

                        kubectl get deployment student-event

                        kubectl get pods -l app=student-event

                        kubectl get service student-event-service
                    '''
                }
            }
        }
    }

    post {

        success {
            echo '========================================'
            echo 'STUDENT EVENT DEPLOYMENT SUCCESSFUL'
            echo '========================================'
            echo "Docker Image: ${DOCKER_IMAGE}:latest"
            echo 'Replicas: 3'
            echo 'NodePort: 30080'
            echo '========================================'
        }

        failure {
            echo '========================================'
            echo 'PIPELINE FAILED'
            echo '========================================'
        }
    }
}