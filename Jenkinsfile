pipeline {

    agent any

    environment {
        DOCKER_IMAGE = "shivam3294/student-event"
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Clone Code') {
            steps {
                echo '========================================'
                echo 'CLONING CODE FROM GITHUB'
                echo '========================================'

                git branch: 'main',
                    url: 'https://github.com/shivamsaini124/student-event.git'
            }
        }

        stage('Docker Build') {
            steps {
                echo '========================================'
                echo 'BUILDING DOCKER IMAGE'
                echo '========================================'

                sh '''
                    docker build --pull=false \
                        -t $DOCKER_IMAGE:$DOCKER_TAG .

                    docker tag \
                        $DOCKER_IMAGE:$DOCKER_TAG \
                        $DOCKER_IMAGE:latest
                '''
            }
        }

        stage('Push Image') {
            steps {
                echo '========================================'
                echo 'PUSHING IMAGE TO DOCKER HUB'
                echo '========================================'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )
                ]) {
                    sh '''
                        echo "$PASS" | docker login \
                            -u "$USER" \
                            --password-stdin

                        docker push $DOCKER_IMAGE:$DOCKER_TAG
                        docker push $DOCKER_IMAGE:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo '========================================'
                echo 'DEPLOYING TO KUBERNETES'
                echo '========================================'

                withCredentials([
                    file(
                        credentialsId: 'kuberconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    sh '''
                        export KUBECONFIG="$KUBECONFIG"

                        echo "Checking Kubernetes cluster..."
                        kubectl config current-context

                        echo "Applying Kubernetes configuration..."
                        kubectl apply -f deployment.yaml

                        echo "Updating deployment to new Docker image..."

                        kubectl set image \
                            deployment/student-event \
                            student-event=$DOCKER_IMAGE:$DOCKER_TAG

                        echo "Waiting for rollout..."

                        kubectl rollout status \
                            deployment/student-event

                        echo "========================================"
                        echo "DEPLOYMENT STATUS"
                        echo "========================================"

                        kubectl get deployment student-event

                        echo "========================================"
                        echo "POD STATUS"
                        echo "========================================"

                        kubectl get pods -l app=student-event

                        echo "========================================"
                        echo "SERVICE STATUS"
                        echo "========================================"

                        kubectl get service student-event-service

                        echo "========================================"
                        echo "IMAGE USED"
                        echo "========================================"

                        kubectl get deployment student-event \
                            -o jsonpath="{.spec.template.spec.containers[0].image}"

                        echo
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
            echo "Docker Image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
            echo 'Replicas: 3'
            echo 'NodePort: 30081'
            echo '========================================'
        }

        failure {
            echo '========================================'
            echo 'PIPELINE FAILED'
            echo '========================================'
        }
    }
}