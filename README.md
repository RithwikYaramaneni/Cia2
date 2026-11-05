# CI/CD Demo

This project is a Spring Boot application packaged with Maven, containerized with
Docker, and deployable to Kubernetes through Jenkins.

## Run locally

```bash
mvn spring-boot:run
```

Open `http://localhost:8080/` to verify the application.

## Test locally

```bash
mvn clean verify
```

The test suite includes an HTTP integration test for the root endpoint.

## Jenkins configuration

Configure these environment values in the Jenkins pipeline or job:

- `DOCKER_REGISTRY`: registry host, such as `docker.io`
- `IMAGE_REPOSITORY`: registry path, such as `rithwik8/cicd-demo`
- `DOCKER_CREDENTIALS_ID`: Jenkins username/password credential ID
- `KUBE_CONTEXT`: the configured context for the cloud Kubernetes cluster

The Jenkinsfile performs checkout, Maven compilation and tests, Docker image
build, registry push, Kubernetes deployment, rollout waiting, and readiness
verification. Do not commit registry passwords or kubeconfig files.

## Kubernetes deployment

The manifest in `k8s/deployment.yaml` exposes the application through a
`LoadBalancer` service. Jenkins applies the manifest and then updates the
deployment to the immutable build-number image tag:

```bash
kubectl apply -f k8s/deployment.yaml
kubectl set image deployment/cicd-demo \
  cicd-demo=docker.io/my-user/cicd-demo:<build-number>
kubectl rollout status deployment/cicd-demo
```

Replace the registry repository in the manifest if you use a different
Docker Hub account.
