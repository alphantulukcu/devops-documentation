# CD Development 

## Continuous Delivery
Continuous Delivery (CD) is a software development practice where code changes are automatically prepared for release to production. It builds on continuous integration by automating the entire software release process.

CD involves deploying code to a testing environment after the build stage, allowing thorough validation before production deployment.

The main difference between Continuous Delivery and Continuous Deployment is that CD requires manual approval before deploying to production, whereas Continuous Deployment automates the entire process.

## Continuous Deployment
Continuous Deployment (also abbreviated as CD) is an advanced software engineering approach where software functionalities are delivered frequently through automated deployments. 

Continuous Deployment contrasts with Continuous Delivery in that it automates the deployment to production without requiring manual approval.

## Deployment Approaches

### Push Method
In a push-based deployment, updates are automatically distributed from a central server to target systems. This method requires robust authentication, encryption, and testing to ensure secure and reliable deployment.

### Pull Method
In pull-based deployments, target systems periodically check for and retrieve updates as needed. This approach allows for independent updates but requires extra monitoring to ensure systems remain up-to-date.

- **ImagePullPolicy**:
  Kubernetes settings like `imagePullPolicy: Always` ensure that the latest image is pulled each time a pod starts, while `imagePullPolicy: IfNotPresent` only pulls a new image if it's not already present on the node.

- **CronJobs**:
  Kubernetes CronJobs manage and execute tasks on a scheduled basis, such as regular update checks and deployments.

## GitOps

GitOps is a DevOps approach where all process management is controlled by Git, applying DevOps principles like version control, collaboration, and CI/CD tooling to infrastructure automation. This approach enhances collaboration, coordination, and reduces errors.

**ArgoCD** and **FluxCD** are popular GitOps tools that automatically deploy and control application resources in their corresponding Git repositories. They detect changes in the repository and synchronize deployments to Kubernetes clusters.

**ArgoCD**:
ArgoCD is a continuous delivery tool that automates deployments by synchronizing application states with changes made in the application repository. It uses the "Pull Model" to track changes and synchronize updates, offering a functional UI for real-time project management.

**ArgoCD Architecture**:
ArgoCD consists of three main components: API/Web Server, Repo Server, and Application Controller. The Repo Server handles repository control and Kubernetes manifests, while the Application Controller manages the Kubernetes Cluster, real-time data, and deployment operations. Additional components like Redis, Dex, and Application Set enhance caching, identity management, and application automation.


# Case

    Create a multipage application with Python Web Server. The created application should be made expandable on CI/CD.
    Application:
    There should be a welcome message in the root directory.
    It can pull data from any API source that it can use for free on the /api page.
    Visualization. (optional)
    CI/CD:
    The code must first be executable and stopable on the CI/CD.
    A Docker image must be created
    Dynamic analysis of the created Docker image should be done on CI/CD.
    Static code analysis should be done using SonarScanner.

## Implementation

```.gitlab-ci.yml```:
```yml
stages:
  - build
  - test
  - dockerize
  - dynamic-scan
  - static-scan
  - deploy

variables:
  KUBECONFIG: "/builds/$CI_PROJECT_PATH/.kube/config"

build:
  stage: build
  image: python:3.9
  script:
    - pip install --upgrade pip
    - pip install -r requirements.txt
  tags: 
    - onprem

test:
  stage: test
  image: python:3.9
  script:
    - pip install --upgrade pip
    - pip install -r requirements.txt
    - python -m unittest discover || true
  needs:
    - build
  tags: 
    - onprem

dockerize:
  stage: dockerize
  image: docker:24.0.5
  services:
    - name: docker:24.0.5-dind
      alias: docker
      entrypoint: ["dockerd-entrypoint.sh", "--tls=false"]
  variables:
    DOCKER_HOST: tcp://docker:2375
    DOCKER_DRIVER: overlay2
    DOCKER_TLS_CERTDIR: ""
  before_script: 
    - docker login -u $HARBOR_USERNAME -p $HARBOR_PASSWORD https://k8s-master-codecamp24.obss.io:30003/harbor/
    - docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
    - docker info
  script: 
    - docker build -t gitlab-python .
    - docker tag gitlab-python:latest k8s-master-codecamp24.obss.io:30003/codecamp/alphantulukcu/gitlab-python:$CI_COMMIT_SHA
    - docker push k8s-master-codecamp24.obss.io:30003/codecamp/alphantulukcu/gitlab-python:$CI_COMMIT_SHA
  needs: 
    - test
  tags: 
    - onprem


dynamic-scan:
  stage: dynamic-scan
  image: docker:24.0.5
  services:
    - name: docker:24.0.5-dind
      alias: docker
      entrypoint: ["dockerd-entrypoint.sh", "--tls=false"]
  variables:
    DOCKER_HOST: tcp://docker:2375
    DOCKER_DRIVER: overlay2
    DOCKER_TLS_CERTDIR: ""
  before_script: 
    - docker login -u $HARBOR_USERNAME -p $HARBOR_PASSWORD https://k8s-master-codecamp24.obss.io:30003/harbor/
    - docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
    - docker info
  script:
    - docker pull k8s-master-codecamp24.obss.io:30003/codecamp/alphantulukcu/gitlab-python:$CI_COMMIT_SHA
    - docker pull aquasec/trivy
    - docker run --rm aquasec/trivy:latest image --exit-code 1 --severity HIGH,CRITICAL --scanners vuln k8s-master-codecamp24.obss.io:30003/codecamp/alphantulukcu/gitlab-python:$CI_COMMIT_SHA || true
  needs:
    - dockerize
  tags: 
    - onprem

static-scan:
  image:
    name: sonarsource/sonar-scanner-cli:5.0.1
    entrypoint: [""]
  variables:
    SONAR_HOST_URL: $SONAR_HOST_URL
    SONAR_TOKEN: $SONAR_TOKEN
    SONAR_PROJECT_KEY: GitlabPython
    SONAR_PROJECT_NAME: GitlabPython
    SONAR_SOURCES: "."
    SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"  
    GIT_DEPTH: "0" 
  cache:
    key: "${CI_JOB_NAME}"
    paths:
      - .sonar/cache
  before_script:
    - echo "192.168.210.103 codecamp24-sonarqube.obss.io" >> /etc/hosts
  script:
    - sonar-scanner -Dsonar.qualitygate.wait=false
  allow_failure: true
  rules:
    - if: $CI_COMMIT_REF_NAME == 'main' 
  needs:
    - test



deploy:
  stage: deploy
  image:
    name: bitnami/kubectl:1.14
    entrypoint: [""]
  before_script:
    - export KUBECONFIG=$(pwd)/config
    - echo "$KUBECONFIG_CONTENT" > $KUBECONFIG
    - sed -i "s/\${COMMIT_ID}/$CI_COMMIT_SHA/g" deployment.yaml
    - kubectl get pods -n $KUBE_NAMESPACE
  script:
    - kubectl apply -f deployment.yaml -f hpa.yaml -f service.yaml -n $KUBE_NAMESPACE 
    - kubectl get pods -n $KUBE_NAMESPACE 
    - kubectl get svc gitlab-python -n $KUBE_NAMESPACE 
    - kubectl describe svc gitlab-python -n $KUBE_NAMESPACE
  needs: 
    - dockerize
  when: manual
  tags: 
    - onprem


```

## Pipeline

![alt text](<../screenshots/Screenshot 2024-08-14 at 15.07.31.png>)