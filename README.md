# Course Registration — DevOps Lab Assessment

## 1. Project Overview

This project implements a Java-based Course Registration application and demonstrates Git version control, Maven multi-module builds, JUnit 5 testing, Jenkins Declarative Pipeline configuration, and Kubernetes deployment using Minikube.

## 2. Prerequisites

- Java JDK 17
- Apache Maven
- Git
- Docker Desktop
- kubectl
- Minikube
- Visual Studio Code (recommended)

Verify the installations:

```powershell
java -version
mvn -version
git --version
docker info
kubectl version --client
minikube version
```

## 3. Project Structure

```text
Student-Starter/
├── pom.xml
├── course-registration-app/
│   ├── pom.xml
│   └── src/
│       ├── main/java/com/university/
│       └── test/java/com/university/
├── Jenkinsfile
├── deployment.yaml
├── service.yaml
└── README.md
```

The root POM is the parent Maven project and has `pom` packaging. It declares `course-registration-app` as a module, centralizes Java and plugin configuration, and manages the JUnit 5 dependency version. The child module inherits the parent coordinates and builds the application as a JAR.

## 4. Build and Test

Run these commands from the repository root:

```powershell
mvn clean test
mvn package
```

The parent Maven project builds the child module through the Maven reactor.

### Test Results

Recorded Surefire results:

| Test suite | Tests | Failures | Errors | Skipped |
|---|---:|---:|---:|---:|
| CourseTest | 3 | 0 | 0 | 0 |
| RegistrationServiceTest | 9 | 0 | 0 | 0 |
| **Total** | **12** | **0** | **0** | **0** |

The Maven reactor reported `BUILD SUCCESS`. The generated test reports are located at:

```text
course-registration-app/target/surefire-reports/
```

The packaged JAR is generated under:

```text
course-registration-app/target/
```

## 5. Registration Defect and Credit-Limit Change

### Course capacity

The capacity validation rejects a registration when the current enrollment is greater than or equal to the course capacity.

The tests verify that:
- A full course rejects a new registration.
- A course with one seat remaining accepts a registration.
- Rejected registration does not change the student's registration state, total credits, or course enrollment count.

### Credit limit

The maximum permitted registered credits has been changed from 20 to 24.

The tests verify that:
- Registration reaching exactly 24 credits is accepted.
- Registration exceeding 24 credits is rejected.
- Rejected registration leaves the student's registered courses, credit total, and course enrollment count unchanged.

## 6. Git Workflow

Development was performed on the feature branch:

```text
feature/registration-fixes
```

The recorded feature-branch commits are:

```text
bc7974b Add Jenkins declarative CI pipeline
5865a40 Refactor project into Maven parent and child modules
d44795b Increase maximum registered credits to 24
29a3c1b Fix course capacity boundary validation
```

The original starter commit is:

```text
cef2bff Initial student assessment starter
```

Final merge status and working-tree cleanliness must be verified before submission.

## 7. Jenkins Declarative Pipeline

The root `Jenkinsfile` targets a Windows Jenkins agent and uses `bat` steps.

The pipeline:
1. Checks out the source from SCM.
2. Verifies Java and Maven availability.
3. Runs `mvn -B clean test`.
4. Runs `mvn -B package` only if the test stage succeeds.
5. Publishes JUnit XML reports from `course-registration-app/target/surefire-reports/TEST-*.xml`.
6. Archives `course-registration-app/target/*.jar` only after overall success.

A live Jenkins execution was not required by the assessment specification. Do not interpret the existence of the Jenkinsfile as evidence of a live Jenkins run.

## 8. Kubernetes Deployment with Minikube

Start the local cluster:

```powershell
minikube start --driver=docker
kubectl get nodes
```

Deploy Nginx and expose it with NodePort:

```powershell
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl rollout status deployment/nginx-deployment
```

Verify the resources:

```powershell
kubectl get deployments
kubectl get pods -o wide
kubectl get services
kubectl describe deployment nginx-deployment
kubectl describe service nginx-service
```

Access the service:

```powershell
minikube service nginx-service --url
```

The Deployment uses the `nginx:1.27` image and initially requests two replicas. The Service uses NodePort 30080 and selects Pods labelled `app=nginx`.

### Scaling and self-healing

Scale the Deployment to three replicas:

```powershell
kubectl scale deployment/nginx-deployment --replicas=3
kubectl rollout status deployment/nginx-deployment
kubectl get deployments
kubectl get pods
```

Delete one Pod to demonstrate self-healing:

```powershell
$pod = kubectl get pods -l app=nginx -o jsonpath="{.items[0].metadata.name}"
kubectl delete pod $pod
kubectl get pods -w
```

The Deployment controller reconciles the actual state with the desired replica count and creates a replacement Pod when a managed Pod is deleted.

### Recorded verification

The observed Deployment status was `3/3` ready replicas. All three Nginx Pods were running, the NodePort Service exposed port `30080`, and the Service described three Pod endpoints. The Nginx URL opened successfully.

Include the actual Pod replacement command output and any screenshots you capture as additional evidence.

## 9. Evidence and Limitations

Evidence should consist of genuine Maven test summaries, Git history, Jenkins configuration, Kubernetes command outputs, and screenshots captured during execution.

The local Kubernetes deployment was verified through Minikube. A live Jenkins run should only be reported if one is actually performed.

Generated `target/` directories must remain uncommitted. No credentials, access tokens, or cloud secrets should be included in the repository.
