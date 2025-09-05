# Deployment Guide

This guide covers different deployment options for the Adaptive Threat Modeler.

## Table of Contents

- [Local Development](#local-development)
- [Docker Deployment](#docker-deployment)
- [Production Deployment](#production-deployment)
- [Cloud Deployment](#cloud-deployment)
- [Kubernetes Deployment](#kubernetes-deployment)
- [Security Considerations](#security-considerations)
- [Monitoring and Logging](#monitoring-and-logging)
- [Troubleshooting](#troubleshooting)

## Local Development

### Quick Start

```bash
# Clone the repository
git clone https://github.com/Denyme24/Adaptive-Threat-Modeler.git
cd Adaptive-Threat-Modeler

# Start all services
docker-compose up -d

# Or start manually
./scripts/start-dev.sh
```

### Manual Setup

```bash
# Backend
cd backend
go mod tidy
cp env.example .env
go run main.go

# Frontend (new terminal)
cd frontend
npm install
npm run dev

# MCP Agent (new terminal)
cd mcp
pip install -r requirements.txt
cp .env.example .env
python api.py
```

## Docker Deployment

### Development Environment

```bash
# Start development environment
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

### Production Environment

```bash
# Build and start production services
docker-compose -f docker-compose.prod.yml up -d

# Scale services if needed
docker-compose -f docker-compose.prod.yml up -d --scale backend=3
```

## Production Deployment

### Prerequisites

- Docker and Docker Compose
- SSL certificates (Let's Encrypt recommended)
- Domain name
- Firewall configuration

### Setup Steps

1. **Prepare the server:**
   ```bash
   # Update system
   sudo apt update && sudo apt upgrade -y
   
   # Install Docker
   curl -fsSL https://get.docker.com -o get-docker.sh
   sh get-docker.sh
   
   # Install Docker Compose
   sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   ```

2. **Clone and configure:**
   ```bash
   git clone https://github.com/Denyme24/Adaptive-Threat-Modeler.git
   cd Adaptive-Threat-Modeler
   
   # Copy and edit environment variables
   cp .env.example .env
   vim .env
   ```

3. **Setup SSL certificates:**
   ```bash
   # Using Certbot (Let's Encrypt)
   sudo apt install certbot
   sudo certbot certonly --standalone -d your-domain.com
   
   # Copy certificates
   sudo cp /etc/letsencrypt/live/your-domain.com/fullchain.pem ssl/
   sudo cp /etc/letsencrypt/live/your-domain.com/privkey.pem ssl/
   ```

4. **Deploy:**
   ```bash
   docker-compose -f docker-compose.prod.yml up -d
   ```

### Environment Variables

```bash
# Production .env file
PORT=8080
HOST=0.0.0.0
DEBUG=false
LOG_LEVEL=warn

# API URLs
VITE_API_URL=https://your-domain.com

# Security
GITHUB_TOKEN=your_github_token
OPENAI_API_KEY=your_openai_key
SLACK_WEBHOOK_URL=your_slack_webhook
```

## Cloud Deployment

### AWS Deployment

#### Using EC2

1. **Launch EC2 instance:**
   - Choose Ubuntu 20.04 LTS
   - t3.medium or larger
   - Configure security groups (ports 80, 443, 22)

2. **Setup and deploy:**
   ```bash
   # Connect to instance
   ssh -i your-key.pem ubuntu@your-instance-ip
   
   # Follow production deployment steps
   ```

#### Using ECS

1. **Create task definitions** for each service
2. **Setup Application Load Balancer**
3. **Configure ECS service** with desired count

#### Using EKS

Use the Kubernetes deployment guide below.

### Google Cloud Platform

#### Using Compute Engine

Similar to AWS EC2 setup.

#### Using GKE

Use the Kubernetes deployment guide below.

### Azure Deployment

#### Using Virtual Machines

Similar to AWS EC2 setup.

#### Using AKS

Use the Kubernetes deployment guide below.

## Kubernetes Deployment

### Prerequisites

- Kubernetes cluster (v1.20+)
- kubectl configured
- Ingress controller (nginx, traefik)

### Deployment Files

Create `k8s/` directory with the following files:

#### Namespace

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: threat-modeler
```

#### ConfigMap

```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: threat-modeler-config
  namespace: threat-modeler
data:
  PORT: "8080"
  HOST: "0.0.0.0"
  DEBUG: "false"
  LOG_LEVEL: "warn"
```

#### Secrets

```yaml
# k8s/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: threat-modeler-secrets
  namespace: threat-modeler
type: Opaque
data:
  github-token: <base64-encoded-token>
  openai-api-key: <base64-encoded-key>
  slack-webhook: <base64-encoded-webhook>
```

#### Backend Deployment

```yaml
# k8s/backend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: threat-modeler
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: adaptive-threat-modeler/backend:latest
        ports:
        - containerPort: 8080
        envFrom:
        - configMapRef:
            name: threat-modeler-config
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
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

#### Services and Ingress

```yaml
# k8s/services.yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: threat-modeler
spec:
  selector:
    app: backend
  ports:
  - port: 8080
    targetPort: 8080
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: threat-modeler-ingress
  namespace: threat-modeler
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - your-domain.com
    secretName: threat-modeler-tls
  rules:
  - host: your-domain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8080
```

### Deploy to Kubernetes

```bash
# Create namespace
kubectl apply -f k8s/namespace.yaml

# Create secrets (encode your values first)
echo -n 'your-github-token' | base64
kubectl apply -f k8s/secrets.yaml

# Deploy services
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/frontend-deployment.yaml
kubectl apply -f k8s/mcp-deployment.yaml
kubectl apply -f k8s/services.yaml

# Check status
kubectl get pods -n threat-modeler
kubectl get services -n threat-modeler
```

## Security Considerations

### Network Security

- Use HTTPS with valid SSL certificates
- Configure firewalls to limit access
- Use VPN for internal access
- Implement rate limiting

### Application Security

- Regular security updates
- Secure environment variable handling
- Database encryption at rest
- API authentication and authorization

### Monitoring

- Enable audit logging
- Monitor for suspicious activities
- Set up alerts for failures
- Regular security assessments

## Monitoring and Logging

### Application Monitoring

```yaml
# Prometheus monitoring
apiVersion: v1
kind: ServiceMonitor
metadata:
  name: threat-modeler-monitor
spec:
  selector:
    matchLabels:
      app: backend
  endpoints:
  - port: metrics
```

### Log Aggregation

```yaml
# Fluentd log collection
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  # Fluentd configuration
```

### Health Checks

```bash
# Backend health
curl -f http://localhost:8080/health

# Frontend health
curl -f http://localhost:3000/

# Database connectivity
kubectl exec -it backend-pod -- curl -f http://localhost:8080/api/v1/info
```

## Troubleshooting

### Common Issues

#### Port Conflicts

```bash
# Check what's using port 8080
sudo lsof -i :8080

# Kill the process
sudo kill -9 <PID>
```

#### Docker Issues

```bash
# Check container logs
docker-compose logs backend

# Restart containers
docker-compose restart

# Rebuild containers
docker-compose build --no-cache
```

#### Kubernetes Issues

```bash
# Check pod status
kubectl describe pod <pod-name> -n threat-modeler

# Check logs
kubectl logs <pod-name> -n threat-modeler

# Debug networking
kubectl exec -it <pod-name> -n threat-modeler -- nslookup backend-service
```

### Performance Tuning

#### Backend Optimization

```bash
# Increase memory limit
export GOMAXPROCS=4
export GOMEMLIMIT=1GiB
```

#### Database Optimization

```bash
# For high-volume deployments
# Use persistent volumes for data
# Configure backup strategies
# Implement read replicas
```

### Scaling

#### Horizontal Scaling

```bash
# Scale backend services
docker-compose -f docker-compose.prod.yml up -d --scale backend=5

# Kubernetes scaling
kubectl scale deployment backend --replicas=5 -n threat-modeler
```

#### Load Balancing

```bash
# Use nginx for load balancing
upstream backend {
    server backend1:8080;
    server backend2:8080;
    server backend3:8080;
}
```

## Backup and Recovery

### Data Backup

```bash
# Backup analysis data
docker run --rm -v threat-modeler_backend-data:/data -v $(pwd):/backup alpine tar czf /backup/data-backup.tar.gz /data

# Restore data
docker run --rm -v threat-modeler_backend-data:/data -v $(pwd):/backup alpine tar xzf /backup/data-backup.tar.gz -C /
```

### Configuration Backup

```bash
# Backup configurations
cp .env .env.backup
cp docker-compose.prod.yml docker-compose.prod.yml.backup

# Backup Kubernetes configs
kubectl get all -n threat-modeler -o yaml > k8s-backup.yaml
```

## Support

For deployment issues:
- Check the [troubleshooting section](#troubleshooting)
- Review logs for error messages
- Consult the [community forums](https://github.com/Denyme24/Adaptive-Threat-Modeler/discussions)
- Open an issue on GitHub