# CI/CD Pipelines Cheatsheet

## Quick Reference Guide for Platform Engineers

### Jenkins Pipeline Syntax (Declarative)

```groovy
// Jenkinsfile - Complete Example
pipeline {
    agent {
        label 'docker'  // Run on agent with 'docker' label
        // OR
        docker {
            image 'node:16'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    
    environment {
        APP_NAME = 'myapp'
        DOCKER_REGISTRY = 'registry.example.com'
        DEPLOY_ENV = "${env.BRANCH_NAME == 'main' ? 'production' : 'staging'}"
    }
    
    parameters {
        string(name: 'VERSION', defaultValue: '1.0.0', description: 'Version to deploy')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Target environment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
    }
    
    triggers {
        cron('H 2 * * *')              // Daily at 2 AM
        pollSCM('H/15 * * * *')         // Poll every 15 minutes
        githubPush()                    // On GitHub push
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
        timeout(time: 1, unit: 'HOURS')
        disableConcurrentBuilds()
        skipDefaultCheckout()
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        
        stage('Test') {
            when {
                expression { params.RUN_TESTS == true }
            }
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm run test:unit'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'npm run test:integration'
                    }
                }
                stage('Lint') {
                    steps {
                        sh 'npm run lint'
                    }
                }
            }
            post {
                always {
                    junit 'test-results/**/*.xml'
                    publishHTML([
                        reportDir: 'coverage',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                sh 'npm audit'
                sh 'trivy fs --severity HIGH,CRITICAL .'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_REGISTRY}/${APP_NAME}:${GIT_COMMIT_SHORT}")
                }
            }
        }
        
        stage('Push Image') {
            when {
                branch 'main'
            }
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-credentials') {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                sh """
                    kubectl set image deployment/${APP_NAME} \
                        ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${GIT_COMMIT_SHORT} \
                        -n ${DEPLOY_ENV}
                """
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            slackSend(
                color: 'good',
                message: "Build Successful: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "Build Failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
            )
            emailext(
                subject: "Build Failed: ${env.JOB_NAME}",
                body: "Build ${env.BUILD_NUMBER} failed. Check console output.",
                to: 'team@example.com'
            )
        }
    }
}
```

### Jenkins Scripted Pipeline

```groovy
// Scripted Pipeline - Flexibility
node('docker') {
    def dockerImage
    def commitHash
    
    try {
        stage('Checkout') {
            checkout scm
            commitHash = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
        }
        
        stage('Build') {
            sh 'npm install'
            sh 'npm run build'
        }
        
        stage('Test') {
            try {
                sh 'npm test'
            } finally {
                junit 'test-results/**/*.xml'
            }
        }
        
        stage('Docker Build') {
            dockerImage = docker.build("myapp:${commitHash}")
        }
        
        stage('Deploy') {
            if (env.BRANCH_NAME == 'main') {
                dockerImage.push()
                sh "kubectl apply -f k8s/"
            }
        }
        
        currentBuild.result = 'SUCCESS'
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        // Cleanup
        cleanWs()
    }
}
```

### GitHub Actions Workflow Syntax

```yaml
# .github/workflows/ci-cd.yml - Complete Example
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
    tags:
      - 'v*'
    paths-ignore:
      - '**.md'
      - 'docs/**'
  pull_request:
    branches: [main]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy to'
        required: true
        type: choice
        options:
          - dev
          - staging
          - production
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM UTC

env:
  NODE_VERSION: '16'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    name: Test
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [14, 16, 18]
        os: [ubuntu-latest, windows-latest]
      fail-fast: false
      max-parallel: 4
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0  # Full history for SonarQube
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
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
          files: ./coverage/lcov.info
          flags: unittests
          name: codecov-${{ matrix.node-version }}
      
      - name: Archive test results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: test-results-${{ matrix.node-version }}
          path: test-results/
          retention-days: 30

  security:
    name: Security Scan
    runs-on: ubuntu-latest
    
    permissions:
      security-events: write
      contents: read
    
    steps:
      - uses: actions/checkout@v3
      
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
      
      - name: NPM Audit
        run: npm audit --audit-level=moderate
        continue-on-error: true

  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [test, security]
    
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
      image-digest: ${{ steps.build.outputs.digest }}
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Log in to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha,prefix={{branch}}-
      
      - name: Build and push Docker image
        id: build
        uses: docker/build-push-action@v4
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          platforms: linux/amd64,linux/arm64

  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/develop'
    environment:
      name: staging
      url: https://staging.example.com
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure kubectl
        uses: azure/k8s-set-context@v3
        with:
          method: kubeconfig
          kubeconfig: ${{ secrets.KUBE_CONFIG_STAGING }}
      
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/myapp \
            myapp=${{ needs.build.outputs.image-tag }} \
            -n staging
          kubectl rollout status deployment/myapp -n staging

  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build
    if: startsWith(github.ref, 'refs/tags/v')
    environment:
      name: production
      url: https://example.com
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to production
        run: |
          echo "Deploying to production..."
          # Deployment commands here
      
      - name: Create release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          draft: false
          prerelease: false

  notify:
    name: Notify
    runs-on: ubuntu-latest
    needs: [deploy-staging, deploy-production]
    if: always()
    
    steps:
      - name: Send Slack notification
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Deployment completed'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### GitHub Actions Reusable Workflows

```yaml
# .github/workflows/reusable-build.yml
name: Reusable Build Workflow

on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string
      environment:
        required: true
        type: string
    secrets:
      NPM_TOKEN:
        required: true
    outputs:
      build-status:
        description: "Build status"
        value: ${{ jobs.build.outputs.status }}

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      status: ${{ steps.build.outcome }}
    
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci
      - run: npm run build
        id: build
        env:
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}

# .github/workflows/main.yml - Using reusable workflow
name: Main Pipeline

on: [push]

jobs:
  call-build:
    uses: ./.github/workflows/reusable-build.yml
    with:
      node-version: '16'
      environment: 'production'
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### GitLab CI/CD

```yaml
# .gitlab-ci.yml - Complete Example
variables:
  DOCKER_REGISTRY: registry.gitlab.com
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE
  DOCKER_TAG: $CI_COMMIT_SHORT_SHA

stages:
  - build
  - test
  - security
  - deploy

default:
  image: node:16
  before_script:
    - npm ci --cache .npm --prefer-offline
  cache:
    key:
      files:
        - package-lock.json
    paths:
      - .npm/
      - node_modules/

# Build Stage
build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week
  rules:
    - if: $CI_COMMIT_BRANCH
      exists:
        - package.json

# Test Stages
test:unit:
  stage: test
  script:
    - npm run test:unit
  coverage: '/Statements\s*:\s*(\d+\.?\d*)%/'
  artifacts:
    when: always
    reports:
      junit: test-results/junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

test:integration:
  stage: test
  services:
    - postgres:13
    - redis:6
  variables:
    POSTGRES_DB: testdb
    POSTGRES_USER: testuser
    POSTGRES_PASSWORD: testpass
    DATABASE_URL: "postgresql://testuser:testpass@postgres:5432/testdb"
    REDIS_URL: "redis://redis:6379"
  script:
    - npm run test:integration

test:e2e:
  stage: test
  image: cypress/browsers:node16.14.0-chrome99-ff97
  script:
    - npm run start:test &
    - npx wait-on http://localhost:3000
    - npm run test:e2e
  artifacts:
    when: always
    paths:
      - cypress/videos/
      - cypress/screenshots/
    expire_in: 1 week

# Security Stage
security:sast:
  stage: security
  image: returntocorp/semgrep
  script:
    - semgrep --config=auto --json --output=sast-report.json .
  artifacts:
    reports:
      sast: sast-report.json
  allow_failure: true

security:dependency:
  stage: security
  script:
    - npm audit --audit-level=moderate
  allow_failure: true

security:container:
  stage: security
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $DOCKER_IMAGE:$DOCKER_TAG .
    - docker run --rm -v /var/run/docker.sock:/var/run/docker.sock 
      aquasec/trivy image --severity HIGH,CRITICAL $DOCKER_IMAGE:$DOCKER_TAG

# Build Docker Image
docker:build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build --cache-from $DOCKER_IMAGE:latest -t $DOCKER_IMAGE:$DOCKER_TAG -t $DOCKER_IMAGE:latest .
    - docker push $DOCKER_IMAGE:$DOCKER_TAG
    - docker push $DOCKER_IMAGE:latest
  only:
    - main
    - develop

# Deploy Stages
deploy:staging:
  stage: deploy
  image: bitnami/kubectl:latest
  environment:
    name: staging
    url: https://staging.example.com
    on_stop: stop:staging
  script:
    - kubectl config use-context staging
    - kubectl set image deployment/myapp myapp=$DOCKER_IMAGE:$DOCKER_TAG -n staging
    - kubectl rollout status deployment/myapp -n staging
  only:
    - develop

stop:staging:
  stage: deploy
  image: bitnami/kubectl:latest
  environment:
    name: staging
    action: stop
  script:
    - kubectl delete deployment myapp -n staging
  when: manual
  only:
    - develop

deploy:production:
  stage: deploy
  image: bitnami/kubectl:latest
  environment:
    name: production
    url: https://example.com
  script:
    - kubectl config use-context production
    - kubectl set image deployment/myapp myapp=$DOCKER_IMAGE:$DOCKER_TAG -n production
    - kubectl rollout status deployment/myapp -n production
  only:
    - main
  when: manual

# Include external CI/CD configuration
include:
  - project: 'my-group/ci-templates'
    file: '/templates/Security.gitlab-ci.yml'
  - template: Code-Quality.gitlab-ci.yml
```

### Common Pipeline Patterns

```yaml
# 1. Matrix Builds (GitHub Actions)
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node: [14, 16, 18]
    include:
      - os: ubuntu-latest
        node: 16
        experimental: true
    exclude:
      - os: windows-latest
        node: 14

# 2. Conditional Execution
# GitHub Actions
if: github.ref == 'refs/heads/main' && !contains(github.event.head_commit.message, '[skip ci]')

# GitLab CI
rules:
  - if: '$CI_COMMIT_BRANCH == "main"'
    when: always
  - if: '$CI_COMMIT_TAG'
    when: never

# 3. Secrets Management
# GitHub Actions
env:
  API_KEY: ${{ secrets.API_KEY }}

# GitLab CI
variables:
  API_KEY: $VAULT_API_KEY

# 4. Artifacts and Caching
# GitHub Actions
- uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-

# GitLab CI
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/

# 5. Parallel Jobs
# GitHub Actions - runs in parallel by default
jobs:
  test-unit:
    runs-on: ubuntu-latest
  test-integration:
    runs-on: ubuntu-latest

# GitLab CI
test:unit:
  stage: test
  parallel: 3
```

### Testing Strategies

```yaml
# 1. Unit Tests
test:unit:
  script:
    - npm run test:unit -- --coverage
    - test $(cat coverage/coverage-summary.json | jq '.total.lines.pct') -gt 80

# 2. Integration Tests
test:integration:
  services:
    - postgres:13
    - redis:6
  script:
    - npm run test:integration

# 3. E2E Tests
test:e2e:
  script:
    - npm run start &
    - npx wait-on http://localhost:3000
    - npm run test:e2e:headless

# 4. Performance Tests
test:performance:
  script:
    - npm run start &
    - npx wait-on http://localhost:3000
    - artillery run performance-test.yml

# 5. Security Tests
test:security:
  script:
    - npm audit
    - npm run test:security
    - snyk test
```

### Deployment Strategies

```yaml
# 1. Blue-Green Deployment
deploy:blue-green:
  script:
    # Deploy to green environment
    - kubectl apply -f k8s/green/
    - kubectl wait --for=condition=ready pod -l version=green
    # Run smoke tests
    - ./scripts/smoke-test.sh green
    # Switch traffic
    - kubectl patch service myapp -p '{"spec":{"selector":{"version":"green"}}}'
    # Delete blue
    - kubectl delete -f k8s/blue/

# 2. Canary Deployment
deploy:canary:
  script:
    # Deploy canary (10% traffic)
    - kubectl apply -f k8s/canary/
    - kubectl patch service myapp -p '{"spec":{"selector":{"version":"canary"}}}'
    - ./scripts/set-traffic.sh canary 10
    # Monitor for 10 minutes
    - sleep 600
    # Check metrics
    - ./scripts/check-metrics.sh
    # Promote or rollback
    - ./scripts/promote-canary.sh

# 3. Rolling Deployment
deploy:rolling:
  script:
    - kubectl set image deployment/myapp myapp=$IMAGE_TAG
    - kubectl rollout status deployment/myapp
    - kubectl rollout history deployment/myapp

# 4. Rolling Rollback
rollback:
  script:
    - kubectl rollout undo deployment/myapp
    - kubectl rollout status deployment/myapp
  when: manual

# 5. Feature Flags
deploy:feature-flags:
  script:
    - export FEATURE_NEW_UI=true
    - kubectl create configmap feature-flags --from-env-file=.env
    - kubectl rollout restart deployment/myapp
```

### Pipeline Optimization

```yaml
# 1. Parallel Execution
stages:
  - build
  - test    # All test jobs run in parallel
  - deploy

# 2. Fail Fast
strategy:
  fail-fast: true  # Stop all jobs on first failure

# 3. Caching Dependencies
cache:
  paths:
    - node_modules/
    - .npm/
    - ~/.cache/pip/

# 4. Docker Layer Caching
docker:build:
  script:
    - docker build --cache-from $IMAGE:latest --build-arg BUILDKIT_INLINE_CACHE=1 -t $IMAGE:$TAG .

# 5. Conditional Stages
deploy:
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      changes:
        - src/**/*
        - Dockerfile

# 6. Job Dependencies
test:integration:
  needs:
    - job: build
      artifacts: true

# 7. Retry on Failure
test:flaky:
  retry:
    max: 2
    when:
      - runner_system_failure
      - stuck_or_timeout_failure
```

### Monitoring and Notifications

```yaml
# 1. Slack Notifications (GitHub Actions)
- name: Slack Notification
  uses: 8398a7/action-slack@v3
  if: always()
  with:
    status: ${{ job.status }}
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
    fields: repo,message,commit,author,action,eventName,ref,workflow

# 2. Email Notifications (GitLab CI)
notify:success:
  stage: .post
  script:
    - echo "Pipeline succeeded" | mail -s "Success" team@example.com
  when: on_success

# 3. Teams Notification
- name: Teams Notification
  uses: aliencube/microsoft-teams-actions@v0.8.0
  with:
    webhook_uri: ${{ secrets.TEAMS_WEBHOOK }}
    title: Build Status
    summary: Build ${{ job.status }}

# 4. Custom Webhooks
notify:webhook:
  script:
    - |
      curl -X POST $WEBHOOK_URL \
        -H "Content-Type: application/json" \
        -d "{\"status\":\"$CI_JOB_STATUS\",\"pipeline\":\"$CI_PIPELINE_ID\"}"
```

## Best Practices

1. **Keep pipelines fast** - Optimize build times with caching and parallelization
2. **Fail fast** - Run quick tests first, expensive tests later
3. **Use semantic versioning** - Tag releases properly
4. **Implement security scanning** - SAST, DAST, dependency scanning
5. **Test before deploying** - Always run tests before deployment
6. **Use environments** - Separate dev, staging, and production
7. **Implement rollback** - Always have a rollback strategy
8. **Monitor deployments** - Track deployment metrics and errors
9. **Use secrets management** - Never hardcode credentials
10. **Document your pipeline** - Add comments and README
11. **Version your pipeline** - Keep pipeline code in version control
12. **Review pipeline changes** - Treat pipeline code like application code

## Common Issues and Solutions

```bash
# 1. Pipeline Timeout
# Solution: Increase timeout or optimize build
timeout: 60m

# 2. Out of Disk Space
# Solution: Clean up artifacts and caches
- rm -rf node_modules/ dist/
- docker system prune -af

# 3. Flaky Tests
# Solution: Retry mechanism
retry: 2

# 4. Slow Dependencies
# Solution: Cache dependencies
cache:
  paths:
    - node_modules/

# 5. Failed Deployments
# Solution: Implement health checks
- kubectl wait --for=condition=ready pod -l app=myapp --timeout=300s
```
