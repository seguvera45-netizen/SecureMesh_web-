# SecureMesh Project Structure

This document provides a complete overview of the SecureMesh project organization.

```
securemesh/
│
├── 📱 FRONTEND (React + Tailwind CSS)
│   │
│   ├── src/app/
│   │   ├── App.tsx                      # Main app component with routing
│   │   │
│   │   ├── components/                  # Reusable components
│   │   │   ├── Sidebar.tsx             # Navigation sidebar with mobile support
│   │   │   └── figma/                  # Figma-specific components
│   │   │
│   │   ├── contexts/                    # React contexts
│   │   │   └── AuthContext.tsx         # Authentication state management
│   │   │
│   │   ├── layouts/                     # Page layouts
│   │   │   └── RootLayout.tsx          # Main layout with auth protection
│   │   │
│   │   ├── lib/                        # Utilities and libraries
│   │   │   └── supabase.ts             # Supabase client configuration
│   │   │
│   │   ├── pages/                      # Application pages
│   │   │   ├── AuthPage.tsx            # 🔐 Login & Signup page
│   │   │   ├── DashboardPage.tsx       # 📊 System overview dashboard
│   │   │   ├── ServiceMonitorPage.tsx  # 🔍 Microservices health monitor
│   │   │   ├── RequestAnalyzerPage.tsx # 📡 API request logs analyzer
│   │   │   └── SecurityControlPage.tsx # ⚙️ Security policies control panel
│   │   │
│   │   └── routes.tsx                  # React Router configuration
│   │
│   ├── styles/                         # Global styles
│   │   ├── fonts.css                   # Font imports
│   │   └── theme.css                   # Theme tokens
│   │
│   ├── package.json                    # Frontend dependencies
│   └── vite.config.ts                  # Vite configuration
│
├── 🔧 BACKEND (Node.js Microservices)
│   │
│   ├── auth-service/                   # Authentication service (Port 3001)
│   │   ├── src/
│   │   │   └── index.js               # Auth API (login, register, validate)
│   │   └── package.json               # Dependencies (express, jwt, bcrypt)
│   │
│   ├── user-service/                   # User management (Port 3002)
│   │   ├── src/
│   │   │   └── index.js               # User API (profile, CRUD operations)
│   │   └── package.json
│   │
│   ├── order-service/                  # Order processing (Port 3003)
│   │   ├── src/
│   │   │   └── index.js               # Order API (create, list, update)
│   │   └── package.json
│   │
│   ├── payment-service/                # Payment processing (Port 3004)
│   │   ├── src/
│   │   │   └── index.js               # Payment API (charge, validate, refund)
│   │   └── package.json
│   │
│   └── Dockerfile                      # Multi-stage Docker build for services
│
├── ☸️ KUBERNETES (Deployment Manifests)
│   │
│   ├── namespace.yaml                  # SecureMesh namespace with Istio injection
│   │
│   └── deployments/                    # Service deployments
│       ├── auth-service.yaml          # Auth deployment + service + SA
│       ├── user-service.yaml          # User deployment + service + SA
│       ├── order-service.yaml         # Order deployment + service + SA
│       └── payment-service.yaml       # Payment deployment + service + SA
│
├── 🛡️ ISTIO (Service Mesh Configuration)
│   │
│   ├── peer-authentication.yaml        # 🔒 STRICT mTLS enforcement
│   ├── request-authentication.yaml     # 🎫 JWT token validation
│   ├── authorization-policies.yaml     # 👮 RBAC and access control
│   ├── virtual-services.yaml           # 🚦 Traffic routing & canary deploys
│   ├── destination-rules.yaml          # ⚡ Circuit breaking & load balancing
│   ├── rate-limiting.yaml              # 🚨 Rate limiting via Envoy filters
│   └── egress-control.yaml             # 🌐 External API access control
│
├── 📊 OBSERVABILITY (Monitoring Stack)
│   │
│   ├── prometheus.yaml                 # Metrics collection configuration
│   ├── grafana-dashboard.json          # Pre-built Grafana dashboard
│   └── jaeger.yaml                     # Distributed tracing setup
│
├── 📚 DOCUMENTATION
│   │
│   ├── README.md                       # Main project documentation
│   ├── ARCHITECTURE.md                 # Detailed architecture guide
│   ├── SUPABASE_SETUP.md              # Supabase integration guide
│   └── PROJECT_STRUCTURE.md            # This file
│
├── 🐳 DOCKER
│   │
│   └── docker-compose.yml              # Local development with Docker
│
├── ⚙️ CONFIGURATION
│   │
│   ├── .env.example                    # Environment variables template
│   ├── package.json                    # Root package.json
│   └── tsconfig.json                   # TypeScript configuration
│
└── 🔑 SECURITY
    └── kubernetes/secrets/              # (Not committed - create manually)
        └── jwt-secret.yaml             # JWT signing key secret
```

## Quick Navigation

### Frontend Pages

| Page | Path | Description |
|------|------|-------------|
| **Authentication** | `/auth` | User login and registration |
| **Dashboard** | `/dashboard` | System health overview |
| **Service Monitor** | `/services` | Real-time service metrics |
| **Request Analyzer** | `/requests` | API request logs with filtering |
| **Security Control** | `/security` | Istio policy management |

### Backend Services

| Service | Port | Endpoints | Purpose |
|---------|------|-----------|---------|
| **auth-service** | 3001 | `/api/v1/auth/*` | Authentication |
| **user-service** | 3002 | `/api/v1/users/*` | User management |
| **order-service** | 3003 | `/api/v1/orders/*` | Order processing |
| **payment-service** | 3004 | `/api/v1/payments/*` | Payment handling |

### Istio Policies

| Policy | Purpose |
|--------|---------|
| **peer-authentication.yaml** | Enforces mTLS between all services |
| **request-authentication.yaml** | Validates JWT tokens |
| **authorization-policies.yaml** | Implements RBAC rules |
| **virtual-services.yaml** | Routes traffic with retries |
| **destination-rules.yaml** | Circuit breaking and load balancing |
| **rate-limiting.yaml** | Protects against abuse |
| **egress-control.yaml** | Controls external API access |

## File Counts

- **React Components**: 6 pages + 1 layout + 1 sidebar = 8 files
- **Backend Services**: 4 microservices
- **Kubernetes Manifests**: 5 files
- **Istio Configurations**: 7 files
- **Documentation**: 4 files

## Key Features Implemented

### ✅ Frontend
- [x] Modern React 18 with TypeScript
- [x] Tailwind CSS for styling
- [x] Motion (Framer Motion) animations
- [x] React Router for navigation
- [x] Supabase integration ready
- [x] Responsive mobile design
- [x] Real-time data visualization (Recharts)
- [x] Dark theme interface

### ✅ Backend
- [x] 4 production-ready microservices
- [x] JWT authentication
- [x] RBAC implementation
- [x] Health check endpoints
- [x] Error handling
- [x] CORS configuration

### ✅ Kubernetes
- [x] Namespace with Istio injection
- [x] Service deployments with replicas
- [x] Health probes (liveness & readiness)
- [x] Resource limits
- [x] ServiceAccounts for each service

### ✅ Istio Security
- [x] STRICT mTLS enforcement
- [x] JWT validation
- [x] Authorization policies (default deny + explicit allow)
- [x] RBAC with role-based claims
- [x] Rate limiting
- [x] Circuit breaking
- [x] Egress traffic control

### ✅ Observability
- [x] Prometheus metrics
- [x] Grafana dashboards
- [x] Jaeger tracing
- [x] Service health monitoring

### ✅ Documentation
- [x] Comprehensive README
- [x] Architecture documentation
- [x] Supabase setup guide
- [x] Project structure overview

## Next Steps

1. **Connect Supabase** (Optional)
   - Follow `SUPABASE_SETUP.md`
   - Enable real authentication

2. **Run Locally**
   ```bash
   # Frontend
   pnpm install && pnpm run dev
   
   # Backend (each service)
   cd backend/auth-service && npm install && npm start
   ```

3. **Deploy to Kubernetes**
   ```bash
   # Install Istio
   istioctl install --set profile=demo -y
   
   # Deploy SecureMesh
   kubectl apply -f kubernetes/
   kubectl apply -f istio/
   ```

4. **Monitor & Observe**
   ```bash
   # Access Grafana
   kubectl port-forward -n istio-system svc/grafana 3000:3000
   
   # Access Jaeger
   kubectl port-forward -n istio-system svc/jaeger-query 16686:16686
   ```

## Development Workflow

1. **Make changes** to frontend or backend
2. **Test locally** with hot reload
3. **Build Docker images** for backend
4. **Deploy to Kubernetes** cluster
5. **Monitor** with Grafana/Jaeger
6. **Review logs** in Request Analyzer

## Production Checklist

- [ ] Change JWT_SECRET to a strong random value
- [ ] Configure real Supabase project
- [ ] Update Istio policies for production
- [ ] Set up proper TLS certificates
- [ ] Configure ingress with real domain
- [ ] Enable monitoring alerts
- [ ] Set up log aggregation
- [ ] Configure backups
- [ ] Review security policies
- [ ] Load test the system

---

🎉 **SecureMesh is ready for deployment!** Follow the README.md for detailed setup instructions.
