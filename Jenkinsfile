pipeline {
    agent any
    environment {
        DOCKER_REGISTRY = 'docker.io'
        IMAGE_REPOSITORY = 'rithwik8/cicd-demo'
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        KUBE_CONTEXT = 'minikube'
        PUSH_IMAGE = 'false'
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
            when {
                environment name: 'PUSH_IMAGE', value: 'true'
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS_ID}",
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh '''
                        set +x
                        echo "$DOCKER_PASSWORD" | docker login "$DOCKER_REGISTRY" \
                            --username "$DOCKER_USERNAME" --password-stdin
                        docker push "$DOCKER_REGISTRY/$IMAGE_REPOSITORY:$IMAGE_TAG"
                    '''
                }
            }
        }
        stage('Load Image into Minikube') {
            when {
                allOf {
                    environment name: 'KUBE_CONTEXT', value: 'minikube'
                    environment name: 'PUSH_IMAGE', value: 'false'
                }
            }
            steps {
                sh '''
                    minikube --profile minikube image load \
                        "$DOCKER_REGISTRY/$IMAGE_REPOSITORY:$IMAGE_TAG"
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
    post {
        always {
            junit testResults: 'target/surefire-reports/*.xml', allowEmptyResults: true
            script {
                if (env.PUSH_IMAGE == 'true') {
                    sh 'docker logout "$DOCKER_REGISTRY" || true'
                }
            }
        }
    }
}