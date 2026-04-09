# SecureMesh - Zero Trust Microservices Platform

A production-ready web application demonstrating Zero Trust architecture with Istio service mesh, featuring modern React frontend, Node.js microservices, and comprehensive security controls.

![SecureMesh Architecture](https://img.shields.io/badge/Architecture-Zero%20Trust-blue)
![Istio](https://img.shields.io/badge/Service%20Mesh-Istio-blue)
![Kubernetes](https://img.shields.io/badge/Platform-Kubernetes-blue)

## 🎯 Overview

SecureMesh implements a complete Zero Trust security model with:

- **Mutual TLS (mTLS)** - Encrypted service-to-service communication
- **JWT Validation** - Token-based authentication for all requests
- **RBAC** - Role-based access control with fine-grained permissions
- **Rate Limiting** - Protection against abuse and DDoS
- **Observability** - Comprehensive monitoring with Prometheus, Grafana, and Jaeger
- **Modern UI** - React + Tailwind CSS dashboard for security monitoring

## 📁 Project Structure

```
securemesh/
├── src/app/                      # React Frontend
│   ├── components/               # Reusable UI components
│   │   └── Sidebar.tsx          # Navigation sidebar
│   ├── contexts/                # React contexts
│   │   └── AuthContext.tsx      # Authentication state
│   ├── layouts/                 # Layout components
│   │   └── RootLayout.tsx       # Main app layout
│   ├── lib/                     # Libraries and utilities
│   │   └── supabase.ts          # Supabase client
│   ├── pages/                   # Application pages
│   │   ├── AuthPage.tsx         # Login & Signup
│   │   ├── DashboardPage.tsx    # System overview
│   │   ├── ServiceMonitorPage.tsx  # Service health
│   │   ├── RequestAnalyzerPage.tsx # Request logs
│   │   └── SecurityControlPage.tsx # Security policies
│   ├── routes.tsx               # Route configuration
│   └── App.tsx                  # App entry point
│
├── backend/                     # Microservices
│   ├── auth-service/           # Authentication service
│   │   ├── src/index.js        # Auth API
│   │   └── package.json        # Dependencies
│   ├── user-service/           # User management
│   │   ├── src/index.js        # User API
│   │   └── package.json
│   ├── order-service/          # Order processing
│   │   ├── src/index.js        # Order API
│   │   └── package.json
│   └── payment-service/        # Payment processing
│       ├── src/index.js        # Payment API
│       └── package.json
│
├── kubernetes/                  # Kubernetes Manifests
│   ├── namespace.yaml          # Namespace with Istio injection
│   └── deployments/            # Service deployments
│       ├── auth-service.yaml
│       ├── user-service.yaml
│       ├── order-service.yaml
│       └── payment-service.yaml
│
├── istio/                       # Istio Configuration
│   ├── peer-authentication.yaml      # mTLS enforcement
│   ├── request-authentication.yaml   # JWT validation
│   ├── authorization-policies.yaml   # RBAC rules
│   ├── virtual-services.yaml         # Traffic routing
│   ├── destination-rules.yaml        # Circuit breaking
│   ├── rate-limiting.yaml            # Rate limits
│   └── egress-control.yaml           # Egress policies
│
├── observability/               # Monitoring Configuration
│   ├── prometheus.yaml         # Metrics collection
│   ├── grafana-dashboard.json  # Dashboard definition
│   └── jaeger.yaml             # Distributed tracing
│
└── docs/                        # Documentation
    ├── SETUP.md                # Setup instructions
    ├── ARCHITECTURE.md         # Architecture details
    └── DEPLOYMENT.md           # Deployment guide
```

## 🚀 Quick Start

### Prerequisites

- **Node.js 18+** and **pnpm**
- **Docker** and **Kubernetes** (minikube, k3s, or cloud provider)
- **Istio 1.20+**
- **kubectl** CLI
- **Supabase** account (for authentication)

### 1. Frontend Setup

```bash
# Install dependencies
pnpm install

# Configure Supabase (optional - see below)
# Create .env file with Supabase credentials
echo "VITE_SUPABASE_URL=your-project-url" > .env
echo "VITE_SUPABASE_ANON_KEY=your-anon-key" >> .env

# Start development server
pnpm run dev
```

**Note:** The frontend works without Supabase using mock data. To enable real authentication:
1. Connect your Supabase project from the **Make settings page**
2. Supabase will automatically configure authentication

### 2. Backend Services Setup

```bash
# Install dependencies for each service
cd backend/auth-service && npm install
cd ../user-service && npm install
cd ../order-service && npm install
cd ../payment-service && npm install

# Set JWT secret (use same secret for all services)
export JWT_SECRET="your-production-secret-key-change-this"

# Start services (in separate terminals)
cd auth-service && npm start    # Port 3001
cd user-service && npm start    # Port 3002
cd order-service && npm start   # Port 3003
cd payment-service && npm start # Port 3004
```

### 3. Kubernetes Deployment

```bash
# Install Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-*
export PATH=$PWD/bin:$PATH
istioctl install --set profile=demo -y

# Create namespace with Istio injection
kubectl apply -f kubernetes/namespace.yaml

# Create JWT secret
kubectl create secret generic jwt-secret \
  --from-literal=secret="your-production-secret-key" \
  -n securemesh

# Deploy services
kubectl apply -f kubernetes/deployments/

# Apply Istio configurations
kubectl apply -f istio/peer-authentication.yaml
kubectl apply -f istio/request-authentication.yaml
kubectl apply -f istio/authorization-policies.yaml
kubectl apply -f istio/virtual-services.yaml
kubectl apply -f istio/destination-rules.yaml
kubectl apply -f istio/rate-limiting.yaml
kubectl apply -f istio/egress-control.yaml

# Verify deployment
kubectl get pods -n securemesh
kubectl get svc -n securemesh
```

### 4. Observability Setup

```bash
# Deploy Prometheus
kubectl apply -f observability/prometheus.yaml

# Install Grafana
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/grafana.yaml

# Import dashboard
kubectl create configmap grafana-dashboard \
  --from-file=observability/grafana-dashboard.json \
  -n istio-system

# Deploy Jaeger
kubectl apply -f observability/jaeger.yaml

# Access dashboards
kubectl port-forward -n istio-system svc/grafana 3000:3000
kubectl port-forward -n istio-system svc/jaeger-query 16686:16686
kubectl port-forward -n istio-system svc/prometheus 9090:9090
```

## 🔐 Security Features

### 1. Mutual TLS (mTLS)

All service-to-service communication is encrypted using STRICT mTLS mode:

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default-mtls
  namespace: securemesh
spec:
  mtls:
    mode: STRICT
```

### 2. JWT Validation

Every request must include a valid JWT token issued by auth-service:

- Token validation at the edge
- Automatic verification of signature, expiry, and issuer
- Claims-based authorization

### 3. Role-Based Access Control (RBAC)

Fine-grained permissions based on user roles:

- **User role**: Can access own resources
- **Admin role**: Full access to all resources
- Service-to-service permissions enforced

### 4. Zero Trust Model

- Default deny-all policy
- Explicit allow rules for each service
- Least privilege access
- No implicit trust between services

### 5. Rate Limiting

Protection against abuse:

- Auth service: 100 requests/minute
- Payment service: 50 requests/minute
- Per-service customizable limits

### 6. Egress Control

Restrict external API access:

- Only whitelisted external domains allowed
- Blocked by default (REGISTRY_ONLY mode)
- Audit trail for all external calls

## 📊 Monitoring & Observability

### Metrics (Prometheus)

- Request volume per service
- Error rates and status codes
- Latency percentiles (P50, P95, P99)
- mTLS connection status
- Circuit breaker events

### Dashboards (Grafana)

- Service health overview
- Security metrics (auth denials, rate limits)
- Performance trends
- Real-time alerts

### Tracing (Jaeger)

- End-to-end request tracing
- Service dependency graph
- Latency bottleneck identification
- Error propagation analysis

## 🎨 Frontend Features

### Authentication Page

- Email/password login
- User registration
- JWT token management
- Error handling and validation

### Dashboard

- System health overview
- Key performance metrics
- Active services status
- Recent security alerts

### Service Monitor

- Real-time service health
- Resource usage (CPU, memory)
- Request rate and latency
- Replica status

### Request Analyzer

- API request logs
- ALLOW/DENY decisions
- Filter by service, status, time
- Export logs

### Security Control Panel

- View active Istio policies
- Simulation mode for testing
- Blocked request history
- Policy configuration

## 🔧 Configuration

### Environment Variables

**Frontend (.env):**
```bash
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key
```

**Backend (all services):**
```bash
PORT=3001-3004           # Service port
JWT_SECRET=secret-key    # JWT signing key (same for all)
NODE_ENV=production      # Environment
```

### Istio Configuration

Customize security policies in `istio/` directory:

- **mTLS mode**: STRICT, PERMISSIVE, or DISABLE
- **JWT issuer**: Configure trusted auth providers
- **RBAC rules**: Add custom authorization policies
- **Rate limits**: Adjust per-service limits
- **Circuit breaker**: Configure thresholds

## 🧪 Testing

### Test JWT Authentication

```bash
# Get token
TOKEN=$(curl -X POST http://localhost:3001/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@securemesh.io","password":"password"}' \
  | jq -r '.token')

# Test authenticated request
curl -H "Authorization: Bearer $TOKEN" \
  http://localhost:3002/api/v1/users/profile
```

### Test Rate Limiting

```bash
# Spam requests to trigger rate limit
for i in {1..150}; do
  curl -X POST http://localhost:3001/api/v1/auth/login \
    -H "Content-Type: application/json" \
    -d '{"email":"test@example.com","password":"test"}'
done
```

### Test Circuit Breaker

```bash
# Simulate service failure
kubectl scale deployment payment-service --replicas=0 -n securemesh

# Trigger circuit breaker
for i in {1..20}; do
  curl -H "Authorization: Bearer $TOKEN" \
    http://localhost:3003/api/v1/orders
done
```

## 📚 API Documentation

### Auth Service (Port 3001)

- `POST /api/v1/auth/register` - User registration
- `POST /api/v1/auth/login` - User login
- `POST /api/v1/auth/validate` - Validate JWT token
- `GET /health` - Health check

### User Service (Port 3002)

- `GET /api/v1/users/profile` - Get current user
- `GET /api/v1/users/:userId` - Get user by ID
- `PUT /api/v1/users/:userId` - Update user
- `GET /api/v1/users` - List users (admin)
- `DELETE /api/v1/users/:userId` - Delete user (admin)

### Order Service (Port 3003)

- `GET /api/v1/orders` - List user orders
- `GET /api/v1/orders/:orderId` - Get order details
- `POST /api/v1/orders` - Create order
- `PUT /api/v1/orders/:orderId` - Update order
- `DELETE /api/v1/orders/:orderId` - Cancel order

### Payment Service (Port 3004)

- `POST /api/v1/payments/validate` - Validate payment method
- `POST /api/v1/payments/charge` - Process payment
- `GET /api/v1/payments/:paymentId` - Get payment details
- `GET /api/v1/payments` - List user payments
- `POST /api/v1/payments/:paymentId/refund` - Refund (admin)

## 🚀 Production Deployment

### Docker Images

Build and push Docker images:

```bash
# Build services
docker build -t securemesh/auth-service:latest backend/auth-service
docker build -t securemesh/user-service:latest backend/user-service
docker build -t securemesh/order-service:latest backend/order-service
docker build -t securemesh/payment-service:latest backend/payment-service

# Push to registry
docker push securemesh/auth-service:latest
docker push securemesh/user-service:latest
docker push securemesh/order-service:latest
docker push securemesh/payment-service:latest
```

### Helm Deployment (Optional)

Create Helm chart for easier deployment:

```bash
helm create securemesh
# Customize values.yaml with service configurations
helm install securemesh ./securemesh -n securemesh
```

### Cloud Deployment

Deploy to managed Kubernetes:

- **AWS EKS**: Use AWS Application Load Balancer
- **Google GKE**: Enable Istio on GKE addon
- **Azure AKS**: Install Istio via Azure CLI

## 🔄 CI/CD Pipeline

Example GitHub Actions workflow:

```yaml
name: Deploy SecureMesh
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build and push images
        run: |
          docker build -t securemesh/auth-service:${{ github.sha }} backend/auth-service
          docker push securemesh/auth-service:${{ github.sha }}
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/auth-service \
            auth-service=securemesh/auth-service:${{ github.sha }} \
            -n securemesh
```

## 🛡️ Security Best Practices

1. **Rotate JWT secrets** regularly
2. **Use Kubernetes secrets** for sensitive data
3. **Enable audit logging** for all API calls
4. **Monitor authorization denials** for suspicious activity
5. **Keep Istio updated** to latest stable version
6. **Implement backup** and disaster recovery
7. **Run security scans** on container images
8. **Use network policies** in addition to Istio RBAC

## 📖 Additional Resources

- [Istio Documentation](https://istio.io/latest/docs/)
- [Kubernetes Security Best Practices](https://kubernetes.io/docs/concepts/security/)
- [Zero Trust Architecture](https://www.nist.gov/publications/zero-trust-architecture)
- [Supabase Authentication](https://supabase.com/docs/guides/auth)

## 🤝 Contributing

Contributions welcome! Please read CONTRIBUTING.md for guidelines.

## 📄 License

MIT License - see LICENSE file for details

## 🆘 Support

- Documentation: `/docs` directory
- Issues: GitHub Issues
- Security: security@securemesh.io

---

Built with ❤️ using React, Istio, Kubernetes, and Zero Trust principles
