pipeline {
    agent any
    environment {
        DOCKER_REGISTRY = 'docker.io'
        IMAGE_REPOSITORY = 'rithwik8/cicd-demo'
        IMAGE_TAG = "${BUILD_NUMBER}"
        KUBE_CONTEXT = 'minikube'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build and Test') {
            steps {
                sh 'mvn clean verify'
            }
        }
        stage('Docker Build') {
            steps {
                sh 'docker build --pull -t ${DOCKER_REGISTRY}/${IMAGE_REPOSITORY}:${IMAGE_TAG} .'
            }
        }
        stage('Push Image') {
            steps {
                sh '''
                    docker push "$DOCKER_REGISTRY/$IMAGE_REPOSITORY:$IMAGE_TAG"
                '''
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl --context "$KUBE_CONTEXT" apply -f k8s/deployment.yaml
                    kubectl --context "$KUBE_CONTEXT" \
                        -n default set image deployment/cicd-demo \
                        cicd-demo="$DOCKER_REGISTRY/$IMAGE_REPOSITORY:$IMAGE_TAG"
                    kubectl --context "$KUBE_CONTEXT" \
                        -n default rollout status deployment/cicd-demo --timeout=120s
                '''
            }
        }
        stage('Verify Availability') {
            steps {
                sh '''
                    kubectl --context "$KUBE_CONTEXT" -n default get deployment cicd-demo
                    kubectl --context "$KUBE_CONTEXT" -n default get pods \
                        -l app=cicd-demo
                    kubectl --context "$KUBE_CONTEXT" -n default get service cicd-demo-service
                '''
            }
        }
    }
}