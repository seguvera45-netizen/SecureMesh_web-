# Supabase Setup Guide for SecureMesh

This guide explains how to connect Supabase to enable real authentication in SecureMesh.

## Why Supabase?

SecureMesh uses Supabase for:
- **User Authentication**: Email/password and OAuth login
- **JWT Token Management**: Secure token generation and validation
- **User Session Management**: Persistent login sessions
- **Security**: Built-in protection against common attacks

## Current Status

✅ **Frontend is fully functional** with mock authentication
❌ **Supabase is not connected** - using placeholder credentials

## How to Connect Supabase

### Option 1: Using Figma Make Settings (Recommended)

1. **Open Make Settings**
   - Click the settings icon in Figma Make
   - Navigate to "Supabase" section

2. **Connect Your Project**
   - Click "Connect Supabase Project"
   - Sign in to your Supabase account
   - Select an existing project or create new one
   - Authorize the connection

3. **Automatic Configuration**
   - Make will automatically:
     - Create `supabase/functions/server/kv_store.tsx`
     - Create `supabase/functions/server/index.tsx`
     - Create `utils/supabase/info.tsx` with credentials
   - Your app will now use real authentication!

### Option 2: Manual Configuration

1. **Create Supabase Project**
   ```bash
   # Visit https://app.supabase.com
   # Click "New Project"
   # Note your project URL and anon key
   ```

2. **Configure Environment Variables**
   ```bash
   # Create .env file
   cp .env.example .env
   
   # Edit .env with your credentials
   VITE_SUPABASE_URL=https://your-project-id.supabase.co
   VITE_SUPABASE_ANON_KEY=your-anon-key-here
   ```

3. **Update Supabase Client**
   
   The app already has Supabase client configured in `src/app/lib/supabase.ts`:
   ```typescript
   import { createClient } from '@supabase/supabase-js';
   
   const supabaseUrl = import.meta.env.VITE_SUPABASE_URL || 'placeholder';
   const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY || 'placeholder';
   
   export const supabase = createClient(supabaseUrl, supabaseAnonKey);
   ```

4. **Enable Authentication Providers**
   
   In Supabase Dashboard:
   - Go to Authentication > Providers
   - Enable Email provider
   - (Optional) Enable Google OAuth:
     - Get OAuth credentials from Google Cloud Console
     - Add to Supabase settings

5. **Test Authentication**
   ```bash
   # Start the app
   pnpm run dev
   
   # Visit http://localhost:5173/auth
   # Try signing up with a real email
   ```

## Authentication Flow with Supabase

### Sign Up Flow
```
User enters email/password
    ↓
Frontend calls supabase.auth.signUp()
    ↓
Supabase creates user account
    ↓
Sends confirmation email
    ↓
User confirms email
    ↓
JWT token generated
    ↓
User redirected to dashboard
```

### Sign In Flow
```
User enters credentials
    ↓
Frontend calls supabase.auth.signInWithPassword()
    ↓
Supabase validates credentials
    ↓
Returns JWT token + user data
    ↓
Token stored in AuthContext
    ↓
Used for API requests
```

## Security Considerations

### JWT Token Management

Supabase tokens include:
- **User ID**: Unique identifier
- **Email**: User email address
- **Role**: User role (authenticated)
- **Expiry**: Automatic expiration (default: 1 hour)
- **Refresh Token**: For seamless re-authentication

### Backend Integration

To validate Supabase JWTs in your microservices:

```javascript
const jwt = require('jsonwebtoken');
const jwksClient = require('jwks-rsa');

const client = jwksClient({
  jwksUri: `${process.env.SUPABASE_URL}/auth/v1/keys`
});

function getKey(header, callback) {
  client.getSigningKey(header.kid, function(err, key) {
    const signingKey = key.publicKey || key.rsaPublicKey;
    callback(null, signingKey);
  });
}

const verifyToken = (token) => {
  return new Promise((resolve, reject) => {
    jwt.verify(token, getKey, {
      issuer: process.env.SUPABASE_URL,
      algorithms: ['RS256']
    }, (err, decoded) => {
      if (err) reject(err);
      else resolve(decoded);
    });
  });
};
```

## Testing Authentication

### Test User Registration

```bash
curl -X POST http://localhost:5173/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@securemesh.io",
    "password": "SecurePass123!"
  }'
```

### Test User Login

```bash
curl -X POST http://localhost:5173/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@securemesh.io",
    "password": "SecurePass123!"
  }'
```

### Test Token Validation

```bash
TOKEN="your-jwt-token-here"

curl -X GET http://localhost:3002/api/v1/users/profile \
  -H "Authorization: Bearer $TOKEN"
```

## Troubleshooting

### Issue: "Invalid API key"

**Solution**: 
- Verify SUPABASE_URL and SUPABASE_ANON_KEY in .env
- Check that keys match your Supabase project
- Restart the dev server after changing .env

### Issue: "Email not confirmed"

**Solution**:
- Check spam folder for confirmation email
- In Supabase Dashboard, go to Authentication > Users
- Manually confirm the user
- Or disable email confirmation in Settings > Authentication

### Issue: "CORS error"

**Solution**:
- Add your local URL to allowed origins in Supabase
- Settings > API > CORS
- Add `http://localhost:5173`

### Issue: "Token expired"

**Solution**:
- Implement automatic token refresh:
  ```typescript
  useEffect(() => {
    const { data: authListener } = supabase.auth.onAuthStateChange(
      async (event, session) => {
        if (event === 'TOKEN_REFRESHED') {
          console.log('Token refreshed!');
        }
      }
    );
    return () => {
      authListener?.unsubscribe();
    };
  }, []);
  ```

## Advanced Features

### Row Level Security (RLS)

Protect your data with Supabase RLS:

```sql
-- Enable RLS on users table
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Policy: Users can only read their own data
CREATE POLICY "Users can view own data"
ON users FOR SELECT
USING (auth.uid() = id);

-- Policy: Only admins can update any user
CREATE POLICY "Admins can update users"
ON users FOR UPDATE
USING (auth.jwt()->>'role' = 'admin');
```

### OAuth Integration

Enable Google Sign-In:

1. Get OAuth credentials from [Google Cloud Console](https://console.cloud.google.com)
2. Add to Supabase: Authentication > Providers > Google
3. Update frontend:
   ```typescript
   const signInWithGoogle = async () => {
     const { data, error } = await supabase.auth.signInWithOAuth({
       provider: 'google',
       options: {
         redirectTo: 'http://localhost:5173/dashboard'
       }
     });
   };
   ```

### Multi-Factor Authentication (MFA)

Enable MFA for enhanced security:

```typescript
// Enable MFA
const { data, error } = await supabase.auth.mfa.enroll({
  factorType: 'totp'
});

// Verify MFA code
const { data, error } = await supabase.auth.mfa.verify({
  factorId: data.id,
  challengeId: data.challengeId,
  code: '123456'
});
```

## Production Deployment

### Environment Variables

In production, use secure secret management:

```bash
# Kubernetes Secret
kubectl create secret generic supabase-secrets \
  --from-literal=url=https://your-project.supabase.co \
  --from-literal=anon-key=your-anon-key \
  -n securemesh
```

### API Rate Limiting

Configure Supabase rate limits:
- Authentication > Rate Limits
- Set appropriate limits per endpoint
- Enable captcha for public endpoints

### Monitoring

Track authentication metrics:
- Failed login attempts
- User registration trends
- Token refresh frequency
- Session duration

## Resources

- [Supabase Documentation](https://supabase.com/docs)
- [Supabase Auth Guide](https://supabase.com/docs/guides/auth)
- [JWT Debugging Tool](https://jwt.io)
- [Supabase Discord](https://discord.supabase.com)

---

**Note**: Make is not intended for collecting PII or securing highly sensitive data. For production applications with strict compliance requirements, consider additional security measures and consult with security experts.
