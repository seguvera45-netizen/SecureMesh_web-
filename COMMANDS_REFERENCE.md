# SecureMesh Commands Reference

Quick reference for all important commands.

## 📦 Frontend Commands

### Development
```bash
# Install dependencies
pnpm install

# Start development server (http://localhost:5173)
pnpm run dev

# Build for production
pnpm run build

# Preview production build
pnpm run preview
```

### Package Management
```bash
# Add a package
pnpm add <package-name>

# Remove a package
pnpm remove <package-name>

# Update all packages
pnpm update
```

## 🔧 Backend Commands

### Individual Services
```bash
# Auth Service (Port 3001)
cd backend/auth-service
npm install
npm start

# User Service (Port 3002)
cd backend/user-service
npm install
npm start

# Order Service (Port 3003)
cd backend/order-service
npm install
npm start

# Payment Service (Port 3004)
cd backend/payment-service
npm install
npm start
```

### All Services with Docker Compose
```bash
# Set JWT secret
export JWT_SECRET="your-secret-key"

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop all services
docker-compose down

# Rebuild and restart
docker-compose up -d --build
```

## 🐳 Docker Commands

### Build Images
```bash
# Build auth service
docker build -t securemesh/auth-service:latest -f backend/Dockerfile backend/auth-service

# Build all services
docker build -t securemesh/auth-service:latest -f backend/Dockerfile backend/auth-service
docker build -t securemesh/user-service:latest -f backend/Dockerfile backend/user-service
docker build -t securemesh/order-service:latest -f backend/Dockerfile backend/order-service
docker build -t securemesh/payment-service:latest -f backend/Dockerfile backend/payment-service
```

### Run Containers
```bash
# Run auth service
docker run -d -p 3001:3001 \
  -e JWT_SECRET="your-secret" \
  --name auth-service \
  securemesh/auth-service:latest

# View logs
docker logs -f auth-service

# Stop container
docker stop auth-service
docker rm auth-service
```

## ☸️ Kubernetes Commands

### Cluster Setup
```bash
# Install Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-*
export PATH=$PWD/bin:$PATH
istioctl install --set profile=demo -y

# Verify Istio installation
istioctl verify-install

# Check Istio version
istioctl version
```

### Namespace & Secrets
```bash
# Create namespace with Istio injection
kubectl apply -f kubernetes/namespace.yaml

# Create JWT secret
kubectl create secret generic jwt-secret \
  --from-literal=secret="your-production-secret-key" \
  -n securemesh

# Verify secret
kubectl get secret jwt-secret -n securemesh
```

### Deploy Services
```bash
# Deploy all services
kubectl apply -f kubernetes/deployments/

# Deploy specific service
kubectl apply -f kubernetes/deployments/auth-service.yaml

# Check deployments
kubectl get deployments -n securemesh

# Check pods
kubectl get pods -n securemesh

# Check services
kubectl get svc -n securemesh
```

### Apply Istio Policies
```bash
# Apply all policies
kubectl apply -f istio/

# Apply specific policy
kubectl apply -f istio/peer-authentication.yaml
kubectl apply -f istio/authorization-policies.yaml

# Verify policies
kubectl get peerauthentication -n securemesh
kubectl get authorizationpolicy -n securemesh
kubectl get virtualservice -n securemesh
```

### Monitoring & Debugging
```bash
# View pod logs
kubectl logs -n securemesh <pod-name>

# Follow logs
kubectl logs -f -n securemesh <pod-name>

# View Istio proxy logs
kubectl logs -n securemesh <pod-name> -c istio-proxy

# Describe pod
kubectl describe pod -n securemesh <pod-name>

# Execute commands in pod
kubectl exec -it -n securemesh <pod-name> -- /bin/sh

# Port forward to service
kubectl port-forward -n securemesh svc/auth-service 3001:3001
```

### Scaling
```bash
# Scale deployment
kubectl scale deployment auth-service --replicas=5 -n securemesh

# Check scaling
kubectl get deployments -n securemesh

# Auto-scaling (HPA)
kubectl autoscale deployment auth-service \
  --cpu-percent=70 \
  --min=3 \
  --max=10 \
  -n securemesh
```

### Cleanup
```bash
# Delete specific deployment
kubectl delete deployment auth-service -n securemesh

# Delete all resources in namespace
kubectl delete -f kubernetes/deployments/ -n securemesh

# Delete namespace (WARNING: deletes everything)
kubectl delete namespace securemesh
```

## 📊 Observability Commands

### Prometheus
```bash
# Port forward Prometheus
kubectl port-forward -n istio-system svc/prometheus 9090:9090

# Access at: http://localhost:9090

# Query example
# istio_requests_total{namespace="securemesh"}
```

### Grafana
```bash
# Install Grafana
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/grafana.yaml

# Port forward Grafana
kubectl port-forward -n istio-system svc/grafana 3000:3000

# Access at: http://localhost:3000
# Default login: admin/admin

# Import custom dashboard
kubectl create configmap grafana-dashboard \
  --from-file=observability/grafana-dashboard.json \
  -n istio-system
```

### Jaeger
```bash
# Deploy Jaeger
kubectl apply -f observability/jaeger.yaml

# Port forward Jaeger UI
kubectl port-forward -n istio-system svc/jaeger-query 16686:16686

# Access at: http://localhost:16686
```

### Kiali (Service Graph)
```bash
# Install Kiali
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/kiali.yaml

# Port forward Kiali
kubectl port-forward -n istio-system svc/kiali 20001:20001

# Access at: http://localhost:20001
```

## 🔍 Istio Commands

### Proxy Status
```bash
# Check proxy sync status
istioctl proxy-status

# Describe specific proxy
istioctl proxy-config cluster <pod-name> -n securemesh

# View routes
istioctl proxy-config route <pod-name> -n securemesh

# View listeners
istioctl proxy-config listener <pod-name> -n securemesh
```

### Policy Analysis
```bash
# Analyze Istio configuration
istioctl analyze -n securemesh

# Check mesh config
istioctl x describe pod <pod-name> -n securemesh

# Validate policies
istioctl validate -f istio/authorization-policies.yaml
```

### Traffic Management
```bash
# Test mTLS
istioctl authn tls-check <pod-name> -n securemesh

# View certificates
istioctl proxy-config secret <pod-name> -n securemesh

# Inject Envoy sidecar manually
istioctl kube-inject -f kubernetes/deployments/auth-service.yaml | kubectl apply -f -
```

## 🧪 API Testing Commands

### Authentication
```bash
# Register user
curl -X POST http://localhost:3001/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@securemesh.io","password":"SecurePass123!"}'

# Login
curl -X POST http://localhost:3001/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@securemesh.io","password":"SecurePass123!"}'

# Save token
TOKEN="paste-jwt-token-here"

# Validate token
curl -X POST http://localhost:3001/api/v1/auth/validate \
  -H "Authorization: Bearer $TOKEN"
```

### User Service
```bash
# Get profile
curl -H "Authorization: Bearer $TOKEN" \
  http://localhost:3002/api/v1/users/profile

# List all users (admin only)
curl -H "Authorization: Bearer $TOKEN" \
  http://localhost:3002/api/v1/users
```

### Order Service
```bash
# Create order
curl -X POST http://localhost:3003/api/v1/orders \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"items":["item1","item2"],"total":150.00}'

# List orders
curl -H "Authorization: Bearer $TOKEN" \
  http://localhost:3003/api/v1/orders
```

### Payment Service
```bash
# Validate payment
curl -X POST http://localhost:3004/api/v1/payments/validate \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"paymentMethod":"credit_card","amount":100.00}'

# Process payment
curl -X POST http://localhost:3004/api/v1/payments/charge \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"orderId":"order-123","amount":100.00,"paymentMethod":"credit_card"}'
```

### Health Checks
```bash
# Check all services
curl http://localhost:3001/health  # auth
curl http://localhost:3002/health  # user
curl http://localhost:3003/health  # order
curl http://localhost:3004/health  # payment
```

## 🛠️ Debugging Commands

### View All Resources
```bash
# All resources in namespace
kubectl get all -n securemesh

# With wide output
kubectl get all -n securemesh -o wide
```

### Troubleshooting
```bash
# Check events
kubectl get events -n securemesh --sort-by='.lastTimestamp'

# Check resource usage
kubectl top pods -n securemesh
kubectl top nodes

# Check Istio injection
kubectl get namespace -L istio-injection

# Test connectivity
kubectl run -it --rm debug --image=curlimages/curl --restart=Never -n securemesh -- sh
# Inside pod:
curl http://auth-service:3001/health
```

### Performance Testing
```bash
# Install hey (HTTP load generator)
# macOS: brew install hey
# Linux: go install github.com/rakyll/hey@latest

# Load test auth service
hey -n 1000 -c 10 http://localhost:3001/health

# Load test with auth
hey -n 1000 -c 10 \
  -H "Authorization: Bearer $TOKEN" \
  http://localhost:3002/api/v1/users/profile
```

## 📝 Useful Aliases

Add to your `.bashrc` or `.zshrc`:

```bash
# Kubernetes
alias k='kubectl'
alias kgp='kubectl get pods -n securemesh'
alias kgs='kubectl get svc -n securemesh'
alias kgd='kubectl get deployments -n securemesh'
alias klogs='kubectl logs -f -n securemesh'

# Istio
alias ipc='istioctl proxy-config'
alias ips='istioctl proxy-status'
alias ia='istioctl analyze'

# Docker
alias dps='docker ps'
alias dlogs='docker logs -f'
alias dstop='docker stop $(docker ps -aq)'
alias drm='docker rm $(docker ps -aq)'

# SecureMesh
alias sm-start='pnpm run dev'
alias sm-backend='docker-compose up -d'
alias sm-logs='docker-compose logs -f'
alias sm-stop='docker-compose down'
```

## 🔄 Common Workflows

### Full Local Development
```bash
# 1. Start backend
export JWT_SECRET="dev-secret"
docker-compose up -d

# 2. Start frontend
pnpm run dev

# 3. Access app
# Frontend: http://localhost:5173
# Auth API: http://localhost:3001
```

### Deploy to Kubernetes
```bash
# 1. Install Istio
istioctl install --set profile=demo -y

# 2. Create namespace
kubectl apply -f kubernetes/namespace.yaml

# 3. Create secrets
kubectl create secret generic jwt-secret \
  --from-literal=secret="prod-secret" \
  -n securemesh

# 4. Deploy services
kubectl apply -f kubernetes/deployments/

# 5. Apply Istio policies
kubectl apply -f istio/

# 6. Verify
kubectl get pods -n securemesh
```

### Update Deployment
```bash
# 1. Build new image
docker build -t securemesh/auth-service:v2 backend/auth-service

# 2. Push to registry
docker push securemesh/auth-service:v2

# 3. Update deployment
kubectl set image deployment/auth-service \
  auth-service=securemesh/auth-service:v2 \
  -n securemesh

# 4. Check rollout
kubectl rollout status deployment/auth-service -n securemesh
```

---

💡 **Tip**: Bookmark this file for quick reference!

For detailed explanations, see README.md and ARCHITECTURE.md.
