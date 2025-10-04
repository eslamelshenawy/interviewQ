# CI/CD Interview Questions and Answers

## Table of Contents
1. [CI/CD Fundamentals](#cicd-fundamentals)
2. [Pipeline Stages](#pipeline-stages)
3. [Jenkins](#jenkins)
4. [GitLab CI](#gitlab-ci)
5. [GitHub Actions](#github-actions)
6. [Docker Integration](#docker-integration)
7. [Kubernetes Deployment](#kubernetes-deployment)
8. [Testing Automation](#testing-automation)
9. [Code Quality (SonarQube)](#code-quality-sonarqube)
10. [Blue-Green Deployment](#blue-green-deployment)
11. [Canary Deployment](#canary-deployment)
12. [Real-World Pipeline Examples](#real-world-pipeline-examples)

---

## CI/CD Fundamentals

### Q1: What is CI/CD and why is it important?

**Answer:**
CI/CD stands for Continuous Integration and Continuous Deployment/Delivery:

- **Continuous Integration (CI)**: Automatically integrating code changes from multiple contributors into a shared repository frequently. Each integration triggers automated builds and tests.

- **Continuous Deployment (CD)**: Automatically deploying all code changes to production after passing automated tests.

- **Continuous Delivery**: Automatically deploying to staging/pre-production, but requiring manual approval for production.

**Importance:**
- Faster time to market
- Early bug detection
- Reduced integration risks
- Improved code quality
- Automated repetitive tasks
- Better team collaboration
- Consistent deployment process

### Q2: What is the difference between Continuous Delivery and Continuous Deployment?

**Answer:**

**Continuous Delivery:**
- Code is automatically prepared for production release
- Requires manual approval/trigger for production deployment
- Gives teams control over when to release
- Suitable for regulated industries or when business timing matters

**Continuous Deployment:**
- Every change that passes automated tests goes to production automatically
- No manual intervention required
- Fastest feedback loop
- Requires high confidence in automated testing
- Suitable for SaaS applications with quick iteration needs

### Q3: What are the key components of a CI/CD pipeline?

**Answer:**

1. **Source Control**: Git repository (GitHub, GitLab, Bitbucket)
2. **Build System**: Compilation and artifact creation
3. **Test Automation**: Unit, integration, and E2E tests
4. **Code Quality Tools**: SonarQube, ESLint, Checkstyle
5. **Artifact Repository**: Nexus, Artifactory, Docker Registry
6. **Deployment Tools**: Kubernetes, Docker, Ansible
7. **Monitoring Tools**: Prometheus, Grafana, ELK Stack
8. **Notification System**: Slack, email, PagerDuty

### Q4: What is Pipeline as Code?

**Answer:**

Pipeline as Code is the practice of defining your CI/CD pipeline in a versioned file stored in the repository alongside your application code.

**Benefits:**
- Version controlled pipeline configuration
- Peer review of pipeline changes
- Easy rollback of pipeline changes
- Pipeline changes tracked with code changes
- Reproducible builds across environments
- Self-documenting infrastructure

**Examples:**
- **Jenkins**: Jenkinsfile
- **GitLab CI**: .gitlab-ci.yml
- **GitHub Actions**: .github/workflows/*.yml
- **Azure DevOps**: azure-pipelines.yml
- **CircleCI**: .circleci/config.yml

### Q5: What are some common CI/CD best practices?

**Answer:**

1. **Commit Frequently**: Small, incremental changes
2. **Maintain Fast Builds**: Keep build time under 10 minutes
3. **Automate Everything**: Testing, deployment, rollback
4. **Test in Production-like Environment**: Staging mirrors production
5. **Implement Feature Flags**: Control feature rollout
6. **Monitor Everything**: Logs, metrics, traces
7. **Secure Your Pipeline**: Scan for vulnerabilities, manage secrets
8. **Version Your Artifacts**: Semantic versioning
9. **Make Deployments Reversible**: Easy rollback mechanism
10. **Keep Pipeline DRY**: Reusable pipeline components

---

## Pipeline Stages

### Q6: Describe the typical stages in a CI/CD pipeline.

**Answer:**

**1. Source Stage:**
- Trigger: Git push, merge request, scheduled
- Checkout code from repository
- Determine what changed

**2. Build Stage:**
- Compile code
- Resolve dependencies
- Create artifacts (JAR, Docker image, etc.)
- Generate build metadata

**3. Test Stage:**
- Unit tests (fast, isolated)
- Integration tests (component interaction)
- End-to-end tests (full user workflows)
- Performance tests (load, stress)
- Security tests (SAST, DAST)

**4. Analysis Stage:**
- Code quality scanning
- Code coverage reports
- Security vulnerability scanning
- License compliance checking

**5. Package Stage:**
- Create deployment packages
- Build Docker images
- Tag and version artifacts
- Push to artifact repository

**6. Deploy Stage:**
- Deploy to staging/test environment
- Run smoke tests
- Deploy to production (with approval if needed)
- Database migrations

**7. Monitor Stage:**
- Health checks
- Performance monitoring
- Error tracking
- User analytics

### Q7: How do you handle secrets in CI/CD pipelines?

**Answer:**

**Best Practices:**

1. **Never Hardcode Secrets**: Don't commit secrets to version control
2. **Use Secret Management Tools**:
   - HashiCorp Vault
   - AWS Secrets Manager
   - Azure Key Vault
   - Google Secret Manager

3. **CI/CD Platform Secret Storage**:
   - Jenkins Credentials Plugin
   - GitLab CI/CD Variables (masked & protected)
   - GitHub Actions Secrets
   - Environment variables (encrypted)

4. **Access Control**:
   - Role-based access control (RBAC)
   - Principle of least privilege
   - Rotate secrets regularly
   - Audit secret access

**Example (GitLab CI):**
```yaml
deploy:
  script:
    - echo $DATABASE_PASSWORD | docker login -u $DATABASE_USER --password-stdin
    - kubectl create secret generic db-secret
        --from-literal=password=$DATABASE_PASSWORD
  variables:
    DATABASE_USER: $DB_USER  # From GitLab CI/CD variables
    DATABASE_PASSWORD: $DB_PASSWORD  # Masked variable
```

### Q8: How do you ensure pipeline reliability and prevent flaky tests?

**Answer:**

**Strategies:**

1. **Test Isolation**:
   - Each test should be independent
   - Clean up test data after each test
   - Use test containers for dependencies

2. **Stable Test Environment**:
   - Use containerized environments
   - Pin dependency versions
   - Avoid external dependencies when possible

3. **Retry Mechanisms**:
   - Retry flaky tests (but investigate root cause)
   - Use exponential backoff for API calls
   - Implement circuit breakers

4. **Test Categorization**:
   - Tag tests by stability
   - Run stable tests first
   - Quarantine flaky tests

5. **Monitoring and Reporting**:
   - Track test failure rates
   - Identify patterns in failures
   - Alert on unusual failure spikes

**Example (Retry in GitLab CI):**
```yaml
test:
  script:
    - npm test
  retry:
    max: 2
    when:
      - runner_system_failure
      - stuck_or_timeout_failure
```

---

## Jenkins

### Q9: What is a Jenkinsfile and what are its types?

**Answer:**

A **Jenkinsfile** is a text file that contains the definition of a Jenkins Pipeline and is checked into source control.

**Types:**

**1. Declarative Pipeline:**
- Structured, opinionated syntax
- Easier to read and write
- Built-in syntax validation
- Recommended for most use cases

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            steps {
                sh './deploy.sh'
            }
        }
    }
}
```

**2. Scripted Pipeline:**
- Groovy-based DSL
- More flexibility and power
- Requires more programming knowledge
- Used for complex scenarios

```groovy
node {
    stage('Build') {
        sh 'mvn clean package'
    }
    stage('Test') {
        sh 'mvn test'
    }
    stage('Deploy') {
        sh './deploy.sh'
    }
}
```

### Q10: Explain Jenkins Pipeline as Code with a complete example.

**Answer:**

**Complete Declarative Jenkinsfile Example:**

```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.8.6-openjdk-11'
            args '-v $HOME/.m2:/root/.m2'
        }
    }

    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        SONAR_HOST = 'https://sonarqube.example.com'
    }

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'production'],
               description: 'Deployment environment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true,
                    description: 'Run test suite')
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
        timeout(time: 1, unit: 'HOURS')
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_MSG = sh(
                        script: 'git log -1 --pretty=%B',
                        returnStdout: true
                    ).trim()
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building application...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Unit Tests') {
            when {
                expression { params.RUN_TESTS == true }
            }
            steps {
                echo 'Running unit tests...'
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                    jacoco(
                        execPattern: 'target/jacoco.exec',
                        classPattern: 'target/classes',
                        sourcePattern: 'src/main/java'
                    )
                }
            }
        }

        stage('Code Quality Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=myapp \
                        -Dsonar.host.url=${SONAR_HOST}
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    dockerImage = docker.build(
                        "${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    )
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_CREDENTIALS_ID) {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Deploy to Environment') {
            steps {
                script {
                    echo "Deploying to ${params.ENVIRONMENT}..."
                    sh """
                        kubectl set image deployment/myapp \
                        myapp=${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} \
                        -n ${params.ENVIRONMENT}
                    """
                }
            }
        }

        stage('Smoke Tests') {
            steps {
                echo 'Running smoke tests...'
                sh './scripts/smoke-tests.sh ${params.ENVIRONMENT}'
            }
        }
    }

    post {
        success {
            slackSend(
                color: 'good',
                message: "Build ${env.BUILD_NUMBER} succeeded: ${env.GIT_COMMIT_MSG}"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "Build ${env.BUILD_NUMBER} failed: ${env.JOB_NAME}"
            )
        }
        always {
            cleanWs()
        }
    }
}
```

### Q11: How do you implement parallel execution in Jenkins Pipeline?

**Answer:**

**Parallel Execution Example:**

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Parallel Tests') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'mvn test'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'mvn verify -P integration-tests'
                    }
                }
                stage('Security Scan') {
                    steps {
                        sh 'npm audit'
                    }
                }
                stage('Code Quality') {
                    steps {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh './deploy.sh'
            }
        }
    }
}
```

**Parallel with Different Agents:**

```groovy
pipeline {
    agent none

    stages {
        stage('Multi-Platform Build') {
            parallel {
                stage('Build on Linux') {
                    agent { label 'linux' }
                    steps {
                        sh 'make build'
                    }
                }
                stage('Build on Windows') {
                    agent { label 'windows' }
                    steps {
                        bat 'build.bat'
                    }
                }
                stage('Build on macOS') {
                    agent { label 'macos' }
                    steps {
                        sh 'make build'
                    }
                }
            }
        }
    }
}
```

### Q12: What is Jenkins Shared Library and how do you use it?

**Answer:**

**Jenkins Shared Library** allows you to share reusable pipeline code across multiple projects.

**Structure:**
```
(root)
+- vars/              # Global variables (simple pipeline steps)
|   +- buildApp.groovy
|   +- deployApp.groovy
+- src/               # Groovy source files
|   +- org/
|       +- company/
|           +- BuildHelper.groovy
+- resources/         # Resource files
    +- config/
        +- pipeline-config.yml
```

**Define in Jenkinsfile:**

```groovy
// Jenkinsfile
@Library('my-shared-library@main') _

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                buildApp(
                    language: 'java',
                    buildTool: 'maven'
                )
            }
        }
        stage('Deploy') {
            steps {
                deployApp(
                    environment: 'production',
                    version: env.BUILD_NUMBER
                )
            }
        }
    }
}
```

**vars/buildApp.groovy:**

```groovy
def call(Map config) {
    def language = config.language
    def buildTool = config.buildTool

    echo "Building ${language} app using ${buildTool}"

    switch(buildTool) {
        case 'maven':
            sh 'mvn clean package'
            break
        case 'gradle':
            sh './gradlew build'
            break
        case 'npm':
            sh 'npm install && npm run build'
            break
        default:
            error "Unsupported build tool: ${buildTool}"
    }
}
```

**vars/deployApp.groovy:**

```groovy
def call(Map config) {
    def environment = config.environment
    def version = config.version

    echo "Deploying version ${version} to ${environment}"

    withCredentials([
        string(credentialsId: "${environment}-api-key", variable: 'API_KEY')
    ]) {
        sh """
            kubectl set image deployment/myapp \
            myapp=myapp:${version} \
            -n ${environment}
        """
    }
}
```

---

## GitLab CI

### Q13: Explain GitLab CI/CD pipeline configuration structure.

**Answer:**

**Basic .gitlab-ci.yml structure:**

```yaml
# Global configuration
image: node:16-alpine

# Define stages (order matters)
stages:
  - build
  - test
  - deploy

# Variables
variables:
  NODE_ENV: "production"
  DATABASE_URL: "postgres://db:5432"

# Cache configuration
cache:
  paths:
    - node_modules/
    - .npm/

# Before script runs before each job
before_script:
  - npm ci --cache .npm --prefer-offline

# Job definitions
build_app:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

test_unit:
  stage: test
  script:
    - npm run test:unit
  coverage: '/Lines\s*:\s*(\d+\.?\d*)%/'

test_integration:
  stage: test
  script:
    - npm run test:integration
  services:
    - postgres:13
    - redis:6

deploy_staging:
  stage: deploy
  script:
    - npm run deploy:staging
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

deploy_production:
  stage: deploy
  script:
    - npm run deploy:production
  environment:
    name: production
    url: https://example.com
  only:
    - main
  when: manual
```

### Q14: How do you implement Docker-in-Docker (DinD) in GitLab CI?

**Answer:**

**Docker-in-Docker Configuration:**

```yaml
# .gitlab-ci.yml
image: docker:24-dind

services:
  - docker:24-dind

variables:
  DOCKER_HOST: tcp://docker:2376
  DOCKER_TLS_CERTDIR: "/certs"
  DOCKER_TLS_VERIFY: 1
  DOCKER_CERT_PATH: "$DOCKER_TLS_CERTDIR/client"
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

before_script:
  - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY

stages:
  - build
  - test
  - push

build_image:
  stage: build
  script:
    - docker build -t $IMAGE_TAG .
    - docker save $IMAGE_TAG > image.tar
  artifacts:
    paths:
      - image.tar
    expire_in: 1 hour

test_image:
  stage: test
  script:
    - docker load < image.tar
    - docker run --rm $IMAGE_TAG npm test

push_image:
  stage: push
  script:
    - docker load < image.tar
    - docker push $IMAGE_TAG
    - docker tag $IMAGE_TAG $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main
```

### Q15: Explain GitLab CI/CD templates and includes.

**Answer:**

**Using Templates and Includes:**

**1. Local Template (.gitlab/ci/templates/build.yml):**

```yaml
# .gitlab/ci/templates/build.yml
.build_template:
  image: node:16
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
  before_script:
    - npm ci
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
```

**2. Main Pipeline (.gitlab-ci.yml):**

```yaml
# .gitlab-ci.yml
include:
  - local: '.gitlab/ci/templates/build.yml'
  - template: Security/SAST.gitlab-ci.yml
  - project: 'group/shared-pipelines'
    ref: main
    file: '/templates/deploy.yml'
  - remote: 'https://gitlab.com/example/ci-templates/-/raw/main/docker.yml'

stages:
  - build
  - test
  - security
  - deploy

# Use the template
build_frontend:
  extends: .build_template
  stage: build

build_backend:
  extends: .build_template
  stage: build
  variables:
    BUILD_DIR: "backend"
  script:
    - cd backend
    - npm run build

# Override template behavior
build_custom:
  extends: .build_template
  script:
    - npm run custom-build
    - npm run optimize
```

**3. Advanced Include with Rules:**

```yaml
include:
  - local: 'ci/base.yml'
  - local: 'ci/production.yml'
    rules:
      - if: '$CI_COMMIT_BRANCH == "main"'
  - local: 'ci/development.yml'
    rules:
      - if: '$CI_COMMIT_BRANCH == "develop"'

variables:
  DEPLOY_ENV:
    value: "staging"
    description: "Deployment environment"

# Dynamic child pipelines
trigger_child:
  stage: deploy
  trigger:
    include:
      - local: ci/child-pipeline.yml
    strategy: depend
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
```

### Q16: How do you implement multi-project pipelines in GitLab?

**Answer:**

**Parent Pipeline (.gitlab-ci.yml in project A):**

```yaml
stages:
  - trigger

# Trigger downstream project
trigger_frontend:
  stage: trigger
  trigger:
    project: group/frontend-app
    branch: main
    strategy: depend
  variables:
    PARENT_PIPELINE_ID: $CI_PIPELINE_ID
    BACKEND_VERSION: $CI_COMMIT_SHORT_SHA

trigger_backend:
  stage: trigger
  trigger:
    project: group/backend-api
    branch: main
  variables:
    DEPLOY_ENV: "staging"
```

**Child Pipeline (in triggered project):**

```yaml
# frontend-app/.gitlab-ci.yml
stages:
  - build
  - deploy

build:
  stage: build
  script:
    - echo "Building with backend version $BACKEND_VERSION"
    - echo "Triggered by pipeline $PARENT_PIPELINE_ID"
    - npm run build

deploy:
  stage: deploy
  script:
    - npm run deploy
  environment:
    name: $DEPLOY_ENV
```

**Multi-Project Pipeline with Artifacts:**

```yaml
# Parent project
build_library:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/

trigger_consumers:
  stage: trigger
  needs: [build_library]
  trigger:
    project: group/consumer-app
    strategy: depend
  variables:
    LIBRARY_VERSION: $CI_COMMIT_SHA
```

---

## GitHub Actions

### Q17: Explain GitHub Actions workflow structure and syntax.

**Answer:**

**Complete GitHub Actions Workflow:**

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

# Triggers
on:
  push:
    branches: [main, develop]
    paths-ignore:
      - '**.md'
      - 'docs/**'
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * 0'  # Weekly on Sunday
  workflow_dispatch:  # Manual trigger
    inputs:
      environment:
        description: 'Deployment environment'
        required: true
        type: choice
        options:
          - staging
          - production

# Environment variables
env:
  NODE_VERSION: '18'
  DOCKER_REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

# Concurrency control
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  # Job 1: Build and Test
  build:
    name: Build and Test
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16, 18, 20]
        os: [ubuntu-latest, windows-latest, macos-latest]

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for SonarQube

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test -- --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage/coverage-final.json
          flags: unittests

      - name: Build application
        run: npm run build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-${{ matrix.os }}-${{ matrix.node-version }}
          path: dist/
          retention-days: 7

  # Job 2: Security Scanning
  security:
    name: Security Scan
    runs-on: ubuntu-latest
    needs: build

    steps:
      - uses: actions/checkout@v4

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

      - name: Dependency Review
        uses: actions/dependency-review-action@v3
        if: github.event_name == 'pull_request'

  # Job 3: Docker Build and Push
  docker:
    name: Build Docker Image
    runs-on: ubuntu-latest
    needs: [build, security]
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.DOCKER_REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha,prefix={{branch}}-

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          platforms: linux/amd64,linux/arm64

  # Job 4: Deploy
  deploy:
    name: Deploy to ${{ github.event.inputs.environment || 'staging' }}
    runs-on: ubuntu-latest
    needs: docker
    if: github.ref == 'refs/heads/main'
    environment:
      name: ${{ github.event.inputs.environment || 'staging' }}
      url: https://${{ github.event.inputs.environment || 'staging' }}.example.com

    steps:
      - uses: actions/checkout@v4

      - name: Configure kubectl
        uses: azure/k8s-set-context@v3
        with:
          method: kubeconfig
          kubeconfig: ${{ secrets.KUBE_CONFIG }}

      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/myapp \
            myapp=${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
            -n ${{ github.event.inputs.environment || 'staging' }}

          kubectl rollout status deployment/myapp \
            -n ${{ github.event.inputs.environment || 'staging' }}

      - name: Run smoke tests
        run: |
          chmod +x ./scripts/smoke-tests.sh
          ./scripts/smoke-tests.sh ${{ github.event.inputs.environment || 'staging' }}

  # Job 5: Notify
  notify:
    name: Notify Team
    runs-on: ubuntu-latest
    needs: [build, security, docker, deploy]
    if: always()

    steps:
      - name: Send Slack notification
        uses: slackapi/slack-github-action@v1
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK }}
          payload: |
            {
              "text": "Deployment ${{ job.status }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*${{ github.workflow }}* pipeline ${{ job.status }}\n*Repository:* ${{ github.repository }}\n*Branch:* ${{ github.ref_name }}\n*Commit:* ${{ github.sha }}"
                  }
                }
              ]
            }
```

### Q18: How do you create reusable workflows in GitHub Actions?

**Answer:**

**Reusable Workflow (.github/workflows/reusable-build.yml):**

```yaml
name: Reusable Build Workflow

on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string
      environment:
        required: false
        type: string
        default: 'staging'
      run-tests:
        required: false
        type: boolean
        default: true
    secrets:
      npm-token:
        required: true
      deploy-key:
        required: false
    outputs:
      artifact-name:
        description: "Name of the build artifact"
        value: ${{ jobs.build.outputs.artifact }}

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      artifact: ${{ steps.upload.outputs.artifact-name }}

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}

      - name: Install dependencies
        run: npm ci
        env:
          NPM_TOKEN: ${{ secrets.npm-token }}

      - name: Build
        run: npm run build

      - name: Test
        if: ${{ inputs.run-tests }}
        run: npm test

      - name: Upload artifact
        id: upload
        uses: actions/upload-artifact@v4
        with:
          name: build-${{ inputs.environment }}
          path: dist/
```

**Caller Workflow (.github/workflows/main.yml):**

```yaml
name: Main Pipeline

on:
  push:
    branches: [main]

jobs:
  build-staging:
    uses: ./.github/workflows/reusable-build.yml
    with:
      node-version: '18'
      environment: 'staging'
      run-tests: true
    secrets:
      npm-token: ${{ secrets.NPM_TOKEN }}

  build-production:
    uses: ./.github/workflows/reusable-build.yml
    with:
      node-version: '20'
      environment: 'production'
      run-tests: true
    secrets:
      npm-token: ${{ secrets.NPM_TOKEN }}
      deploy-key: ${{ secrets.PROD_DEPLOY_KEY }}

  deploy:
    needs: [build-staging, build-production]
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        run: echo "Deploying artifacts"
```

### Q19: How do you implement composite actions in GitHub Actions?

**Answer:**

**Composite Action (actions/setup-environment/action.yml):**

```yaml
name: 'Setup Build Environment'
description: 'Sets up Node.js, caches dependencies, and installs packages'

inputs:
  node-version:
    description: 'Node.js version'
    required: true
    default: '18'
  cache-key:
    description: 'Cache key prefix'
    required: false
    default: 'npm'
  working-directory:
    description: 'Working directory'
    required: false
    default: '.'

outputs:
  cache-hit:
    description: 'Whether cache was hit'
    value: ${{ steps.cache.outputs.cache-hit }}
  node-version:
    description: 'Installed Node.js version'
    value: ${{ steps.setup-node.outputs.node-version }}

runs:
  using: "composite"
  steps:
    - name: Setup Node.js
      id: setup-node
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}

    - name: Cache dependencies
      id: cache
      uses: actions/cache@v3
      with:
        path: |
          ${{ inputs.working-directory }}/node_modules
          ~/.npm
        key: ${{ inputs.cache-key }}-${{ runner.os }}-node-${{ hashFiles(format('{0}/package-lock.json', inputs.working-directory)) }}
        restore-keys: |
          ${{ inputs.cache-key }}-${{ runner.os }}-node-

    - name: Install dependencies
      if: steps.cache.outputs.cache-hit != 'true'
      shell: bash
      working-directory: ${{ inputs.working-directory }}
      run: npm ci

    - name: Display versions
      shell: bash
      run: |
        echo "Node version: $(node --version)"
        echo "NPM version: $(npm --version)"
```

**Using Composite Action:**

```yaml
name: Build with Composite Action

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup environment
        uses: ./actions/setup-environment
        with:
          node-version: '18'
          cache-key: 'my-app'

      - name: Build
        run: npm run build

      - name: Test
        run: npm test
```

---

## Docker Integration

### Q20: How do you build optimized Docker images in CI/CD?

**Answer:**

**Multi-stage Dockerfile:**

```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production && \
    npm cache clean --force

# Copy source code
COPY . .

# Build application
RUN npm run build

# Test stage
FROM builder AS tester

RUN npm ci
COPY . .
RUN npm test

# Production stage
FROM node:18-alpine AS production

# Security: Run as non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copy only necessary files from builder
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/package*.json ./

# Use non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node healthcheck.js

# Start application
CMD ["node", "dist/server.js"]
```

**CI/CD Pipeline for Docker:**

```yaml
# GitLab CI
docker_build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  variables:
    DOCKER_BUILDKIT: 1
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    # Build with BuildKit for better caching
    - |
      docker build \
        --target production \
        --cache-from $CI_REGISTRY_IMAGE:latest \
        --build-arg BUILDKIT_INLINE_CACHE=1 \
        --tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA \
        --tag $CI_REGISTRY_IMAGE:latest \
        .

    # Run security scan
    - docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
        aquasec/trivy image --severity HIGH,CRITICAL \
        $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

    # Push images
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main
    - develop
```

### Q21: How do you implement Docker layer caching in CI/CD?

**Answer:**

**GitHub Actions with BuildKit:**

```yaml
name: Docker Build with Cache

on: [push]

jobs:
  docker:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            myapp:latest
            myapp:${{ github.sha }}
          # GitHub Actions cache
          cache-from: type=gha
          cache-to: type=gha,mode=max
          # Registry cache
          cache-from: type=registry,ref=myapp:buildcache
          cache-to: type=registry,ref=myapp:buildcache,mode=max
```

**Jenkins with Docker:**

```groovy
pipeline {
    agent any

    environment {
        DOCKER_BUILDKIT = '1'
        REGISTRY = 'docker.io'
        IMAGE_NAME = 'myapp'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    // Pull previous image for cache
                    sh """
                        docker pull ${REGISTRY}/${IMAGE_NAME}:latest || true
                    """

                    // Build with cache
                    sh """
                        docker build \
                          --cache-from ${REGISTRY}/${IMAGE_NAME}:latest \
                          --build-arg BUILDKIT_INLINE_CACHE=1 \
                          -t ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} \
                          -t ${REGISTRY}/${IMAGE_NAME}:latest \
                          .
                    """
                }
            }
        }

        stage('Test Image') {
            steps {
                sh """
                    docker run --rm ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} \
                      npm test
                """
            }
        }

        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry('', 'dockerhub-credentials') {
                        sh """
                            docker push ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}
                            docker push ${REGISTRY}/${IMAGE_NAME}:latest
                        """
                    }
                }
            }
        }
    }
}
```

---

## Kubernetes Deployment

### Q22: How do you implement Kubernetes deployment in CI/CD?

**Answer:**

**Kubernetes Manifests:**

**deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
  labels:
    app: myapp
    version: v1
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        version: v1
    spec:
      containers:
      - name: myapp
        image: myapp:latest  # Updated by CI/CD
        ports:
        - containerPort: 3000
          name: http
        env:
        - name: NODE_ENV
          value: "production"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: production
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 3000
  type: LoadBalancer
```

**GitLab CI Kubernetes Deployment:**

```yaml
# .gitlab-ci.yml
stages:
  - build
  - deploy

variables:
  KUBE_NAMESPACE: production
  APP_NAME: myapp
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker build -t $DOCKER_IMAGE .
    - docker push $DOCKER_IMAGE

deploy_k8s:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    # Create namespace if not exists
    - kubectl create namespace $KUBE_NAMESPACE --dry-run=client -o yaml | kubectl apply -f -

    # Update deployment image
    - kubectl set image deployment/$APP_NAME
        $APP_NAME=$DOCKER_IMAGE
        -n $KUBE_NAMESPACE

    # Wait for rollout
    - kubectl rollout status deployment/$APP_NAME -n $KUBE_NAMESPACE

    # Verify deployment
    - kubectl get pods -n $KUBE_NAMESPACE -l app=$APP_NAME
  environment:
    name: production
    kubernetes:
      namespace: $KUBE_NAMESPACE
  only:
    - main
```

**GitHub Actions Kubernetes Deployment:**

```yaml
name: Deploy to Kubernetes

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Configure kubectl
        uses: azure/k8s-set-context@v3
        with:
          method: kubeconfig
          kubeconfig: ${{ secrets.KUBE_CONFIG }}

      - name: Create namespace
        run: |
          kubectl create namespace production --dry-run=client -o yaml | kubectl apply -f -

      - name: Deploy to Kubernetes
        uses: azure/k8s-deploy@v4
        with:
          namespace: production
          manifests: |
            k8s/deployment.yaml
            k8s/service.yaml
          images: |
            myapp:${{ github.sha }}
          strategy: rolling

      - name: Verify deployment
        run: |
          kubectl rollout status deployment/myapp -n production
          kubectl get services -n production
```

### Q23: How do you implement Helm charts in CI/CD?

**Answer:**

**Helm Chart Structure:**

```
myapp/
├── Chart.yaml
├── values.yaml
├── values-prod.yaml
├── values-staging.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    ├── configmap.yaml
    └── secret.yaml
```

**Chart.yaml:**
```yaml
apiVersion: v2
name: myapp
description: My Application Helm Chart
type: application
version: 1.0.0
appVersion: "1.0.0"
```

**values.yaml:**
```yaml
replicaCount: 3

image:
  repository: myapp
  pullPolicy: IfNotPresent
  tag: "latest"

service:
  type: ClusterIP
  port: 80
  targetPort: 3000

ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: myapp-tls
      hosts:
        - myapp.example.com

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
```

**templates/deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "myapp.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: {{ .Values.service.targetPort }}
        resources:
          {{- toYaml .Values.resources | nindent 12 }}
```

**CI/CD with Helm (GitLab CI):**

```yaml
# .gitlab-ci.yml
stages:
  - build
  - deploy

variables:
  HELM_CHART_PATH: ./helm/myapp
  RELEASE_NAME: myapp

build_image:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

deploy_staging:
  stage: deploy
  image: alpine/helm:latest
  script:
    - helm upgrade --install $RELEASE_NAME $HELM_CHART_PATH
        --namespace staging
        --create-namespace
        --values $HELM_CHART_PATH/values-staging.yaml
        --set image.tag=$CI_COMMIT_SHA
        --wait
        --timeout 5m
  environment:
    name: staging
  only:
    - develop

deploy_production:
  stage: deploy
  image: alpine/helm:latest
  script:
    - helm upgrade --install $RELEASE_NAME $HELM_CHART_PATH
        --namespace production
        --create-namespace
        --values $HELM_CHART_PATH/values-prod.yaml
        --set image.tag=$CI_COMMIT_SHA
        --wait
        --timeout 10m

    # Run smoke tests
    - helm test $RELEASE_NAME -n production
  environment:
    name: production
  when: manual
  only:
    - main
```

**Jenkins with Helm:**

```groovy
pipeline {
    agent any

    environment {
        HELM_CHART = './helm/myapp'
        RELEASE_NAME = 'myapp'
        NAMESPACE = 'production'
    }

    stages {
        stage('Lint Helm Chart') {
            steps {
                sh 'helm lint ${HELM_CHART}'
            }
        }

        stage('Deploy with Helm') {
            steps {
                script {
                    sh """
                        helm upgrade --install ${RELEASE_NAME} ${HELM_CHART} \
                          --namespace ${NAMESPACE} \
                          --create-namespace \
                          --set image.tag=${BUILD_NUMBER} \
                          --values ${HELM_CHART}/values-prod.yaml \
                          --wait \
                          --timeout 10m
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh """
                    helm status ${RELEASE_NAME} -n ${NAMESPACE}
                    helm test ${RELEASE_NAME} -n ${NAMESPACE}
                """
            }
        }
    }

    post {
        failure {
            sh "helm rollback ${RELEASE_NAME} -n ${NAMESPACE}"
        }
    }
}
```

---

## Testing Automation

### Q24: How do you implement different types of testing in CI/CD?

**Answer:**

**Complete Testing Pipeline:**

```yaml
# .gitlab-ci.yml
stages:
  - test:unit
  - test:integration
  - test:e2e
  - test:performance
  - test:security

variables:
  POSTGRES_DB: testdb
  POSTGRES_USER: testuser
  POSTGRES_PASSWORD: testpass

# Unit Tests
unit_tests:
  stage: test:unit
  image: node:18
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
  script:
    - npm ci
    - npm run test:unit -- --coverage --ci
  coverage: '/Lines\s*:\s*(\d+\.?\d*)%/'
  artifacts:
    reports:
      junit: junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
    paths:
      - coverage/
    expire_in: 1 week

# Integration Tests
integration_tests:
  stage: test:integration
  image: node:18
  services:
    - postgres:13
    - redis:6
  variables:
    DATABASE_URL: "postgresql://testuser:testpass@postgres:5432/testdb"
    REDIS_URL: "redis://redis:6379"
  script:
    - npm ci
    - npm run migrate:test
    - npm run test:integration
  artifacts:
    when: always
    reports:
      junit: integration-junit.xml

# E2E Tests
e2e_tests:
  stage: test:e2e
  image: cypress/included:12.0.0
  services:
    - name: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
      alias: app
  script:
    - npm ci
    - npm run test:e2e -- --config baseUrl=http://app:3000
  artifacts:
    when: always
    paths:
      - cypress/videos/
      - cypress/screenshots/
    expire_in: 1 week
  retry:
    max: 2
    when: runner_system_failure

# Performance Tests
performance_tests:
  stage: test:performance
  image: grafana/k6:latest
  script:
    - k6 run --out json=test-results.json tests/load-test.js
  artifacts:
    reports:
      load_performance: test-results.json
  only:
    - main
    - develop

# Security Tests
security_scan:
  stage: test:security
  image: node:18
  script:
    # Dependency scanning
    - npm audit --audit-level=moderate

    # SAST scanning
    - npm install -g @shiftleft/scan
    - scan --src . --type nodejs
  allow_failure: true
  artifacts:
    reports:
      sast: gl-sast-report.json
```

**E2E Test Example (Cypress):**

```javascript
// cypress/e2e/user-flow.cy.js
describe('User Authentication Flow', () => {
  beforeEach(() => {
    cy.visit('/');
  });

  it('should register a new user', () => {
    cy.get('[data-testid="register-button"]').click();
    cy.get('[data-testid="email-input"]').type('test@example.com');
    cy.get('[data-testid="password-input"]').type('SecurePass123!');
    cy.get('[data-testid="submit-button"]').click();

    cy.url().should('include', '/dashboard');
    cy.contains('Welcome').should('be.visible');
  });

  it('should login existing user', () => {
    cy.get('[data-testid="login-button"]').click();
    cy.get('[data-testid="email-input"]').type('existing@example.com');
    cy.get('[data-testid="password-input"]').type('password123');
    cy.get('[data-testid="submit-button"]').click();

    cy.url().should('include', '/dashboard');
  });
});
```

**Performance Test (K6):**

```javascript
// tests/load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate } from 'k6/metrics';

const errorRate = new Rate('errors');

export const options = {
  stages: [
    { duration: '2m', target: 100 }, // Ramp up
    { duration: '5m', target: 100 }, // Stay at 100 users
    { duration: '2m', target: 200 }, // Spike to 200 users
    { duration: '5m', target: 200 }, // Stay at 200 users
    { duration: '2m', target: 0 },   // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% requests under 500ms
    errors: ['rate<0.1'],             // Error rate below 10%
  },
};

export default function () {
  const res = http.get('https://api.example.com/users');

  const result = check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });

  errorRate.add(!result);
  sleep(1);
}
```

### Q25: How do you implement test parallelization in CI/CD?

**Answer:**

**GitHub Actions Matrix Strategy:**

```yaml
name: Parallel Testing

on: [push]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [16, 18, 20]
        test-suite: [unit, integration, e2e]
        exclude:
          # Skip E2E on Windows
          - os: windows-latest
            test-suite: e2e

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - name: Install dependencies
        run: npm ci

      - name: Run ${{ matrix.test-suite }} tests
        run: npm run test:${{ matrix.test-suite }}
        env:
          CI: true
```

**GitLab CI Parallel Tests:**

```yaml
# .gitlab-ci.yml
test:unit:
  stage: test
  parallel: 5
  script:
    - npm ci
    - npm run test:unit -- --shard=${CI_NODE_INDEX}/${CI_NODE_TOTAL}
  artifacts:
    reports:
      junit: junit-${CI_NODE_INDEX}.xml

test:integration:
  stage: test
  parallel:
    matrix:
      - DATABASE: [postgres, mysql, mongodb]
        VERSION: ['13', '14', '15']
  services:
    - name: $DATABASE:$VERSION
  script:
    - npm run test:integration
  variables:
    DB_TYPE: $DATABASE
    DB_VERSION: $VERSION
```

**Jenkins Parallel Stages:**

```groovy
pipeline {
    agent any

    stages {
        stage('Parallel Tests') {
            parallel {
                stage('Unit Tests') {
                    agent { label 'test-runner' }
                    steps {
                        sh 'npm run test:unit'
                    }
                }

                stage('Integration Tests') {
                    agent { label 'test-runner' }
                    steps {
                        sh 'npm run test:integration'
                    }
                }

                stage('E2E Tests - Chrome') {
                    agent { label 'e2e-runner' }
                    steps {
                        sh 'npm run test:e2e -- --browser=chrome'
                    }
                }

                stage('E2E Tests - Firefox') {
                    agent { label 'e2e-runner' }
                    steps {
                        sh 'npm run test:e2e -- --browser=firefox'
                    }
                }

                stage('Security Scan') {
                    agent { label 'security' }
                    steps {
                        sh 'npm audit'
                        sh 'trivy fs .'
                    }
                }
            }
        }
    }
}
```

---

## Code Quality (SonarQube)

### Q26: How do you integrate SonarQube in CI/CD pipeline?

**Answer:**

**SonarQube Configuration:**

**sonar-project.properties:**
```properties
sonar.projectKey=myapp
sonar.projectName=My Application
sonar.projectVersion=1.0
sonar.sources=src
sonar.tests=tests
sonar.sourceEncoding=UTF-8

# Code coverage
sonar.javascript.lcov.reportPaths=coverage/lcov.info
sonar.coverage.exclusions=**/*.test.js,**/*.spec.js

# Test execution
sonar.testExecutionReportPaths=test-report.xml

# Quality gates
sonar.qualitygate.wait=true
sonar.qualitygate.timeout=300
```

**GitLab CI with SonarQube:**

```yaml
# .gitlab-ci.yml
stages:
  - test
  - analyze
  - quality-gate

variables:
  SONAR_HOST_URL: "https://sonarqube.example.com"
  SONAR_PROJECT_KEY: "myapp"

test:
  stage: test
  image: node:18
  script:
    - npm ci
    - npm run test -- --coverage --testResultsProcessor=jest-sonar-reporter
  artifacts:
    paths:
      - coverage/
      - test-report.xml
    expire_in: 1 week

sonarqube_scan:
  stage: analyze
  image: sonarsource/sonar-scanner-cli:latest
  dependencies:
    - test
  script:
    - sonar-scanner
        -Dsonar.projectKey=$SONAR_PROJECT_KEY
        -Dsonar.sources=src
        -Dsonar.host.url=$SONAR_HOST_URL
        -Dsonar.login=$SONAR_TOKEN
        -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
  allow_failure: false
  only:
    - main
    - develop
    - merge_requests

quality_gate:
  stage: quality-gate
  image: curlimages/curl:latest
  script:
    - |
      # Wait for quality gate result
      status="PENDING"
      timeout=300
      elapsed=0

      while [ "$status" = "PENDING" ] && [ $elapsed -lt $timeout ]; do
        sleep 5
        elapsed=$((elapsed + 5))

        response=$(curl -s -u $SONAR_TOKEN: \
          "$SONAR_HOST_URL/api/qualitygates/project_status?projectKey=$SONAR_PROJECT_KEY")

        status=$(echo $response | jq -r '.projectStatus.status')
        echo "Quality Gate Status: $status"
      done

      if [ "$status" != "OK" ]; then
        echo "Quality Gate Failed!"
        exit 1
      fi
  dependencies:
    - sonarqube_scan
```

**GitHub Actions with SonarQube:**

```yaml
name: SonarQube Analysis

on:
  push:
    branches: [main, develop]
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  sonarqube:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for better analysis

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Run tests with coverage
        run: npm run test -- --coverage

      - name: SonarQube Scan
        uses: sonarsource/sonarqube-scan-action@master
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
        with:
          args: >
            -Dsonar.projectKey=myapp
            -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
            -Dsonar.sources=src
            -Dsonar.tests=tests

      - name: SonarQube Quality Gate Check
        uses: sonarsource/sonarqube-quality-gate-action@master
        timeout-minutes: 5
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

**Jenkins with SonarQube:**

```groovy
pipeline {
    agent any

    environment {
        SONAR_HOST = 'https://sonarqube.example.com'
        SCANNER_HOME = tool 'SonarQube Scanner'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                sh 'npm ci'
                sh 'npm run test -- --coverage'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=myapp \
                        -Dsonar.sources=src \
                        -Dsonar.tests=tests \
                        -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
                        -Dsonar.coverage.exclusions=**/*.test.js
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        always {
            publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'coverage/lcov-report',
                reportFiles: 'index.html',
                reportName: 'Coverage Report'
            ])
        }
    }
}
```

### Q27: What are SonarQube Quality Gates and how do you configure them?

**Answer:**

**Quality Gate Definition:**

Quality Gates are a set of threshold measures that a project must meet before it can be released.

**Default Quality Gate Conditions:**
- New Code Coverage >= 80%
- Duplicated Lines on New Code <= 3%
- Maintainability Rating on New Code = A
- Reliability Rating on New Code = A
- Security Rating on New Code = A
- Security Hotspots Reviewed = 100%

**Custom Quality Gate (via SonarQube API):**

```bash
# Create quality gate
curl -X POST -u $SONAR_TOKEN: \
  "$SONAR_HOST/api/qualitygates/create?name=MyCustomGate"

# Add conditions
curl -X POST -u $SONAR_TOKEN: \
  "$SONAR_HOST/api/qualitygates/create_condition" \
  -d "gateId=1" \
  -d "metric=new_coverage" \
  -d "op=LT" \
  -d "error=80"

curl -X POST -u $SONAR_TOKEN: \
  "$SONAR_HOST/api/qualitygates/create_condition" \
  -d "gateId=1" \
  -d "metric=new_bugs" \
  -d "op=GT" \
  -d "error=0"

# Set as default
curl -X POST -u $SONAR_TOKEN: \
  "$SONAR_HOST/api/qualitygates/set_as_default?id=1"
```

**Quality Gate in Pipeline:**

```yaml
# .gitlab-ci.yml
check_quality_gate:
  stage: quality-gate
  image: curlimages/curl:latest
  script:
    - |
      # Get quality gate status
      qg_status=$(curl -s -u $SONAR_TOKEN: \
        "$SONAR_HOST/api/qualitygates/project_status?projectKey=$PROJECT_KEY" \
        | jq -r '.projectStatus.status')

      echo "Quality Gate Status: $qg_status"

      # Get detailed conditions
      curl -s -u $SONAR_TOKEN: \
        "$SONAR_HOST/api/qualitygates/project_status?projectKey=$PROJECT_KEY" \
        | jq '.projectStatus.conditions[] |
          "Metric: \(.metricKey), Status: \(.status), Value: \(.actualValue)"'

      if [ "$qg_status" != "OK" ]; then
        echo "Quality Gate Failed - Pipeline Aborted"
        exit 1
      fi
  only:
    - main
    - merge_requests
```

---

## Blue-Green Deployment

### Q28: Explain Blue-Green deployment and how to implement it in CI/CD.

**Answer:**

**Blue-Green Deployment Concept:**

Blue-Green deployment is a technique that reduces downtime and risk by running two identical production environments (Blue and Green). At any time, only one environment serves production traffic.

**Process:**
1. Blue = Current production
2. Deploy new version to Green
3. Test Green environment
4. Switch traffic from Blue to Green
5. Keep Blue as backup for rollback

**Kubernetes Implementation:**

**blue-deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
  labels:
    app: myapp
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: myapp
        image: myapp:v1.0
        ports:
        - containerPort: 3000
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
  labels:
    app: myapp
    version: green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: myapp
        image: myapp:v2.0
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
    version: blue  # Points to blue initially
  ports:
  - port: 80
    targetPort: 3000
  type: LoadBalancer
```

**CI/CD Pipeline (GitLab CI):**

```yaml
# .gitlab-ci.yml
stages:
  - build
  - deploy:green
  - test:green
  - switch:traffic
  - cleanup

variables:
  APP_NAME: myapp
  NAMESPACE: production
  BLUE_DEPLOYMENT: ${APP_NAME}-blue
  GREEN_DEPLOYMENT: ${APP_NAME}-green

build_image:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

deploy_green:
  stage: deploy:green
  image: bitnami/kubectl:latest
  script:
    # Get current active version
    - ACTIVE_VERSION=$(kubectl get service ${APP_NAME}-service -n ${NAMESPACE}
        -o jsonpath='{.spec.selector.version}')
    - echo "Current active version: $ACTIVE_VERSION"

    # Deploy to inactive version (green)
    - |
      if [ "$ACTIVE_VERSION" = "blue" ]; then
        TARGET_DEPLOYMENT=$GREEN_DEPLOYMENT
        TARGET_VERSION="green"
      else
        TARGET_DEPLOYMENT=$BLUE_DEPLOYMENT
        TARGET_VERSION="blue"
      fi

    - echo "Deploying to $TARGET_VERSION environment"

    # Update green deployment
    - kubectl set image deployment/$TARGET_DEPLOYMENT
        ${APP_NAME}=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
        -n ${NAMESPACE}

    # Wait for rollout
    - kubectl rollout status deployment/$TARGET_DEPLOYMENT -n ${NAMESPACE}

    - echo "TARGET_VERSION=$TARGET_VERSION" >> deploy.env
  artifacts:
    reports:
      dotenv: deploy.env

test_green:
  stage: test:green
  image: curlimages/curl:latest
  script:
    # Get green service endpoint
    - GREEN_ENDPOINT=$(kubectl get service ${APP_NAME}-${TARGET_VERSION}
        -n ${NAMESPACE} -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

    # Run smoke tests
    - |
      for i in {1..10}; do
        response=$(curl -s -o /dev/null -w "%{http_code}" http://$GREEN_ENDPOINT/health)
        if [ "$response" = "200" ]; then
          echo "Health check passed"
        else
          echo "Health check failed with status $response"
          exit 1
        fi
        sleep 2
      done
  dependencies:
    - deploy_green

switch_traffic:
  stage: switch:traffic
  image: bitnami/kubectl:latest
  script:
    - echo "Switching traffic to $TARGET_VERSION"

    # Update service selector to point to green
    - kubectl patch service ${APP_NAME}-service -n ${NAMESPACE}
        -p '{"spec":{"selector":{"version":"'${TARGET_VERSION}'"}}}'

    - echo "Traffic switched successfully to $TARGET_VERSION"
  when: manual
  dependencies:
    - test_green

cleanup_old:
  stage: cleanup
  image: bitnami/kubectl:latest
  script:
    # Scale down old deployment
    - |
      if [ "$TARGET_VERSION" = "green" ]; then
        kubectl scale deployment/$BLUE_DEPLOYMENT --replicas=0 -n ${NAMESPACE}
      else
        kubectl scale deployment/$GREEN_DEPLOYMENT --replicas=0 -n ${NAMESPACE}
      fi
  when: manual
  dependencies:
    - switch_traffic
```

**Rollback Script:**

```bash
#!/bin/bash
# rollback.sh

NAMESPACE="production"
SERVICE_NAME="myapp-service"
CURRENT_VERSION=$(kubectl get service $SERVICE_NAME -n $NAMESPACE \
  -o jsonpath='{.spec.selector.version}')

if [ "$CURRENT_VERSION" = "green" ]; then
  ROLLBACK_VERSION="blue"
else
  ROLLBACK_VERSION="green"
fi

echo "Rolling back from $CURRENT_VERSION to $ROLLBACK_VERSION"

# Scale up old version
kubectl scale deployment/myapp-$ROLLBACK_VERSION --replicas=3 -n $NAMESPACE

# Wait for pods to be ready
kubectl wait --for=condition=available --timeout=300s \
  deployment/myapp-$ROLLBACK_VERSION -n $NAMESPACE

# Switch traffic
kubectl patch service $SERVICE_NAME -n $NAMESPACE \
  -p '{"spec":{"selector":{"version":"'$ROLLBACK_VERSION'"}}}'

echo "Rollback completed successfully"
```

**GitHub Actions Blue-Green:**

```yaml
name: Blue-Green Deployment

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Configure kubectl
        uses: azure/k8s-set-context@v3
        with:
          kubeconfig: ${{ secrets.KUBE_CONFIG }}

      - name: Determine target environment
        id: target
        run: |
          ACTIVE=$(kubectl get svc myapp-service -n production \
            -o jsonpath='{.spec.selector.version}')

          if [ "$ACTIVE" = "blue" ]; then
            echo "target=green" >> $GITHUB_OUTPUT
          else
            echo "target=blue" >> $GITHUB_OUTPUT
          fi

      - name: Deploy to ${{ steps.target.outputs.target }}
        run: |
          kubectl set image deployment/myapp-${{ steps.target.outputs.target }} \
            myapp=myapp:${{ github.sha }} -n production

          kubectl rollout status deployment/myapp-${{ steps.target.outputs.target }} \
            -n production

      - name: Run smoke tests
        run: ./scripts/smoke-tests.sh ${{ steps.target.outputs.target }}

      - name: Switch traffic
        run: |
          kubectl patch svc myapp-service -n production \
            -p '{"spec":{"selector":{"version":"${{ steps.target.outputs.target }}"}}}'
```

---

## Canary Deployment

### Q29: Explain Canary deployment and how to implement it.

**Answer:**

**Canary Deployment Concept:**

Canary deployment is a pattern for rolling out releases to a subset of users or servers gradually. The idea is to test the new version on a small percentage of traffic before rolling it out to everyone.

**Process:**
1. Deploy new version to small subset (e.g., 10%)
2. Monitor metrics and errors
3. Gradually increase traffic to canary
4. If successful, route 100% traffic to new version
5. If issues found, rollback immediately

**Kubernetes with Istio:**

**deployment.yaml:**
```yaml
# Stable version
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-stable
spec:
  replicas: 9
  selector:
    matchLabels:
      app: myapp
      version: stable
  template:
    metadata:
      labels:
        app: myapp
        version: stable
    spec:
      containers:
      - name: myapp
        image: myapp:v1.0
        ports:
        - containerPort: 3000
---
# Canary version
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
      version: canary
  template:
    metadata:
      labels:
        app: myapp
        version: canary
    spec:
      containers:
      - name: myapp
        image: myapp:v2.0
        ports:
        - containerPort: 3000
---
# Service
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 3000
```

**Istio Virtual Service (Traffic Splitting):**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp
  http:
  - match:
    - headers:
        user-agent:
          regex: ".*Mobile.*"
    route:
    - destination:
        host: myapp
        subset: canary
      weight: 50
    - destination:
        host: myapp
        subset: stable
      weight: 50
  - route:
    - destination:
        host: myapp
        subset: stable
      weight: 90
    - destination:
        host: myapp
        subset: canary
      weight: 10
---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: myapp
spec:
  host: myapp
  subsets:
  - name: stable
    labels:
      version: stable
  - name: canary
    labels:
      version: canary
```

**GitLab CI Canary Deployment:**

```yaml
# .gitlab-ci.yml
stages:
  - build
  - deploy:canary
  - monitor
  - promote
  - rollback

variables:
  APP_NAME: myapp
  NAMESPACE: production
  CANARY_WEIGHT: 10

build:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

deploy_canary:
  stage: deploy:canary
  image: bitnami/kubectl:latest
  script:
    # Deploy canary version
    - kubectl set image deployment/${APP_NAME}-canary
        ${APP_NAME}=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
        -n ${NAMESPACE}

    # Wait for rollout
    - kubectl rollout status deployment/${APP_NAME}-canary -n ${NAMESPACE}

    # Update Istio VirtualService to route 10% traffic to canary
    - |
      cat <<EOF | kubectl apply -f -
      apiVersion: networking.istio.io/v1beta1
      kind: VirtualService
      metadata:
        name: ${APP_NAME}
        namespace: ${NAMESPACE}
      spec:
        hosts:
        - ${APP_NAME}
        http:
        - route:
          - destination:
              host: ${APP_NAME}
              subset: stable
            weight: 90
          - destination:
              host: ${APP_NAME}
              subset: canary
            weight: 10
      EOF
  environment:
    name: production-canary

monitor_canary:
  stage: monitor
  image: curlimages/curl:latest
  script:
    # Monitor for 10 minutes
    - |
      echo "Monitoring canary deployment..."

      # Query Prometheus for error rate
      ERROR_RATE=$(curl -s "http://prometheus:9090/api/v1/query" \
        --data-urlencode 'query=rate(http_requests_total{status=~"5..",version="canary"}[5m])' \
        | jq -r '.data.result[0].value[1]')

      # Check if error rate is acceptable
      if (( $(echo "$ERROR_RATE > 0.01" | bc -l) )); then
        echo "Error rate too high: $ERROR_RATE"
        exit 1
      fi

      # Check response time (p99)
      P99_LATENCY=$(curl -s "http://prometheus:9090/api/v1/query" \
        --data-urlencode 'query=histogram_quantile(0.99,rate(http_request_duration_seconds_bucket{version="canary"}[5m]))' \
        | jq -r '.data.result[0].value[1]')

      if (( $(echo "$P99_LATENCY > 1.0" | bc -l) )); then
        echo "P99 latency too high: $P99_LATENCY"
        exit 1
      fi

      echo "Canary metrics are healthy"
  dependencies:
    - deploy_canary

promote_canary:
  stage: promote
  image: bitnami/kubectl:latest
  script:
    - echo "Promoting canary to stable"

    # Gradually increase canary traffic
    - |
      for weight in 25 50 75 100; do
        echo "Routing ${weight}% traffic to canary"

        stable_weight=$((100 - weight))

        cat <<EOF | kubectl apply -f -
        apiVersion: networking.istio.io/v1beta1
        kind: VirtualService
        metadata:
          name: ${APP_NAME}
          namespace: ${NAMESPACE}
        spec:
          hosts:
          - ${APP_NAME}
          http:
          - route:
            - destination:
                host: ${APP_NAME}
                subset: stable
              weight: ${stable_weight}
            - destination:
                host: ${APP_NAME}
                subset: canary
              weight: ${weight}
        EOF

        sleep 120  # Wait 2 minutes between increments
      done

    # Update stable deployment with new version
    - kubectl set image deployment/${APP_NAME}-stable
        ${APP_NAME}=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
        -n ${NAMESPACE}

    # Revert traffic to stable only
    - |
      cat <<EOF | kubectl apply -f -
      apiVersion: networking.istio.io/v1beta1
      kind: VirtualService
      metadata:
        name: ${APP_NAME}
        namespace: ${NAMESPACE}
      spec:
        hosts:
        - ${APP_NAME}
        http:
        - route:
          - destination:
              host: ${APP_NAME}
              subset: stable
            weight: 100
      EOF
  when: manual
  dependencies:
    - monitor_canary

rollback_canary:
  stage: rollback
  image: bitnami/kubectl:latest
  script:
    - echo "Rolling back canary deployment"

    # Revert traffic to stable
    - |
      cat <<EOF | kubectl apply -f -
      apiVersion: networking.istio.io/v1beta1
      kind: VirtualService
      metadata:
        name: ${APP_NAME}
        namespace: ${NAMESPACE}
      spec:
        hosts:
        - ${APP_NAME}
        http:
        - route:
          - destination:
              host: ${APP_NAME}
              subset: stable
            weight: 100
      EOF

    - echo "Canary rollback completed"
  when: manual
  dependencies:
    - deploy_canary
```

**Automated Canary with Flagger:**

```yaml
# flagger-canary.yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: myapp
  namespace: production
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  service:
    port: 80
    targetPort: 3000
  analysis:
    interval: 1m
    threshold: 10
    maxWeight: 50
    stepWeight: 10
    metrics:
    - name: request-success-rate
      thresholdRange:
        min: 99
      interval: 1m
    - name: request-duration
      thresholdRange:
        max: 500
      interval: 1m
    webhooks:
    - name: load-test
      url: http://flagger-loadtester/
      timeout: 5s
      metadata:
        cmd: "hey -z 1m -q 10 -c 2 http://myapp-canary:80/"
  progressDeadlineSeconds: 600
```

**Jenkins Canary Pipeline:**

```groovy
pipeline {
    agent any

    parameters {
        choice(name: 'CANARY_PERCENTAGE',
               choices: ['10', '25', '50', '75', '100'],
               description: 'Percentage of traffic to canary')
    }

    stages {
        stage('Deploy Canary') {
            steps {
                script {
                    sh """
                        kubectl set image deployment/myapp-canary \
                          myapp=myapp:${BUILD_NUMBER} -n production

                        kubectl rollout status deployment/myapp-canary -n production
                    """
                }
            }
        }

        stage('Route Traffic to Canary') {
            steps {
                script {
                    def stableWeight = 100 - params.CANARY_PERCENTAGE.toInteger()

                    sh """
                        kubectl apply -f - <<EOF
                        apiVersion: networking.istio.io/v1beta1
                        kind: VirtualService
                        metadata:
                          name: myapp
                          namespace: production
                        spec:
                          hosts:
                          - myapp
                          http:
                          - route:
                            - destination:
                                host: myapp
                                subset: stable
                              weight: ${stableWeight}
                            - destination:
                                host: myapp
                                subset: canary
                              weight: ${params.CANARY_PERCENTAGE}
                        EOF
                    """
                }
            }
        }

        stage('Monitor Canary') {
            steps {
                script {
                    timeout(time: 10, unit: 'MINUTES') {
                        // Monitor metrics
                        sh './scripts/monitor-canary.sh'
                    }
                }
            }
        }

        stage('Promote or Rollback') {
            steps {
                input message: 'Promote canary to production?',
                      ok: 'Promote',
                      submitter: 'admin'

                script {
                    sh """
                        kubectl set image deployment/myapp-stable \
                          myapp=myapp:${BUILD_NUMBER} -n production

                        # Route all traffic back to stable
                        kubectl patch virtualservice myapp -n production --type merge \
                          -p '{"spec":{"http":[{"route":[{"destination":{"host":"myapp","subset":"stable"},"weight":100}]}]}}'
                    """
                }
            }
        }
    }

    post {
        failure {
            sh """
                # Rollback: Route all traffic to stable
                kubectl patch virtualservice myapp -n production --type merge \
                  -p '{"spec":{"http":[{"route":[{"destination":{"host":"myapp","subset":"stable"},"weight":100}]}]}}'
            """
        }
    }
}
```

---

## Real-World Pipeline Examples

### Q30: Provide a complete real-world microservices CI/CD pipeline.

**Answer:**

**Microservices Architecture Pipeline:**

**Project Structure:**
```
microservices-app/
├── services/
│   ├── user-service/
│   ├── product-service/
│   ├── order-service/
│   └── notification-service/
├── .gitlab-ci.yml
├── docker-compose.yml
└── k8s/
    ├── base/
    └── overlays/
```

**Root .gitlab-ci.yml:**

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - package
  - deploy:staging
  - test:e2e
  - deploy:production

variables:
  DOCKER_REGISTRY: registry.gitlab.com
  K8S_NAMESPACE_STAGING: staging
  K8S_NAMESPACE_PROD: production

# Shared templates
.build_template: &build_template
  image: node:18
  cache:
    key: ${CI_COMMIT_REF_SLUG}-${SERVICE_NAME}
    paths:
      - ${SERVICE_PATH}/node_modules/
  before_script:
    - cd ${SERVICE_PATH}
    - npm ci
  script:
    - npm run build
  artifacts:
    paths:
      - ${SERVICE_PATH}/dist/
    expire_in: 1 hour

.test_template: &test_template
  image: node:18
  cache:
    key: ${CI_COMMIT_REF_SLUG}-${SERVICE_NAME}
    paths:
      - ${SERVICE_PATH}/node_modules/
  before_script:
    - cd ${SERVICE_PATH}
  script:
    - npm run test -- --coverage
  coverage: '/Lines\s*:\s*(\d+\.?\d*)%/'
  artifacts:
    reports:
      junit: ${SERVICE_PATH}/junit.xml
      coverage_report:
        coverage_format: cobertura
        path: ${SERVICE_PATH}/coverage/cobertura-coverage.xml

.docker_build_template: &docker_build
  image: docker:24
  services:
    - docker:24-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - cd ${SERVICE_PATH}
    - |
      docker build \
        --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
        --build-arg VCS_REF=$CI_COMMIT_SHA \
        --tag $CI_REGISTRY_IMAGE/${SERVICE_NAME}:$CI_COMMIT_SHA \
        --tag $CI_REGISTRY_IMAGE/${SERVICE_NAME}:latest \
        .
    - docker push $CI_REGISTRY_IMAGE/${SERVICE_NAME}:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE/${SERVICE_NAME}:latest

# User Service Jobs
build:user-service:
  stage: build
  variables:
    SERVICE_NAME: user-service
    SERVICE_PATH: services/user-service
  <<: *build_template

test:user-service:
  stage: test
  variables:
    SERVICE_NAME: user-service
    SERVICE_PATH: services/user-service
  <<: *test_template
  dependencies:
    - build:user-service

package:user-service:
  stage: package
  variables:
    SERVICE_NAME: user-service
    SERVICE_PATH: services/user-service
  <<: *docker_build
  dependencies:
    - build:user-service
  only:
    changes:
      - services/user-service/**/*
      - .gitlab-ci.yml

# Product Service Jobs
build:product-service:
  stage: build
  variables:
    SERVICE_NAME: product-service
    SERVICE_PATH: services/product-service
  <<: *build_template

test:product-service:
  stage: test
  variables:
    SERVICE_NAME: product-service
    SERVICE_PATH: services/product-service
  <<: *test_template
  dependencies:
    - build:product-service

package:product-service:
  stage: package
  variables:
    SERVICE_NAME: product-service
    SERVICE_PATH: services/product-service
  <<: *docker_build
  dependencies:
    - build:product-service
  only:
    changes:
      - services/product-service/**/*
      - .gitlab-ci.yml

# Order Service Jobs
build:order-service:
  stage: build
  variables:
    SERVICE_NAME: order-service
    SERVICE_PATH: services/order-service
  <<: *build_template

test:order-service:
  stage: test
  variables:
    SERVICE_NAME: order-service
    SERVICE_PATH: services/order-service
  <<: *test_template
  dependencies:
    - build:order-service

package:order-service:
  stage: package
  variables:
    SERVICE_NAME: order-service
    SERVICE_PATH: services/order-service
  <<: *docker_build
  dependencies:
    - build:order-service
  only:
    changes:
      - services/order-service/**/*
      - .gitlab-ci.yml

# Deploy to Staging
deploy:staging:
  stage: deploy:staging
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context staging

    # Deploy each service
    - |
      for service in user-service product-service order-service; do
        if kubectl get deployment $service -n $K8S_NAMESPACE_STAGING > /dev/null 2>&1; then
          kubectl set image deployment/$service \
            $service=$CI_REGISTRY_IMAGE/$service:$CI_COMMIT_SHA \
            -n $K8S_NAMESPACE_STAGING
        else
          kubectl apply -f k8s/overlays/staging/$service/ -n $K8S_NAMESPACE_STAGING
        fi

        kubectl rollout status deployment/$service -n $K8S_NAMESPACE_STAGING
      done
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

# E2E Tests
test:e2e:
  stage: test:e2e
  image: cypress/included:12.0.0
  services:
    - name: postgres:13
      alias: postgres
    - name: redis:6
      alias: redis
  variables:
    DATABASE_URL: "postgresql://testuser:testpass@postgres:5432/testdb"
    REDIS_URL: "redis://redis:6379"
  script:
    - npm ci
    - npm run test:e2e -- --config baseUrl=https://staging.example.com
  artifacts:
    when: always
    paths:
      - cypress/videos/
      - cypress/screenshots/
  dependencies:
    - deploy:staging
  only:
    - develop

# Deploy to Production
deploy:production:
  stage: deploy:production
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context production

    # Canary deployment for each service
    - |
      for service in user-service product-service order-service; do
        # Deploy canary
        kubectl set image deployment/$service-canary \
          $service=$CI_REGISTRY_IMAGE/$service:$CI_COMMIT_SHA \
          -n $K8S_NAMESPACE_PROD

        kubectl rollout status deployment/$service-canary -n $K8S_NAMESPACE_PROD

        # Update traffic split (10% canary)
        kubectl apply -f - <<EOF
        apiVersion: networking.istio.io/v1beta1
        kind: VirtualService
        metadata:
          name: $service
          namespace: $K8S_NAMESPACE_PROD
        spec:
          hosts:
          - $service
          http:
          - route:
            - destination:
                host: $service
                subset: stable
              weight: 90
            - destination:
                host: $service
                subset: canary
              weight: 10
        EOF
      done

    # Wait and monitor (would integrate with monitoring tools)
    - sleep 300

    # Promote canary to stable
    - |
      for service in user-service product-service order-service; do
        kubectl set image deployment/$service-stable \
          $service=$CI_REGISTRY_IMAGE/$service:$CI_COMMIT_SHA \
          -n $K8S_NAMESPACE_PROD

        kubectl rollout status deployment/$service-stable -n $K8S_NAMESPACE_PROD

        # Route all traffic to stable
        kubectl patch virtualservice $service -n $K8S_NAMESPACE_PROD --type merge \
          -p '{"spec":{"http":[{"route":[{"destination":{"host":"'$service'","subset":"stable"},"weight":100}]}]}}'
      done
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - main
```

### Q31: Provide a complete Node.js/React fullstack application pipeline.

**Answer:**

**GitHub Actions Full-Stack Pipeline:**

```yaml
# .github/workflows/fullstack-pipeline.yml
name: Full-Stack CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '18'
  DOCKER_REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Frontend Jobs
  frontend-lint:
    name: Frontend Lint
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./frontend

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint

      - name: Run Prettier
        run: npm run format:check

  frontend-test:
    name: Frontend Tests
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./frontend

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test -- --coverage --watchAll=false

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./frontend/coverage/coverage-final.json
          flags: frontend

  frontend-build:
    name: Frontend Build
    runs-on: ubuntu-latest
    needs: [frontend-lint, frontend-test]
    defaults:
      run:
        working-directory: ./frontend

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build
        env:
          REACT_APP_API_URL: ${{ secrets.API_URL }}

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: frontend-build
          path: frontend/build/

  # Backend Jobs
  backend-lint:
    name: Backend Lint
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./backend

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          cache-dependency-path: backend/package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint

  backend-test:
    name: Backend Tests
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./backend

    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:6
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          cache-dependency-path: backend/package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: Run migrations
        run: npm run migrate
        env:
          DATABASE_URL: postgresql://testuser:testpass@localhost:5432/testdb

      - name: Run tests
        run: npm test -- --coverage
        env:
          DATABASE_URL: postgresql://testuser:testpass@localhost:5432/testdb
          REDIS_URL: redis://localhost:6379

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./backend/coverage/coverage-final.json
          flags: backend

  # Docker Build
  docker-build:
    name: Build Docker Images
    runs-on: ubuntu-latest
    needs: [frontend-build, backend-test]
    permissions:
      contents: read
      packages: write

    strategy:
      matrix:
        service: [frontend, backend]

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.DOCKER_REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}-${{ matrix.service }}
          tags: |
            type=ref,event=branch
            type=sha,prefix={{branch}}-
            type=semver,pattern={{version}}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: ./${{ matrix.service }}
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # E2E Tests
  e2e-tests:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: [docker-build]

    steps:
      - uses: actions/checkout@v4

      - name: Start services
        run: docker-compose -f docker-compose.test.yml up -d

      - name: Wait for services
        run: |
          timeout 60 bash -c 'until curl -f http://localhost:3000/health; do sleep 2; done'
          timeout 60 bash -c 'until curl -f http://localhost:4000/health; do sleep 2; done'

      - name: Run E2E tests
        run: |
          npm ci
          npm run test:e2e

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: e2e-results
          path: |
            cypress/videos/
            cypress/screenshots/

      - name: Stop services
        if: always()
        run: docker-compose -f docker-compose.test.yml down

  # Deploy to Staging
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: [e2e-tests]
    if: github.ref == 'refs/heads/develop'
    environment:
      name: staging
      url: https://staging.example.com

    steps:
      - uses: actions/checkout@v4

      - name: Configure kubectl
        uses: azure/k8s-set-context@v3
        with:
          kubeconfig: ${{ secrets.KUBE_CONFIG_STAGING }}

      - name: Deploy frontend
        run: |
          kubectl set image deployment/frontend \
            frontend=${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}-frontend:${{ github.sha }} \
            -n staging
          kubectl rollout status deployment/frontend -n staging

      - name: Deploy backend
        run: |
          kubectl set image deployment/backend \
            backend=${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}-backend:${{ github.sha }} \
            -n staging
          kubectl rollout status deployment/backend -n staging

      - name: Run smoke tests
        run: ./scripts/smoke-tests.sh staging

  # Deploy to Production
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: [e2e-tests]
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://example.com

    steps:
      - uses: actions/checkout@v4

      - name: Configure kubectl
        uses: azure/k8s-set-context@v3
        with:
          kubeconfig: ${{ secrets.KUBE_CONFIG_PROD }}

      - name: Deploy with Helm
        run: |
          helm upgrade --install fullstack-app ./helm/fullstack \
            --namespace production \
            --set frontend.image.tag=${{ github.sha }} \
            --set backend.image.tag=${{ github.sha }} \
            --set frontend.replicas=3 \
            --set backend.replicas=5 \
            --wait \
            --timeout 10m

      - name: Verify deployment
        run: |
          kubectl rollout status deployment/frontend -n production
          kubectl rollout status deployment/backend -n production

      - name: Run production smoke tests
        run: ./scripts/smoke-tests.sh production

  # Notify
  notify:
    name: Notify Team
    runs-on: ubuntu-latest
    needs: [deploy-staging, deploy-production]
    if: always()

    steps:
      - name: Slack Notification
        uses: slackapi/slack-github-action@v1
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK }}
          payload: |
            {
              "text": "Deployment ${{ job.status }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Deployment Status:* ${{ job.status }}\n*Branch:* ${{ github.ref_name }}\n*Commit:* ${{ github.sha }}\n*Author:* ${{ github.actor }}"
                  }
                }
              ]
            }
```

---

This comprehensive CI/CD documentation covers all the topics you requested with detailed explanations, real-world examples, and complete pipeline configurations. The documentation is designed to help both beginners and experienced professionals understand and implement CI/CD best practices.

I've created comprehensive CI/CD documentation at **E:\learn\English\CI-CD\** with two detailed files:

## 📁 Created Files:

### 1. **README.md** - Overview Document
Covers:
- CI/CD fundamentals and concepts
- Pipeline stages and workflows
- Popular CI/CD tools (Jenkins, GitLab CI, GitHub Actions, CircleCI, etc.)
- Deployment strategies (Blue-Green, Canary, Rolling)
- Infrastructure as Code
- Best practices and optimization techniques

### 2. **CI-CD-Interview-Questions.md** - Comprehensive Q&A Guide
Contains **31 detailed questions and answers** covering:

#### Core Topics:
- **CI/CD Fundamentals** (Q1-Q5): Basic concepts, Pipeline as Code, best practices
- **Pipeline Stages** (Q6-Q8): Stage structure, secrets management, test reliability
- **Jenkins** (Q9-Q12): Jenkinsfile types, complete examples, parallel execution, shared libraries
- **GitLab CI** (Q13-Q16): YAML configuration, Docker-in-Docker, templates, multi-project pipelines
- **GitHub Actions** (Q17-Q19): Workflow syntax, reusable workflows, composite actions
- **Docker Integration** (Q20-Q21): Multi-stage builds, layer caching, optimization
- **Kubernetes Deployment** (Q22-Q23): K8s manifests, Helm charts, deployment automation
- **Testing Automation** (Q24-Q25): Unit/Integration/E2E tests, test parallelization, performance testing
- **Code Quality** (Q26-Q27): SonarQube integration, quality gates, code analysis
- **Blue-Green Deployment** (Q28): Complete implementation with rollback strategies
- **Canary Deployment** (Q29): Gradual rollout with Istio, Flagger automation
- **Real-World Examples** (Q30-Q31): Complete microservices and full-stack pipelines

#### Special Features:
✅ **Complete pipeline configurations** for Jenkins, GitLab CI, and GitHub Actions
✅ **Production-ready examples** with Docker, Kubernetes, and Helm
✅ **Advanced deployment patterns** with traffic splitting and monitoring
✅ **Security best practices** including secret management and vulnerability scanning
✅ **Real-world microservices** and full-stack application pipelines
✅ **Monitoring and rollback strategies** for safe deployments

All examples include working code that can be adapted to your projects!