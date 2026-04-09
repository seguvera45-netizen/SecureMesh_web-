# SecureMesh Quick Start Guide

Get SecureMesh up and running in 5 minutes!

## 🚀 Prerequisites

- Node.js 18+ and pnpm installed
- (Optional) Docker for backend services
- (Optional) Kubernetes cluster for full deployment

## ⚡ Fastest Path: Frontend Only

The frontend works immediately with mock data - perfect for exploring the UI!

```bash
# 1. Install dependencies
pnpm install

# 2. Start the app
pnpm run dev

# 3. Open in browser
# Visit: http://localhost:5173

# 4. Login with any email/password
# The app uses mock authentication for demo purposes
```

**That's it!** You now have a fully functional Zero Trust monitoring dashboard.

## 🎯 What You Can Do

### Explore the Dashboard
- View system health metrics
- Monitor service performance
- Analyze request logs
- Review security policies

### Navigate the App
- **Dashboard** (`/dashboard`) - System overview
- **Services** (`/services`) - Microservice health monitoring
- **Requests** (`/requests`) - API request analyzer with filters
- **Security** (`/security`) - Istio policy control panel

## 🔐 Enable Real Authentication (Optional)

Connect Supabase for real user accounts:

### Option 1: Figma Make Settings (Easiest)
1. Click settings in Figma Make
2. Go to Supabase section
3. Click "Connect Supabase Project"
4. Authorize and you're done!

### Option 2: Manual Setup
```bash
# 1. Create Supabase project at https://app.supabase.com

# 2. Copy credentials

# 3. Create .env file
cp .env.example .env

# 4. Add your credentials to .env
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key

# 5. Restart dev server
pnpm run dev
```

See `SUPABASE_SETUP.md` for detailed instructions.

## 🔧 Run Backend Services (Optional)

Want to run the actual microservices?

### Using Docker Compose (Recommended)
```bash
# 1. Set JWT secret
export JWT_SECRET="my-secret-key"

# 2. Start all services
docker-compose up -d

# 3. Check service health
curl http://localhost:3001/health  # auth-service
curl http://localhost:3002/health  # user-service
curl http://localhost:3003/health  # order-service
curl http://localhost:3004/health  # payment-service
```

### Using Node.js Directly
```bash
# Terminal 1: Auth Service
cd backend/auth-service
npm install
npm start

# Terminal 2: User Service
cd backend/user-service
npm install
npm start

# Terminal 3: Order Service
cd backend/order-service
npm install
npm start

# Terminal 4: Payment Service
cd backend/payment-service
npm install
npm start
```

## ☸️ Deploy to Kubernetes (Advanced)

Full Zero Trust deployment with Istio:

```bash
# 1. Install Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-*
istioctl install --set profile=demo -y

# 2. Create namespace
kubectl apply -f kubernetes/namespace.yaml

# 3. Create secrets
kubectl create secret generic jwt-secret \
  --from-literal=secret="your-production-secret" \
  -n securemesh

# 4. Deploy services
kubectl apply -f kubernetes/deployments/

# 5. Apply Istio security
kubectl apply -f istio/

# 6. Check status
kubectl get pods -n securemesh
kubectl get svc -n securemesh

# 7. Access the app
kubectl port-forward -n securemesh svc/frontend 8080:80
```

## 🔍 Test API Endpoints

### Test Authentication
```bash
# Register a new user
curl -X POST http://localhost:3001/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'

# Login
curl -X POST http://localhost:3001/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'

# Save the token from response
TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

# Use token to access protected endpoint
curl -H "Authorization: Bearer $TOKEN" \
  http://localhost:3002/api/v1/users/profile
```

### Test Authorization
```bash
# Try to access admin endpoint (should fail)
curl -H "Authorization: Bearer $TOKEN" \
  http://localhost:3002/api/v1/users

# Response: {"error": "Admin access required"}
```

## 📊 Access Monitoring Dashboards

After Kubernetes deployment:

```bash
# Grafana (metrics)
kubectl port-forward -n istio-system svc/grafana 3000:3000
# Open: http://localhost:3000

# Jaeger (tracing)
kubectl port-forward -n istio-system svc/jaeger-query 16686:16686
# Open: http://localhost:16686

# Prometheus (raw metrics)
kubectl port-forward -n istio-system svc/prometheus 9090:9090
# Open: http://localhost:9090
```

## 🐛 Troubleshooting

### Frontend won't start
```bash
# Clear node_modules and reinstall
rm -rf node_modules
pnpm install
pnpm run dev
```

### Backend service fails
```bash
# Check if port is already in use
lsof -i :3001  # Replace with your port

# Kill the process if needed
kill -9 <PID>
```

### Kubernetes pods not starting
```bash
# Check pod logs
kubectl logs -n securemesh <pod-name>

# Check Istio injection
kubectl get pods -n securemesh -o jsonpath='{.items[*].spec.containers[*].name}'
# Should see "istio-proxy" container

# Restart pod
kubectl delete pod -n securemesh <pod-name>
```

### Supabase connection error
```bash
# Verify environment variables
echo $VITE_SUPABASE_URL
echo $VITE_SUPABASE_ANON_KEY

# Check .env file exists
cat .env

# Restart dev server
pnpm run dev
```

## 📚 Next Steps

1. **Read the docs**
   - `README.md` - Complete documentation
   - `ARCHITECTURE.md` - System design details
   - `SUPABASE_SETUP.md` - Authentication guide

2. **Explore the code**
   - `src/app/pages/` - Frontend pages
   - `backend/*/src/index.js` - Backend services
   - `istio/` - Security policies

3. **Customize**
   - Add new services
   - Modify security policies
   - Create custom dashboards

4. **Deploy to production**
   - Set up CI/CD pipeline
   - Configure domain and TLS
   - Enable monitoring alerts

## 💡 Pro Tips

- **Use pnpm** instead of npm for faster installs
- **Enable Istio logs** for debugging: `istioctl dashboard envoy <pod-name>`
- **Monitor metrics** in real-time with Grafana
- **Test security** with attack simulation mode
- **Check Istio docs** for advanced features

## 🎉 You're All Set!

SecureMesh is now running. Happy securing! 🔒

For questions or issues, check:
- README.md for detailed docs
- GitHub Issues for known problems
- Istio documentation for mesh questions

---

**Need help?** See the full documentation in README.md
