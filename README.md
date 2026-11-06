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
- `PUSH_IMAGE`: set to `true` to log in and push to the configured registry;
  leave it `false` for the local minikube workflow

The Jenkinsfile performs checkout, Maven compilation and tests, Docker image
build, Kubernetes deployment, rollout waiting, and readiness verification.
When `PUSH_IMAGE=true`, it also performs an authenticated registry push. When
using the default local minikube workflow, Jenkins loads the image directly
into minikube instead. Do not commit registry passwords or kubeconfig files.

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

The deployment uses `IfNotPresent` so it works with both images loaded into
minikube and images pulled from a registry. Replace the registry repository in
the manifest if you use a different Docker Hub account.
