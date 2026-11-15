# Docker & GCP for Full-Stack Developers

## **Overview**

Docker and Google Cloud Platform are critical for modern deployment. This guide covers:
- Containerizing Java/Angular/PostgreSQL applications
- Deploying to GCP services
- Production best practices
- Interview Q&A

---

## **Part 1: Docker Fundamentals**

### **1. Docker Concepts**

**Image**: Blueprint for container (like a class)
- Contains application code
- Dependencies
- Runtime environment
- Configuration

**Container**: Running instance of image (like an object)
- Isolated process
- Has its own filesystem
- Own networking
- Own resources

**Dockerfile**: Instructions to build image
- Base image selection
- Install dependencies
- Copy code
- Configure app
- Specify entry point

**Registry**: Repository for images
- Docker Hub (public)
- Google Container Registry (GCR)
- Artifact Registry
- Private registries

---

### **2. Dockerfile Best Practices**

#### **Backend Dockerfile (Java/Spring Boot)**

**Basic Version (Works but not optimal):**
```dockerfile
FROM openjdk:11
COPY target/app.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Better Version (Multi-stage build):**
```dockerfile
# Stage 1: Build
FROM maven:3.8-openjdk-11 as builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline

COPY . .
RUN mvn clean package -DskipTests

# Stage 2: Runtime (smaller image)
FROM openjdk:11-jre-slim
WORKDIR /app

# Create non-root user for security
RUN useradd -m -u 1000 appuser

# Copy jar from builder
COPY --from=builder --chown=appuser:appuser /app/target/app.jar .

# Use non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

EXPOSE 8080

# Use exec form (forwards signals properly)
ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-jar", "app.jar"]
```

**Key Improvements:**
- Multi-stage build reduces image size
- Non-root user for security
- Health check for orchestration
- Proper signal handling

**Production Version (with optimization):**
```dockerfile
FROM maven:3.8-openjdk-11 as builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline

COPY . .
RUN mvn clean package -DskipTests

# Runtime with JVM optimization
FROM openjdk:11-jre-slim
WORKDIR /app

RUN useradd -m -u 1000 appuser
COPY --from=builder --chown=appuser:appuser /app/target/app.jar .

USER appuser

# JVM memory settings for containers
ENV JAVA_OPTS="-XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:+ParallelRefProcEnabled"

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

EXPOSE 8080
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

---

#### **Frontend Dockerfile (Angular)**

**Basic Version:**
```dockerfile
# Build stage
FROM node:16 as builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Serve stage
FROM nginx:alpine
COPY --from=builder /app/dist/your-app /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**nginx.conf (for Angular routing):**
```nginx
server {
  listen 80;
  location / {
    root /usr/share/nginx/html;
    try_files $uri $uri/ /index.html;
  }
  
  # API proxy
  location /api/ {
    proxy_pass http://backend:8080;
  }
}
```

**Production Version:**
```dockerfile
FROM node:16-alpine as builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm ci

COPY . .
RUN npm run build

# Nginx with security headers
FROM nginx:alpine
COPY --from=builder /app/dist/your-app /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Add security headers
RUN echo 'add_header X-Content-Type-Options "nosniff" always;' >> /etc/nginx/conf.d/default.conf && \
    echo 'add_header X-Frame-Options "DENY" always;' >> /etc/nginx/conf.d/default.conf && \
    echo 'add_header X-XSS-Protection "1; mode=block" always;' >> /etc/nginx/conf.d/default.conf

USER nginx
EXPOSE 80
HEALTHCHECK CMD wget --quiet --tries=1 --spider http://localhost/health || exit 1
CMD ["nginx", "-g", "daemon off;"]
```

---

### **3. docker-compose.yml (Local Development)**

```yaml
version: '3.8'

services:
  # Backend
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/mydb
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: postgres
      SPRING_JPA_HIBERNATE_DDL_AUTO: validate
    depends_on:
      - postgres
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 3s
      retries: 3

  # Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "4200:80"
    depends_on:
      - backend
    networks:
      - app-network
    environment:
      API_BASE_URL: http://backend:8080

  # Database
  postgres:
    image: postgres:14-alpine
    environment:
      POSTGRES_DB: mydb
      POSTGRES_PASSWORD: postgres
      POSTGRES_USER: postgres
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - app-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # pgAdmin for database management (optional)
  pgadmin:
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: admin
    ports:
      - "5050:80"
    depends_on:
      - postgres
    networks:
      - app-network

volumes:
  postgres_data:

networks:
  app-network:
    driver: bridge
```

**Usage:**
```bash
# Start all services
docker-compose up

# Start in background
docker-compose up -d

# View logs
docker-compose logs -f

# Stop
docker-compose down

# Remove volumes too (careful!)
docker-compose down -v
```

---

### **4. Docker Best Practices**

**DO's:**
✅ Use multi-stage builds (reduce image size)
✅ Use specific base image versions (avoid `latest`)
✅ Run as non-root user (security)
✅ Use .dockerignore (faster builds)
✅ Combine commands to reduce layers
✅ Add health checks
✅ Use environment variables for config
✅ Keep images small (Alpine Linux)
✅ Scan images for vulnerabilities
✅ Use container registries

**DON'Ts:**
❌ Run as root
❌ Use large base images unnecessarily
❌ Hardcode secrets in Dockerfile
❌ Use `latest` tag in production
❌ Leave unused layers
❌ Put everything in one stage
❌ Ignore security vulnerabilities
❌ Use large application images

---

### **5. Docker Commands**

```bash
# Build image
docker build -t myapp:1.0 .
docker build -t myapp:latest -t myapp:1.0 .  # Multiple tags

# Run container
docker run -d -p 8080:8080 --name my-container myapp:1.0

# With environment variables
docker run -d -p 8080:8080 \
  -e DATABASE_URL=postgresql://localhost/mydb \
  myapp:1.0

# With volume mount
docker run -d -p 8080:8080 \
  -v /local/path:/container/path \
  myapp:1.0

# View running containers
docker ps
docker ps -a  # Including stopped

# View logs
docker logs container-name
docker logs -f container-name  # Follow

# Execute command in container
docker exec -it container-name bash
docker exec -it container-name curl localhost:8080

# Stop/remove container
docker stop container-name
docker rm container-name
docker rm $(docker ps -aq)  # Remove all stopped

# View images
docker images

# Push to registry
docker tag myapp:1.0 gcr.io/my-project/myapp:1.0
docker push gcr.io/my-project/myapp:1.0

# Security scan
docker scout cves myapp:1.0
```

---

## **Part 2: Google Cloud Platform (GCP)**

### **1. GCP Services for Full-Stack Applications**

| Service | Purpose | Best For |
|---------|---------|----------|
| **Cloud Run** | Serverless container deployment | Stateless apps, simple deployments |
| **App Engine** | Fully managed platform | Quick deployments, auto-scaling |
| **Compute Engine** | Virtual machines | Complex apps, more control |
| **Kubernetes Engine (GKE)** | Kubernetes cluster | Microservices, complex orchestration |
| **Cloud SQL** | Managed databases | PostgreSQL, MySQL, SQL Server |
| **Cloud Storage** | Object storage | Files, images, backups |
| **Cloud CDN** | Content delivery | Static content caching |
| **Cloud Load Balancing** | Traffic distribution | High availability |
| **Cloud Firestore** | NoSQL database | Real-time data |
| **Cloud Pub/Sub** | Messaging | Event streaming |
| **Cloud Tasks** | Task queues | Scheduled jobs |
| **Cloud Build** | CI/CD | Automated builds |
| **Secret Manager** | Secrets storage | API keys, credentials |
| **Cloud Monitoring** | Observability | Metrics, logs, alerts |

---

### **2. Deploying to Cloud Run (Most Common)**

**Why Cloud Run:**
- Serverless (pay per use)
- Auto-scaling
- No infrastructure management
- Easy deployment
- Good for stateless apps

#### **Setup & Deployment**

**1. Create GCP Project:**
```bash
# Create project
gcloud projects create my-project --name="My Project"

# Set as default
gcloud config set project my-project

# Enable necessary APIs
gcloud services enable run.googleapis.com
gcloud services enable cloudbuild.googleapis.com
gcloud services enable containerregistry.googleapis.com
gcloud services enable sqladmin.googleapis.com
```

**2. Create Cloud SQL Instance (PostgreSQL):**
```bash
# Create instance
gcloud sql instances create my-postgres \
  --database-version=POSTGRES_14 \
  --tier=db-f1-micro \
  --region=us-central1

# Create database
gcloud sql databases create mydb \
  --instance=my-postgres

# Create user
gcloud sql users create postgres \
  --instance=my-postgres \
  --password

# Get connection details
gcloud sql instances describe my-postgres
```

**3. Deploy Backend to Cloud Run:**

```bash
# Build and push image
gcloud builds submit \
  --tag gcr.io/my-project/myapp-backend:1.0 \
  --source ./backend

# Or build locally
docker build -t gcr.io/my-project/myapp-backend:1.0 ./backend
docker push gcr.io/my-project/myapp-backend:1.0

# Deploy to Cloud Run
gcloud run deploy myapp-backend \
  --image gcr.io/my-project/myapp-backend:1.0 \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --port 8080 \
  --memory 512Mi \
  --cpu 1 \
  --timeout 300 \
  --set-env-vars="SPRING_DATASOURCE_URL=jdbc:postgresql://CLOUD_SQL_IP:5432/mydb,SPRING_DATASOURCE_USERNAME=postgres"
```

**4. Deploy Frontend to Cloud Run:**

```bash
docker build -t gcr.io/my-project/myapp-frontend:1.0 ./frontend
docker push gcr.io/my-project/myapp-frontend:1.0

gcloud run deploy myapp-frontend \
  --image gcr.io/my-project/myapp-frontend:1.0 \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --port 80 \
  --memory 256Mi
```

**5. Setup Cloud SQL Proxy (Connection):**

In your Java application, use Cloud SQL Connector:

```xml
<!-- pom.xml -->
<dependency>
  <groupId>com.google.cloud.sql</groupId>
  <artifactId>cloud-sql-connector-java-postgres</artifactId>
  <version>1.4.3</version>
</dependency>
```

```properties
# application.properties
spring.cloud.sql.database-name=mydb
spring.cloud.sql.sql-alchemy-uri=postgresql://
spring.datasource.url=jdbc:postgresql://cloudsql/mydb
```

Or use TCP connection (simpler):
```bash
# Get Cloud SQL IP
gcloud sql instances describe my-postgres --format='value(ipAddresses[0].ipAddress)'

# Use in environment variable
SPRING_DATASOURCE_URL=jdbc:postgresql://IP:5432/mydb
```

---

### **3. Cloud SQL Best Practices**

```bash
# Backup configuration
gcloud sql backups create \
  --instance=my-postgres

# Scheduled backups
gcloud sql instances patch my-postgres \
  --backup-start-time 03:00

# Enable backup for automatic backups
gcloud sql instances patch my-postgres \
  --enable-bin-log

# View backups
gcloud sql backups list --instance=my-postgres

# Restore from backup
gcloud sql backups restore BACKUP_ID \
  --backup-instance=my-postgres
```

---

### **4. Kubernetes Engine (GKE) - For Complex Apps**

**When to use GKE:**
- Microservices architecture
- Complex orchestration needs
- High availability requirements
- Multi-region deployments

**Setup:**

```bash
# Create GKE cluster
gcloud container clusters create my-cluster \
  --zone us-central1-a \
  --num-nodes 3 \
  --machine-type n1-standard-1 \
  --enable-autoscaling \
  --min-nodes 1 \
  --max-nodes 5

# Get credentials
gcloud container clusters get-credentials my-cluster --zone us-central1-a

# Verify kubectl
kubectl cluster-info
```

**Kubernetes Manifests:**

```yaml
# deployment.yaml - Backend
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp-backend
  template:
    metadata:
      labels:
        app: myapp-backend
    spec:
      containers:
      - name: backend
        image: gcr.io/my-project/myapp-backend:1.0
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_DATASOURCE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
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
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5

---
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-backend
spec:
  selector:
    app: myapp-backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer

---
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: "INFO"
  ENVIRONMENT: "production"

---
# secret.yaml (create with: kubectl create secret)
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  url: BASE64_ENCODED_URL
  username: BASE64_ENCODED_USERNAME
  password: BASE64_ENCODED_PASSWORD
```

**Deploy to GKE:**

```bash
# Create secret
kubectl create secret generic db-credentials \
  --from-literal=url='jdbc:postgresql://cloudsql:5432/mydb' \
  --from-literal=username=postgres \
  --from-literal=password=yourpassword

# Deploy
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f configmap.yaml

# View deployments
kubectl get deployments
kubectl get pods
kubectl get services

# Check logs
kubectl logs -f deployment/myapp-backend

# Scale deployment
kubectl scale deployment myapp-backend --replicas=5

# Rolling update
kubectl set image deployment/myapp-backend \
  backend=gcr.io/my-project/myapp-backend:2.0 \
  --record

# Rollback
kubectl rollout undo deployment/myapp-backend
```

---

### **5. Cloud Build (CI/CD)**

**cloudbuild.yaml:**

```yaml
steps:
  # Step 1: Run tests
  - name: 'gcr.io/cloud-builders/maven'
    args: ['test']
    dir: 'backend'

  # Step 2: Build image
  - name: 'gcr.io/cloud-builders/docker'
    args:
      - 'build'
      - '-t'
      - 'gcr.io/$PROJECT_ID/myapp-backend:$SHORT_SHA'
      - '-t'
      - 'gcr.io/$PROJECT_ID/myapp-backend:latest'
      - './backend'

  # Step 3: Push image
  - name: 'gcr.io/cloud-builders/docker'
    args:
      - 'push'
      - 'gcr.io/$PROJECT_ID/myapp-backend:$SHORT_SHA'

  # Step 4: Deploy to Cloud Run
  - name: 'gcr.io/cloud-builders/gke-deploy'
    args:
      - run
      - --filename=backend/k8s/
      - --image=gcr.io/$PROJECT_ID/myapp-backend:$SHORT_SHA
      - --location=us-central1
      - --cluster=my-cluster

images:
  - 'gcr.io/$PROJECT_ID/myapp-backend:$SHORT_SHA'
  - 'gcr.io/$PROJECT_ID/myapp-backend:latest'
```

**Setup trigger:**
```bash
gcloud builds submit \
  --config=cloudbuild.yaml

# Or create GitHub trigger
gcloud builds connect github
gcloud builds triggers create github \
  --name=myapp-build \
  --repo-name=myapp \
  --repo-owner=myusername \
  --branch-pattern=^main$ \
  --build-config=cloudbuild.yaml
```

---

### **6. Cloud Storage (Files & Backups)**

```bash
# Create bucket
gsutil mb gs://my-app-bucket

# Upload file
gsutil cp local-file.zip gs://my-app-bucket/

# Set public access
gsutil acl ch -u AllUsers:R gs://my-app-bucket/file.txt

# List contents
gsutil ls -r gs://my-app-bucket

# Setup lifecycle (delete old files)
gsutil lifecycle set lifecycle.json gs://my-app-bucket
```

**lifecycle.json:**
```json
{
  "lifecycle": {
    "rule": [
      {
        "action": {"type": "Delete"},
        "condition": {"age": 30}
      }
    ]
  }
}
```

---

### **7. Monitoring & Logging**

```bash
# View logs
gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=myapp-backend"

# Create alert
gcloud alpha monitoring policies create \
  --notification-channels=CHANNEL_ID \
  --display-name="High Error Rate" \
  --condition-threshold-filter='resource.type="cloud_run_revision" AND metric.type="run.googleapis.com/request_count"'

# View metrics
gcloud monitoring metrics-descriptors list
```

---

### **8. Environment Configuration**

**app.yaml (for App Engine):**

```yaml
runtime: java11
entrypoint: java -jar app.jar

env_variables:
  SPRING_DATASOURCE_URL: "jdbc:postgresql://cloudsql/mydb"
  SPRING_DATASOURCE_USERNAME: "postgres"

env_variables_beta:
  SPRING_DATASOURCE_PASSWORD: "secret:db_password"

automatic_scaling:
  min_instances: 1
  max_instances: 5
  min_idle_instances: 1
  max_idle_instances: 3
```

---

## **Complete Example: Deploy Full-Stack App**

### **Step 1: Prepare Project**
```
myapp/
├── backend/
│   ├── pom.xml
│   ├── src/
│   └── Dockerfile
├── frontend/
│   ├── package.json
│   ├── src/
│   └── Dockerfile
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
├── docker-compose.yml
└── cloudbuild.yaml
```

### **Step 2: Set up GCP**
```bash
gcloud projects create myapp-prod
gcloud config set project myapp-prod
gcloud services enable run.googleapis.com sqladmin.googleapis.com cloudbuild.googleapis.com

# Create Cloud SQL
gcloud sql instances create myapp-postgres \
  --database-version=POSTGRES_14 \
  --tier=db-f1-micro \
  --region=us-central1

# Create database
gcloud sql databases create myapp \
  --instance=myapp-postgres

# Create user
gcloud sql users create postgres \
  --instance=myapp-postgres \
  --password
```

### **Step 3: Deploy**
```bash
# Build and push images
docker build -t gcr.io/myapp-prod/backend:1.0 ./backend
docker push gcr.io/myapp-prod/backend:1.0

docker build -t gcr.io/myapp-prod/frontend:1.0 ./frontend
docker push gcr.io/myapp-prod/frontend:1.0

# Deploy backend
gcloud run deploy myapp-backend \
  --image gcr.io/myapp-prod/backend:1.0 \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars="SPRING_DATASOURCE_URL=jdbc:postgresql://CLOUD_SQL_IP:5432/myapp,SPRING_DATASOURCE_USERNAME=postgres"

# Deploy frontend
gcloud run deploy myapp-frontend \
  --image gcr.io/myapp-prod/frontend:1.0 \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated
```

---

## **Interview Questions on Docker & GCP**

### **Docker Questions**

1. **"Why use Docker?"**
   - Consistency across environments
   - Simplified deployment
   - Isolation between services
   - Easy scaling

2. **"What's the difference between image and container?"**
   - Image: Blueprint (static)
   - Container: Running instance (dynamic)

3. **"Explain multi-stage Docker builds."**
   - Reduce image size
   - First stage builds, second stage runs
   - Only include necessary runtime

4. **"How do you optimize Docker images?"**
   - Use multi-stage builds
   - Use Alpine base images
   - Minimize layers
   - Use .dockerignore
   - Non-root users

5. **"What are Docker networking modes?"**
   - Bridge (default, isolated)
   - Host (shares host network)
   - None (no networking)
   - Overlay (for swarm/k8s)

---

### **GCP Questions**

1. **"When would you use Cloud Run vs Compute Engine vs GKE?"**
   - Cloud Run: Stateless apps, quick deployments
   - Compute Engine: Full control, complex apps
   - GKE: Microservices, complex orchestration

2. **"How do you connect to Cloud SQL from Cloud Run?"**
   - Cloud SQL Connector library
   - TCP connection with IP
   - Cloud SQL Proxy

3. **"What's the difference between App Engine and Cloud Run?"**
   - App Engine: Older, tightly integrated
   - Cloud Run: Newer, container-based, more flexible

4. **"How do you manage secrets in GCP?"**
   - Secret Manager service
   - Environment variables
   - Kubernetes secrets
   - Secret rotation

5. **"Explain Cloud Build CI/CD pipeline."**
   - Automated builds on Git push
   - Run tests
   - Build images
   - Deploy to Cloud Run/GKE

6. **"How do you scale applications on GCP?"**
   - Cloud Run: Automatic
   - App Engine: Auto-scaling config
   - GKE: HPA (Horizontal Pod Autoscaler)
   - Compute Engine: Managed instance groups

---

## **Best Practices**

### **Docker**
✅ Use non-root users
✅ Multi-stage builds
✅ Specific base image versions
✅ Health checks
✅ Scan for vulnerabilities
✅ Keep images small

### **GCP**
✅ Use Cloud SQL for databases
✅ Enable automatic backups
✅ Use IAM for access control
✅ Enable monitoring/logging
✅ Use Secrets Manager for credentials
✅ Set up alerts for errors
✅ Use CDN for static content
✅ Implement proper scaling

---

## **Common Mistakes**

❌ Running containers as root
❌ Using `latest` tag in production
❌ Hardcoding secrets in images
❌ Not setting resource limits
❌ Ignoring monitoring
❌ Not backing up databases
❌ Over-engineering small apps
❌ Ignoring security groups/IAM
❌ No health checks
❌ Not testing locally with docker-compose first
