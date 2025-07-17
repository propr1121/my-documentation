---
title: Deployment Guide
layout: default
nav_order: 10
last_modified_date: 2025-07-17
---

{% include freshness_indicator.html %}

# Stream Systems Deployment Guide

{: .fs-3 }
Complete deployment documentation for Stream Systems financial technology platform, covering installation procedures, configuration management, and production deployment strategies.

---

## 🚀 Deployment Overview

### Deployment Architecture
Stream Systems follows a **multi-tier deployment** architecture with the following environments:

- **Development**: Developer workstation and testing
- **Testing**: Integration testing and QA validation
- **Staging**: Pre-production validation and UAT
- **Production**: Live trading environment
- **Disaster Recovery**: Geographically separate backup site

### Deployment Strategy
- **Blue-Green Deployment**: Zero-downtime production deployments
- **Canary Releases**: Gradual rollout for risk mitigation
- **Feature Flags**: Progressive feature activation
- **Automated Rollback**: Immediate reversion capability

---

## 📋 Prerequisites

### Infrastructure Requirements

#### Hardware Specifications
| Component | Development | Testing | Production |
|-----------|-------------|---------|------------|
| **Application Servers** | 4 vCPU, 8GB RAM | 8 vCPU, 16GB RAM | 16 vCPU, 32GB RAM |
| **Database Servers** | 4 vCPU, 16GB RAM | 8 vCPU, 32GB RAM | 32 vCPU, 128GB RAM |
| **Load Balancers** | 2 vCPU, 4GB RAM | 4 vCPU, 8GB RAM | 8 vCPU, 16GB RAM |
| **Storage** | 100GB SSD | 500GB SSD | 2TB NVMe SSD |
| **Network** | 1Gbps | 1Gbps | 10Gbps |

#### Operating System
- **Linux Distribution**: Ubuntu 22.04 LTS or RHEL 9
- **Kernel Version**: 5.15 or later
- **Docker**: 24.0 or later
- **Kubernetes**: 1.28 or later (for containerized deployments)

### Software Dependencies

#### Runtime Requirements
```bash
# Java Runtime Environment
OpenJDK 17 LTS
Java Memory: -Xmx4g -Xms2g

# Database Systems
PostgreSQL 15.x (Primary database)
Redis 7.x (Caching layer)
MongoDB 7.x (Document storage)

# Message Queue
Apache Kafka 3.x
RabbitMQ 3.x (Internal messaging)

# Web Server
Nginx 1.24.x (Reverse proxy)
Apache Tomcat 10.x (Application server)
```

#### Development Tools
```bash
# Build Tools
Maven 3.9.x
Gradle 8.x
Node.js 18.x LTS
npm 9.x

# Version Control
Git 2.40.x
GitLab CI/CD or Jenkins

# Monitoring
Prometheus 2.45.x
Grafana 10.x
ELK Stack 8.x
```

---

## 🔧 Environment Setup

### Development Environment

#### Local Development Setup
```bash
# Clone repository
git clone https://github.com/streamsystems/platform.git
cd platform

# Setup environment variables
cp .env.example .env.local
nano .env.local

# Install dependencies
./scripts/setup-dev.sh

# Start local services
docker-compose -f docker-compose.dev.yml up -d

# Run application
./gradlew bootRun
```

#### Environment Configuration
```properties
# Database Configuration
DB_HOST=localhost
DB_PORT=5432
DB_NAME=streamsystems_dev
DB_USER=dev_user
DB_PASSWORD=dev_password

# Redis Configuration
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=redis_dev_pass

# External Services
MARKET_DATA_URL=https://api-dev.marketdata.com
REGULATORY_API_URL=https://sandbox.regulatory.gov
```

### Testing Environment

#### Automated Testing Setup
```bash
# Setup test database
./scripts/setup-test-db.sh

# Run unit tests
./gradlew test

# Run integration tests
./gradlew integrationTest

# Run end-to-end tests
./scripts/e2e-tests.sh

# Generate test reports
./gradlew jacocoTestReport
```

#### Test Data Management
```sql
-- Load test fixtures
INSERT INTO clients (id, name, status) VALUES 
  ('TEST_CLIENT_001', 'Test Client Alpha', 'ACTIVE'),
  ('TEST_CLIENT_002', 'Test Client Beta', 'ACTIVE');

-- Load test instruments
INSERT INTO instruments (symbol, name, type) VALUES
  ('EUR/USD', 'Euro US Dollar', 'FX_SPOT'),
  ('GBP/USD', 'British Pound US Dollar', 'FX_SPOT');
```

---

## 🏗️ Build & Packaging

### Build Process

#### Maven Build Configuration
```xml
<!-- pom.xml -->
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <configuration>
        <image>
          <name>streamsystems/${project.artifactId}:${project.version}</name>
        </image>
      </configuration>
    </plugin>
    <plugin>
      <groupId>org.jacoco</groupId>
      <artifactId>jacoco-maven-plugin</artifactId>
    </plugin>
  </plugins>
</build>
```

#### Gradle Build Configuration
```gradle
// build.gradle
plugins {
    id 'org.springframework.boot' version '3.1.5'
    id 'io.spring.dependency-management' version '1.1.3'
    id 'org.sonarqube' version '4.4.1'
    id 'jacoco'
}

jar {
    archiveBaseName = 'stream-systems-platform'
    archiveVersion = version
}

bootJar {
    layered {
        enabled = true
    }
}
```

### Docker Containerization

#### Application Dockerfile
```dockerfile
# Multi-stage build
FROM openjdk:17-jdk-alpine AS builder
WORKDIR /app
COPY . .
RUN ./gradlew clean bootJar

FROM openjdk:17-jre-alpine AS runtime
WORKDIR /app

# Security: Create non-root user
RUN addgroup -g 1001 streamapp && \
    adduser -u 1001 -G streamapp -s /bin/sh -D streamapp

# Copy application
COPY --from=builder /app/build/libs/*.jar app.jar

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

USER streamapp
EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### Docker Compose for Services
```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=production
      - DB_HOST=database
      - REDIS_HOST=cache
    depends_on:
      - database
      - cache
    
  database:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: streamsystems
      POSTGRES_USER: stream_user
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    
  cache:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}
    
volumes:
  postgres_data:
```

---

## 🚀 Production Deployment

### Kubernetes Deployment

#### Application Deployment Manifest
```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: stream-systems-app
  namespace: production
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: stream-systems-app
  template:
    metadata:
      labels:
        app: stream-systems-app
    spec:
      containers:
      - name: app
        image: streamsystems/platform:v2.1.0
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "production"
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: password
        resources:
          requests:
            memory: "2Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
```

#### Service Configuration
```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: stream-systems-service
  namespace: production
spec:
  selector:
    app: stream-systems-app
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: stream-systems-ingress
  namespace: production
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - platform.streamsystems.io
    secretName: stream-systems-tls
  rules:
  - host: platform.streamsystems.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: stream-systems-service
            port:
              number: 80
```

### Database Deployment

#### PostgreSQL High Availability
```yaml
# k8s/postgres-ha.yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: postgres-cluster
  namespace: production
spec:
  instances: 3
  primaryUpdateStrategy: unsupervised
  
  postgresql:
    parameters:
      max_connections: "200"
      shared_buffers: "256MB"
      effective_cache_size: "1GB"
      
  bootstrap:
    initdb:
      database: streamsystems
      owner: stream_user
      secret:
        name: postgres-credentials
        
  storage:
    size: 1Ti
    storageClass: fast-ssd
    
  monitoring:
    enabled: true
```

### Load Balancer Configuration

#### Nginx Configuration
```nginx
# /etc/nginx/sites-available/streamsystems
upstream stream_backend {
    least_conn;
    server 10.1.1.10:8080 max_fails=3 fail_timeout=30s;
    server 10.1.1.11:8080 max_fails=3 fail_timeout=30s;
    server 10.1.1.12:8080 max_fails=3 fail_timeout=30s;
}

server {
    listen 443 ssl http2;
    server_name platform.streamsystems.io;
    
    ssl_certificate /etc/ssl/certs/streamsystems.crt;
    ssl_certificate_key /etc/ssl/private/streamsystems.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384;
    
    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options DENY always;
    add_header X-Content-Type-Options nosniff always;
    
    location / {
        proxy_pass http://stream_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts
        proxy_connect_timeout 30s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
    
    location /actuator/health {
        proxy_pass http://stream_backend;
        access_log off;
    }
}
```

---

## ⚙️ Configuration Management

### Environment-Specific Configuration

#### Production Configuration
```yaml
# application-production.yml
spring:
  profiles:
    active: production
    
  datasource:
    url: jdbc:postgresql://postgres-cluster-rw:5432/streamsystems
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 30000
      
  redis:
    host: ${REDIS_HOST}
    port: 6379
    password: ${REDIS_PASSWORD}
    timeout: 2000ms
    
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: ${JWT_ISSUER_URI}
          
logging:
  level:
    com.streamsystems: INFO
    org.springframework.security: WARN
  pattern:
    file: "%d{ISO8601} [%thread] %-5level %logger{36} - %msg%n"
  file:
    name: /var/log/streamsystems/application.log
    max-size: 100MB
    max-history: 30

management:
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus
  endpoint:
    health:
      show-details: when-authorized
      
streamsystems:
  trading:
    risk-limits:
      max-position-size: 100000000
      max-daily-volume: 1000000000
    market-hours:
      start: "08:00"
      end: "17:00"
      timezone: "America/New_York"
```

### Secrets Management

#### Kubernetes Secrets
```bash
# Create database credentials
kubectl create secret generic db-credentials \
  --from-literal=username=stream_user \
  --from-literal=password=super_secure_password \
  --namespace=production

# Create Redis credentials  
kubectl create secret generic redis-credentials \
  --from-literal=password=redis_secure_password \
  --namespace=production

# Create TLS certificates
kubectl create secret tls stream-systems-tls \
  --cert=streamsystems.crt \
  --key=streamsystems.key \
  --namespace=production
```

#### HashiCorp Vault Integration
```yaml
# vault-config.yaml
apiVersion: v1
kind: Secret
metadata:
  name: vault-auth
  namespace: production
type: Opaque
data:
  token: <base64-encoded-vault-token>
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: stream-systems-app
spec:
  template:
    metadata:
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "stream-systems"
        vault.hashicorp.com/agent-inject-secret-db: "secret/data/production/db"
        vault.hashicorp.com/agent-inject-template-db: |
          {% raw %}{{- with secret "secret/data/production/db" -}}
          DB_PASSWORD={{ .Data.data.password }}
          {{- end }}{% endraw %}
```

---

## 📊 Monitoring & Observability

### Application Monitoring

#### Prometheus Configuration
```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'stream-systems'
    static_configs:
      - targets: ['stream-systems-service:80']
    metrics_path: /actuator/prometheus
    scrape_interval: 10s
    
  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres-exporter:9187']
      
  - job_name: 'redis'
    static_configs:
      - targets: ['redis-exporter:9121']
```

#### Grafana Dashboard
```json
{
  "dashboard": {
    "title": "Stream Systems Platform",
    "panels": [
      {
        "title": "Application Health",
        "type": "stat",
        "targets": [
          {
            "expr": "up{job=\"stream-systems\"}",
            "legendFormat": "{{instance}}"
          }
        ]
      },
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{uri}}"
          }
        ]
      },
      {
        "title": "Database Connections",
        "type": "graph",
        "targets": [
          {
            "expr": "hikaricp_connections_active",
            "legendFormat": "Active Connections"
          }
        ]
      }
    ]
  }
}
```

### Log Management

#### ELK Stack Configuration
```yaml
# filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/streamsystems/*.log
  fields:
    service: stream-systems
    environment: production

output.elasticsearch:
  hosts: ["elasticsearch:9200"]
  index: "stream-systems-%{+yyyy.MM.dd}"

logging.level: info
```

---

## 🔄 CI/CD Pipeline

### GitLab CI Configuration

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - security
  - deploy

variables:
  DOCKER_REGISTRY: registry.streamsystems.io
  IMAGE_NAME: stream-systems-platform

test:
  stage: test
  script:
    - ./gradlew test
    - ./gradlew jacocoTestReport
  artifacts:
    reports:
      junit: build/test-results/test/*.xml
      coverage: build/reports/jacoco/test/jacocoTestReport.xml

build:
  stage: build
  script:
    - docker build -t $IMAGE_NAME:$CI_COMMIT_SHA .
    - docker tag $IMAGE_NAME:$CI_COMMIT_SHA $DOCKER_REGISTRY/$IMAGE_NAME:$CI_COMMIT_SHA
    - docker push $DOCKER_REGISTRY/$IMAGE_NAME:$CI_COMMIT_SHA
  only:
    - main
    - develop

security-scan:
  stage: security
  script:
    - trivy image $DOCKER_REGISTRY/$IMAGE_NAME:$CI_COMMIT_SHA
  allow_failure: false

deploy-staging:
  stage: deploy
  script:
    - kubectl set image deployment/stream-systems-app app=$DOCKER_REGISTRY/$IMAGE_NAME:$CI_COMMIT_SHA -n staging
    - kubectl rollout status deployment/stream-systems-app -n staging
  environment:
    name: staging
    url: https://staging.streamsystems.io
  only:
    - develop

deploy-production:
  stage: deploy
  script:
    - kubectl set image deployment/stream-systems-app app=$DOCKER_REGISTRY/$IMAGE_NAME:$CI_COMMIT_SHA -n production
    - kubectl rollout status deployment/stream-systems-app -n production
  environment:
    name: production
    url: https://platform.streamsystems.io
  when: manual
  only:
    - main
```

### Deployment Scripts

#### Blue-Green Deployment Script
```bash
#!/bin/bash
# deploy-blue-green.sh

set -e

NAMESPACE="production"
NEW_VERSION=$1
CURRENT_COLOR=$(kubectl get service stream-systems-service -n $NAMESPACE -o jsonpath='{.spec.selector.color}')

if [ "$CURRENT_COLOR" == "blue" ]; then
    NEW_COLOR="green"
else
    NEW_COLOR="blue"
fi

echo "Deploying version $NEW_VERSION to $NEW_COLOR environment"

# Update deployment with new version
kubectl set image deployment/stream-systems-$NEW_COLOR app=$NEW_VERSION -n $NAMESPACE

# Wait for rollout to complete
kubectl rollout status deployment/stream-systems-$NEW_COLOR -n $NAMESPACE --timeout=300s

# Health check
echo "Performing health check..."
kubectl exec -n $NAMESPACE deployment/stream-systems-$NEW_COLOR -- curl -f http://localhost:8080/actuator/health

# Switch traffic
echo "Switching traffic to $NEW_COLOR"
kubectl patch service stream-systems-service -n $NAMESPACE -p '{"spec":{"selector":{"color":"'$NEW_COLOR'"}}}'

echo "Deployment complete. Traffic switched to $NEW_COLOR"
```

---

*Deployment Guide last updated: {{ page.last_modified_date | date: "%B %d, %Y" }}*  
*Platform Version: v2.1.0*  
*Next scheduled review: {{ page.last_modified_date | date: "%s" | plus: 2592000 | date: "%B %d, %Y" }}* 