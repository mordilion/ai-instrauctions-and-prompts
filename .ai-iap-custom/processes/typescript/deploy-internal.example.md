# Deploy to Internal Platform (TypeScript/Node.js)

> **Scope**: TypeScript/Node.js backend services
> **Purpose**: Deploy to company Kubernetes cluster with Helm
> **Type**: Custom Process

---

## Overview

This guide establishes deployment to the company's internal Kubernetes platform using Helm charts and CI/CD pipelines.

---

## Prerequisites

> **BEFORE starting**, ensure you have:

- ✅ Access to company VPN
- ✅ `kubectl` configured for company cluster
- ✅ `helm` CLI installed (v3.0+)
- ✅ Access to Harbor registry (`harbor.company.com`)
- ✅ GitLab CI/CD runner configured
- ✅ Service account credentials from Platform team

---

## Phase 1: Prepare Application

### 1.1: Create Dockerfile

```dockerfile
# Use company base image
FROM harbor.company.com/base/node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies (use internal registry)
RUN npm config set registry https://npm.company.com
RUN npm ci --only=production

# Copy application code
COPY dist/ ./dist/
COPY config/ ./config/

# Set non-root user
USER node

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

# Start application
CMD ["node", "dist/main.js"]
```

### 1.2: Create `.dockerignore`

```
node_modules/
npm-debug.log
.git/
.env
.env.local
*.md
test/
coverage/
.github/
```

### 1.3: Add Health Check Endpoint

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('health')
export class HealthController {
  @Get()
  check() {
    return {
      status: 'ok',
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
      memory: process.memoryUsage(),
    };
  }

  @Get('ready')
  readiness() {
    // Check database, Redis, etc.
    return { status: 'ready' };
  }

  @Get('live')
  liveness() {
    return { status: 'alive' };
  }
}
```

---

## Phase 2: Create Helm Chart

### 2.1: Initialize Helm Chart

```bash
mkdir -p helm/myapp
cd helm/myapp

# Create chart structure
helm create myapp
```

### 2.2: Configure `Chart.yaml`

```yaml
apiVersion: v2
name: myapp
description: My Application Helm Chart
type: application
version: 1.0.0
appVersion: "1.0.0"
maintainers:
  - name: Your Team
    email: team@company.com
```

### 2.3: Configure `values.yaml`

```yaml
replicaCount: 3

image:
  repository: harbor.company.com/myteam/myapp
  pullPolicy: IfNotPresent
  tag: ""  # Overridden by CI/CD

imagePullSecrets:
  - name: harbor-credentials

service:
  type: ClusterIP
  port: 3000
  targetPort: 3000

ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: myapp.company.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: myapp-tls
      hosts:
        - myapp.company.com

resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

livenessProbe:
  httpGet:
    path: /health/live
    port: 3000
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /health/ready
    port: 3000
  initialDelaySeconds: 10
  periodSeconds: 5

env:
  - name: NODE_ENV
    value: production
  - name: DATABASE_URL
    valueFrom:
      secretKeyRef:
        name: myapp-secrets
        key: database-url
```

---

## Phase 3: Set Up CI/CD Pipeline

### 3.1: Create `.gitlab-ci.yml`

```yaml
stages:
  - test
  - build
  - deploy

variables:
  DOCKER_REGISTRY: harbor.company.com
  DOCKER_IMAGE: $DOCKER_REGISTRY/myteam/myapp
  HELM_CHART: helm/myapp

# Run tests
test:
  stage: test
  image: node:18
  script:
    - npm ci
    - npm run test
    - npm run lint
  only:
    - merge_requests
    - main

# Build Docker image
build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $HARBOR_USER -p $HARBOR_PASSWORD $DOCKER_REGISTRY
  script:
    - docker build -t $DOCKER_IMAGE:$CI_COMMIT_SHORT_SHA .
    - docker tag $DOCKER_IMAGE:$CI_COMMIT_SHORT_SHA $DOCKER_IMAGE:latest
    - docker push $DOCKER_IMAGE:$CI_COMMIT_SHORT_SHA
    - docker push $DOCKER_IMAGE:latest
  only:
    - main

# Deploy to staging
deploy:staging:
  stage: deploy
  image: alpine/helm:latest
  before_script:
    - kubectl config use-context company-staging
  script:
    - helm upgrade --install myapp $HELM_CHART
        --namespace myteam-staging
        --create-namespace
        --set image.tag=$CI_COMMIT_SHORT_SHA
        --set env.NODE_ENV=staging
        --wait
  environment:
    name: staging
    url: https://myapp-staging.company.com
  only:
    - main

# Deploy to production (manual)
deploy:production:
  stage: deploy
  image: alpine/helm:latest
  before_script:
    - kubectl config use-context company-production
  script:
    - helm upgrade --install myapp $HELM_CHART
        --namespace myteam-production
        --create-namespace
        --set image.tag=$CI_COMMIT_SHORT_SHA
        --set replicaCount=5
        --wait
  environment:
    name: production
    url: https://myapp.company.com
  when: manual
  only:
    - main
```

---

## Phase 4: Deploy & Monitor

### 4.1: Deploy to Staging

```bash
# Manual deployment
helm upgrade --install myapp ./helm/myapp \
  --namespace myteam-staging \
  --create-namespace \
  --set image.tag=v1.0.0

# Check deployment
kubectl get pods -n myteam-staging
kubectl logs -f -n myteam-staging deployment/myapp

# Check service
kubectl get svc -n myteam-staging
kubectl get ingress -n myteam-staging
```

### 4.2: Verify Deployment

```bash
# Test health endpoints
curl https://myapp-staging.company.com/health
curl https://myapp-staging.company.com/health/ready
curl https://myapp-staging.company.com/health/live

# Check metrics
kubectl top pods -n myteam-staging

# View logs
kubectl logs -n myteam-staging -l app=myapp --tail=100
```

### 4.3: Set Up Monitoring

Add to `values.yaml`:

```yaml
serviceMonitor:
  enabled: true
  interval: 30s
  path: /metrics
  labels:
    prometheus: company

podAnnotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "3000"
  prometheus.io/path: "/metrics"
```

---

## Git Workflow

> **ALWAYS** follow this workflow:

```bash
# Phase 1: Development
git checkout -b feature/my-feature
# Make changes
git add .
git commit -m "feat: add new feature"
git push origin feature/my-feature

# Phase 2: Create MR → triggers CI tests
# Wait for tests to pass

# Phase 3: Merge to main → triggers build + staging deploy
# Verify in staging environment

# Phase 4: Manual production deploy
# Go to GitLab → Pipelines → Run manual deploy job
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Image pull error | Check Harbor credentials: `kubectl get secret harbor-credentials -n myteam-staging` |
| Pod crash loop | Check logs: `kubectl logs -n myteam-staging <pod-name>` |
| Health check failing | Increase `initialDelaySeconds` in values.yaml |
| Out of memory | Increase `resources.limits.memory` |
| Slow startup | Check database connection timeout |
| Ingress not working | Verify DNS: `nslookup myapp-staging.company.com` |

---

## AI Self-Check

Before marking deployment as complete:

- [ ] Dockerfile uses company base image
- [ ] Health endpoints implemented (`/health`, `/health/ready`, `/health/live`)
- [ ] Helm chart created with all required values
- [ ] Resources limits configured (CPU, memory)
- [ ] Autoscaling enabled (min 3 replicas)
- [ ] CI/CD pipeline configured in GitLab
- [ ] Secrets stored in Kubernetes secrets (not in code)
- [ ] Staging deployment successful
- [ ] Production deployment approved by team lead
- [ ] Monitoring alerts configured in Grafana
- [ ] Documentation updated in Confluence
- [ ] Runbook created for on-call team

---

## Internal Resources

- **Confluence**: [Internal Platform Docs](https://wiki.company.com/platform)
- **Slack**: #platform-support
- **Kubernetes Dashboard**: [https://k8s.company.com](https://k8s.company.com)
- **Harbor Registry**: [https://harbor.company.com](https://harbor.company.com)
- **Grafana**: [https://grafana.company.com](https://grafana.company.com)
