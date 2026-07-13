# LifeOS AI — AUTHENTICATION IMPLEMENTATION (Phase 4)

**Version:** 1.0  
**Date:** July 2026  
**Status:** Ready for Development

---

## EXECUTIVE SUMMARY

This document provides a complete guide to implementing secure authentication for LifeOS AI using Supabase Auth. It covers:
- **Setup & Configuration** – Getting Supabase ready
- **Authentication Flows** – Login, signup, password reset
- **Social Login** – Google & GitHub integration
- **Backend Implementation** – API endpoints and security
- **Frontend Implementation** – React components and state management
- **Security Best Practices** – Protecting user data

---

## AUTHENTICATION ARCHITECTURE

```
┌─────────────────────────────────────────────────────┐
│                   CLIENT (Browser)                  │
│  ┌──────────────────────────────────────────────┐   │
│  │  React App                                   │   │
│  │  ├─ Login Component                          │   │
│  │  ├─ Signup Component                         │   │
│  │  ├─ Protected Routes                         │   │
│  │  └─ Session Management                       │   │
│  └──────────────────────────────────────────────┘   │
│              │                          │            │
│              │ HTTPS                    │ HTTPS      │
│              ▼                          ▼            │
├─────────────────────────────────────────────────────┤
│                  EDGE/MIDDLEWARE                     │
│  ├─ Verify JWT Token                               │
│  ├─ Refresh Token if Needed                         │
│  └─ Attach User Context                             │
├─────────────────────────────────────────────────────┤
│                   API LAYER (Next.js)                │
│  ┌──────────────────────────────────────────────┐   │
│  │ POST /api/auth/signup                        │   │
│  │ POST /api/auth/login                         │   │
│  │ POST /api/auth/logout                        │   │
│  │ POST /api/auth/refresh                       │   │
│  │ POST /api/auth/password-reset                │   │
│  │ GET  /api/auth/me (current user)             │   │
│  └──────────────────────────────────────────────┘   │
│              │                          │            │
│              │ JWT Validation           │ JWT in DB  │
│              ▼                          ▼            │
├─────────────────────────────────────────────────────┤
│              SUPABASE AUTH SERVICE                   │
│  ┌──────────────────────────────────────────────┐   │
│  │ JWT Token Generation                         │   │
│  │ User Session Management                      │   │
│  │ Password Hashing (bcrypt)                    │   │
│  │ Email Verification                           │   │
│  │ Social OAuth Providers                       │   │
│  └──────────────────────────────────────────────┘   │
│              │                          │            │
│              ▼                          ▼            │
│        PostgreSQL Database    Email Service (SendGrid)
│        ┌──────────────────┐
│        │ auth.users       │
│        │ user_profiles    │
│        │ sessions         │
│        └──────────────────┘
```

---

## SUPABASE SETUP

### Step 1: Create Supabase Project

**Process:**
1. Go to https://supabase.com
2. Sign up / Log in
3. Click "New Project"
4. Choose organization and project name
5. Set database password (save this!)
6. Select region (closest to users)
7. Wait for setup (~2 minutes)

**Get Your Credentials:**
- `SUPABASE_URL` – From project settings
- `SUPABASE_ANON_KEY` – Public key for client-side
- `SUPABASE_SERVICE_ROLE_KEY` – Secret key for server-side only

### Step 2: Enable Auth Providers

**In Supabase Dashboard → Authentication → Providers:**

```
Enable:
✓ Email (default, always on)
✓ Google OAuth
✓ GitHub OAuth

Configuration:
- Email confirmations: Optional (for MVP, can skip)
- Password requirements: Min 8 chars
- Session expiry: 24 hours
- Refresh token expiry: 7 days
```

### Step 3: Configure Google OAuth

1. Go to Google Cloud Console
2. Create new project or select existing
3. Enable "Google+ API"
4. Create OAuth 2.0 credentials (Web application)
5. Add authorized redirect URIs:
   ```
   http://localhost:3000/auth/callback
   https://yourdomain.com/auth/callback
   ```
6. Copy Client ID and Secret
7. In Supabase: Add to Google provider settings

### Step 4: Configure GitHub OAuth

1. Go to GitHub → Settings → Developer settings → OAuth Apps
2. Click "New OAuth App"
3. Fill in:
   - Application name: LifeOS AI
   - Homepage URL: https://yourdomain.com
   - Authorization callback URL: https://yourdomain.com/auth/callback
4. Copy Client ID and Secret
5. In Supabase: Add to GitHub provider settings

---

## ENVIRONMENT VARIABLES

**.env.local** (Frontend & Backend)

```bash
# Supabase (Public - safe to expose)
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key

# Backend Only (Never expose these)
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
JWT_SECRET=your-jwt-secret (optional, for custom tokens)

# Email Service
SENDGRID_API_KEY=your-sendgrid-key

# OAuth
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret (server-only)

GITHUB_CLIENT_ID=your-github-client-id
GITHUB_CLIENT_SECRET=your-github-client-secret (server-only)

# App URLs
NEXT_PUBLIC_APP_URL=http://localhost:3000 (dev)
NEXT_PUBLIC_APP_URL=https://lifeos.ai (prod)
```

**.env.production** (Production only)
```bash
# Same as above but with production URLs
NEXT_PUBLIC_SUPABASE_URL=https://prod-project.supabase.co
NEXT_PUBLIC_APP_URL=https://lifeos.ai
```

---

## CORE AUTHENTICATION FLOWS

### Flow 1: Sign Up (Email/Password)

```
┌─────────────────────────────────────────────┐
│          USER ENTERS EMAIL & PASSWORD       │
└─────────────────────────────────────────────┘
              │
              ▼
┌──────────────────���──────────────────────────┐
│  1. Client: Validate input                  │
│     - Email format valid?                   │
│     - Password >= 8 chars?                  │
│     - Passwords match?                      │
└─────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────┐
│  2. Client: POST /api/auth/signup           │
│     {                                       │
│       "email": "user@example.com",          │
│       "password": "secure123",              │
│       "full_name": "Aria Chen",             │
│       "age": 25                             │
│     }                                       │
└─────────────────────────────────────────────┘
              │
              ▼
┌───────────────────────────────────────��─────┐
│  3. Backend: Validate input                 │
│     - Email not already registered?         │
│     - Password meets requirements?          │
│     - Check rate limiting                   │
└─────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────┐
│  4. Backend: Call Supabase                  │
│     supabase.auth.signUp({                  │
│       email,                                │
│       password,                             │
│       options: {                            │
│         data: { full_name, age }            │
│       }                                     │
│     })                                      │
└─────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────┐
│  5. Supabase: Hash password + Create user   │
│     - User record created                   │
│     - JWT token generated                   │
│     - Refresh token generated               │
│     - Email sent (if enabled)               │
└─────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────┐
│  6. Backend: Create user_profile record     │
│     INSERT INTO user_profiles (             │
│       id (from auth),                       │
│       full_name,                            │
│       email,                                │
│       age,                                  │
│       subscription_tier = 'free'            │
│     )                                       │
└─────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────┐
│  7. Backend: Return to Client               │
��     {                                       │
│       "success": true,                      │
│       "user": { id, email, ... },           │
│       "session": {                          │
│         "access_token": "jwt...",           │
│         "refresh_token": "...",             │
│         "expires_in": 3600                  │
│       }                                     │
│     }                                       │
└─────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────┐
│  8. Client: Store tokens & Redirect         │
│     - localStorage.setItem(                 │
│         'access_token',                     │
│         token                               │
│       )                                     │
│     - Redirect to /onboarding               │
└─────────────────────────────────────────────┘
```

### Flow 2: Login (Email/Password)

```
USER ENTERS EMAIL & PASSWORD
    ↓
CLIENT: Validate input
    ↓
CLIENT: POST /api/auth/login
    ├─ { email, password }
    ↓
BACKEND: Validate input
    ├─ Email exists?
    ├─ Not rate limited?
    ↓
BACKEND: Call Supabase.auth.signInWithPassword()
    ├─ Verify email exists
    ├─ Hash password & compare
    ├─ Generate JWT & refresh token
    ↓
SUPABASE: Return session
    ├─ { access_token, refresh_token, user, expires_in }
    ↓
BACKEND: Return to client
    ├─ Set httpOnly cookie with token
    ├─ Return user data
    ↓
CLIENT: Store tokens
    ├─ Save to localStorage or sessionStorage
    ├─ Update auth state
    ├─ Redirect to /dashboard
    ↓
SUCCESS ✅
```

### Flow 3: Password Reset

```
USER: Enters email and clicks "Forgot Password?"
    ↓
CLIENT: POST /api/auth/forgot-password
    ├─ { email }
    ↓
BACKEND: Validate email
    ├─ Does user exist?
    ├─ Not rate limited?
    ↓
BACKEND: Call Supabase.auth.resetPasswordForEmail()
    ├─ Generate password reset token
    ├─ Send email with reset link
    ↓
CLIENT: Show "Check your email" message
    ↓
USER: Clicks link in email
    ├─ Link format: https://app.com/auth/reset?token=abc123
    ↓
CLIENT: Detects token in URL
    ├─ Renders password reset form
    ↓
USER: Enters new password (twice)
    ↓
CLIENT: POST /api/auth/reset-password
    ├─ { token, new_password }
    ↓
BACKEND: Call Supabase.auth.updateUser()
    ├─ Verify token valid
    ├─ Update password
    ↓
CLIENT: Show success
    ├─ Redirect to login
    ↓
SUCCESS ✅
```

### Flow 4: Social Login (Google/GitHub)

```
USER: Clicks "Login with Google"
    ↓
CLIENT: Call Supabase.auth.signInWithOAuth()
    ├─ provider: 'google'
    ├─ redirectTo: https://app.com/auth/callback
    ↓
SUPABASE: Redirects to Google login
    ↓
GOOGLE: User authenticates
    ├─ Returns authorization code
    ↓
SUPABASE: Exchanges code for Google tokens
    ├─ Gets user info (email, name, avatar)
    ├─ Creates/updates user record
    ├─ Generates JWT
    ↓
SUPABASE: Redirects back to /auth/callback?code=...
    ↓
CLIENT: Handles callback
    ├─ Calls supabase.auth.exchangeCodeForSession()
    ├─ Gets session (access_token, refresh_token)
    ├─ Stores tokens
    ↓
BACKEND: Creates/updates user_profile
    ├─ Check if user exists
    ├─ If new: create profile with 'free' tier
    ├─ If existing: update last_login
    ↓
CLIENT: Redirect to dashboard or onboarding
    ↓
SUCCESS ✅
```

---

## BACKEND IMPLEMENTATION

### Setup Next.js with Supabase

**1. Install Dependencies**

```bash
npm install @supabase/supabase-js @supabase/auth-helpers-nextjs
npm install bcryptjs jsonwebtoken
npm install rate-limiter-flexible  # For rate limiting
```

**2. Create Supabase Client**

**File: `lib/supabase.ts`**

```typescript
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!;
const supabaseKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!;

export const supabaseClient = createClient(supabaseUrl, supabaseKey);

// For server-side only (use service role key for admin operations)
export const supabaseAdmin = createClient(
  supabaseUrl,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);
```

**3. Create Auth Service**

**File: `lib/auth.ts`**

```typescript
import { supabaseClient, supabaseAdmin } from './supabase';
import type { AuthResponse, User } from '@supabase/supabase-js';

interface SignUpPayload {
  email: string;
  password: string;
  full_name: string;
  age: number;
}

interface LoginPayload {
  email: string;
  password: string;
}

interface PasswordResetPayload {
  email: string;
}

export const authService = {
  // SIGNUP
  async signup(payload: SignUpPayload) {
    try {
      // 1. Create auth user
      const { data: authData, error: authError } = await supabaseAdmin.auth.admin.createUser({
        email: payload.email,
        password: payload.password,
        email_confirm: true, // Skip email confirmation for MVP
        user_metadata: {
          full_name: payload.full_name,
          age: payload.age,
        },
      });

      if (authError) throw authError;

      // 2. Create user profile
      const { error: profileError } = await supabaseAdmin
        .from('user_profiles')
        .insert({
          id: authData.user!.id,
          full_name: payload.full_name,
          email: payload.email,
          age: payload.age,
          subscription_tier: 'free',
          profile_complete: false,
          created_at: new Date(),
          updated_at: new Date(),
        });

      if (profileError) throw profileError;

      // 3. Return session
      const { data: sessionData, error: sessionError } = await supabaseClient.auth.signInWithPassword({
        email: payload.email,
        password: payload.password,
      });

      if (sessionError) throw sessionError;

      return {
        success: true,
        user: sessionData.user,
        session: sessionData.session,
      };
    } catch (error) {
      console.error('Signup error:', error);
      throw error;
    }
  },

  // LOGIN
  async login(payload: LoginPayload) {
    try {
      const { data, error } = await supabaseClient.auth.signInWithPassword({
        email: payload.email,
        password: payload.password,
      });

      if (error) throw error;

      return {
        success: true,
        user: data.user,
        session: data.session,
      };
    } catch (error) {
      console.error('Login error:', error);
      throw error;
    }
  },

  // PASSWORD RESET
  async forgotPassword(payload: PasswordResetPayload) {
    try {
      const { error } = await supabaseClient.auth.resetPasswordForEmail(
        payload.email,
        {
          redirectTo: `${process.env.NEXT_PUBLIC_APP_URL}/auth/reset`,
        }
      );

      if (error) throw error;

      return {
        success: true,
        message: 'Password reset email sent',
      };
    } catch (error) {
      console.error('Password reset error:', error);
      throw error;
    }
  },

  // VERIFY TOKEN
  async verifyToken(token: string) {
    try {
      const { data, error } = await supabaseClient.auth.getUser(token);

      if (error) throw error;

      return {
        success: true,
        user: data.user,
      };
    } catch (error) {
      console.error('Token verification error:', error);
      return {
        success: false,
        user: null,
      };
    }
  },

  // REFRESH TOKEN
  async refreshToken(refreshToken: string) {
    try {
      const { data, error } = await supabaseClient.auth.refreshSession({
        refresh_token: refreshToken,
      });

      if (error) throw error;

      return {
        success: true,
        session: data.session,
      };
    } catch (error) {
      console.error('Token refresh error:', error);
      throw error;
    }
  },

  // GET CURRENT USER
  async getCurrentUser(accessToken: string) {
    try {
      const { data, error } = await supabaseClient.auth.getUser(accessToken);

      if (error) throw error;

      // Get extended profile data
      const { data: profile } = await supabaseClient
        .from('user_profiles')
        .select('*')
        .eq('id', data.user!.id)
        .single();

      return {
        success: true,
        user: data.user,
        profile,
      };
    } catch (error) {
      console.error('Get current user error:', error);
      throw error;
    }
  },

  // LOGOUT
  async logout(accessToken: string) {
    try {
      const { error } = await supabaseClient.auth.signOut();

      if (error) throw error;

      return {
        success: true,
      };
    } catch (error) {
      console.error('Logout error:', error);
      throw error;
    }
  },
};
```

### API Endpoints

**File: `pages/api/auth/signup.ts`**

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';
import { authService } from '@/lib/auth';
import { rateLimiter } from '@/lib/rate-limiter';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    // Rate limiting
    await rateLimiter.consume(req.ip, 1);

    const { email, password, full_name, age } = req.body;

    // Validation
    if (!email || !password || !full_name || !age) {
      return res.status(400).json({ error: 'Missing required fields' });
    }

    if (password.length < 8) {
      return res.status(400).json({ error: 'Password must be at least 8 characters' });
    }

    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
      return res.status(400).json({ error: 'Invalid email format' });
    }

    // Call auth service
    const result = await authService.signup({
      email,
      password,
      full_name,
      age,
    });

    // Set httpOnly cookie
    res.setHeader('Set-Cookie', [
      `accessToken=${result.session?.access_token}; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=3600`,
      `refreshToken=${result.session?.refresh_token}; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=604800`,
    ]);

    return res.status(201).json({
      success: true,
      user: result.user,
    });
  } catch (error: any) {
    console.error('Signup API error:', error);

    if (error.message.includes('already registered')) {
      return res.status(409).json({ error: 'Email already registered' });
    }

    return res.status(500).json({ error: 'Signup failed' });
  }
}
```

**File: `pages/api/auth/login.ts`**

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';
import { authService } from '@/lib/auth';
import { rateLimiter } from '@/lib/rate-limiter';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    await rateLimiter.consume(req.ip, 1);

    const { email, password } = req.body;

    if (!email || !password) {
      return res.status(400).json({ error: 'Email and password required' });
    }

    const result = await authService.login({ email, password });

    // Set httpOnly cookies
    res.setHeader('Set-Cookie', [
      `accessToken=${result.session?.access_token}; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=3600`,
      `refreshToken=${result.session?.refresh_token}; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=604800`,
    ]);

    return res.status(200).json({
      success: true,
      user: result.user,
    });
  } catch (error: any) {
    console.error('Login API error:', error);

    if (error.message.includes('Invalid login credentials')) {
      return res.status(401).json({ error: 'Invalid email or password' });
    }

    return res.status(500).json({ error: 'Login failed' });
  }
}
```

**File: `pages/api/auth/logout.ts`**

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  // Clear cookies
  res.setHeader('Set-Cookie', [
    'accessToken=; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=0',
    'refreshToken=; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=0',
  ]);

  return res.status(200).json({ success: true });
}
```

**File: `pages/api/auth/me.ts`**

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';
import { withAuth } from '@/middleware/auth';

async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'GET') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    // User is attached by middleware
    const user = (req as any).user;

    return res.status(200).json({
      success: true,
      user,
    });
  } catch (error) {
    return res.status(500).json({ error: 'Failed to get current user' });
  }
}

export default withAuth(handler);
```

**File: `pages/api/auth/forgot-password.ts`**

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';
import { authService } from '@/lib/auth';
import { rateLimiter } from '@/lib/rate-limiter';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    await rateLimiter.consume(req.ip, 1);

    const { email } = req.body;

    if (!email) {
      return res.status(400).json({ error: 'Email required' });
    }

    const result = await authService.forgotPassword({ email });

    return res.status(200).json(result);
  } catch (error) {
    console.error('Forgot password error:', error);
    // Don't reveal if email exists (security)
    return res.status(200).json({
      success: true,
      message: 'If email exists, password reset link sent',
    });
  }
}
```

### Middleware: JWT Verification

**File: `middleware/auth.ts`**

```typescript
import { NextApiRequest, NextApiResponse } from 'next';
import { supabaseClient } from '@/lib/supabase';

export function withAuth(
  handler: (req: NextApiRequest, res: NextApiResponse) => Promise<void>
) {
  return async (req: NextApiRequest, res: NextApiResponse) => {
    try {
      // Get token from cookies or Authorization header
      const token =
        req.cookies.accessToken ||
        req.headers.authorization?.replace('Bearer ', '');

      if (!token) {
        return res.status(401).json({ error: 'Unauthorized' });
      }

      // Verify token with Supabase
      const { data, error } = await supabaseClient.auth.getUser(token);

      if (error || !data.user) {
        return res.status(401).json({ error: 'Invalid token' });
      }

      // Attach user to request
      (req as any).user = data.user;
      (req as any).userId = data.user.id;

      // Call handler
      return handler(req, res);
    } catch (error) {
      console.error('Auth middleware error:', error);
      return res.status(500).json({ error: 'Authentication failed' });
    }
  };
}
```

### Rate Limiting

**File: `lib/rate-limiter.ts`**

```typescript
import { RateLimiterMemory } from 'rate-limiter-flexible';

// Allow 5 requests per 15 minutes per IP
const limiter = new RateLimiterMemory({
  points: 5,
  duration: 15 * 60, // 15 minutes
});

export const rateLimiter = {
  async consume(ip: string | string[] | undefined, points: number = 1) {
    if (!ip) ip = 'unknown';
    if (Array.isArray(ip)) ip = ip[0];

    try {
      await limiter.consume(ip, points);
    } catch (error) {
      throw new Error('Too many requests. Please try again later.');
    }
  },
};
```

---

## FRONTEND IMPLEMENTATION

### React Context for Auth State

**File: `context/AuthContext.tsx`**

```typescript
import React, { createContext, useContext, useState, useEffect } from 'react';
import type { User } from '@supabase/supabase-js';

interface AuthContextType {
  user: User | null;
  isLoading: boolean;
  isAuthenticated: boolean;
  signup: (email: string, password: string, full_name: string, age: number) => Promise<void>;
  login: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
  forgotPassword: (email: string) => Promise<void>;
  error: string | null;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  // Check if user is already logged in (on mount)
  useEffect(() => {
    const checkAuth = async () => {
      try {
        const response = await fetch('/api/auth/me');
        if (response.ok) {
          const { user } = await response.json();
          setUser(user);
        }
      } catch (err) {
        console.error('Auth check error:', err);
      } finally {
        setIsLoading(false);
      }
    };

    checkAuth();
  }, []);

  const signup = async (
    email: string,
    password: string,
    full_name: string,
    age: number
  ) => {
    setIsLoading(true);
    setError(null);

    try {
      const response = await fetch('/api/auth/signup', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password, full_name, age }),
      });

      if (!response.ok) {
        const { error } = await response.json();
        throw new Error(error);
      }

      const { user } = await response.json();
      setUser(user);
    } catch (err: any) {
      setError(err.message);
      throw err;
    } finally {
      setIsLoading(false);
    }
  };

  const login = async (email: string, password: string) => {
    setIsLoading(true);
    setError(null);

    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password }),
      });

      if (!response.ok) {
        const { error } = await response.json();
        throw new Error(error);
      }

      const { user } = await response.json();
      setUser(user);
    } catch (err: any) {
      setError(err.message);
      throw err;
    } finally {
      setIsLoading(false);
    }
  };

  const logout = async () => {
    setIsLoading(true);

    try {
      await fetch('/api/auth/logout', { method: 'POST' });
      setUser(null);
    } catch (err: any) {
      setError(err.message);
    } finally {
      setIsLoading(false);
    }
  };

  const forgotPassword = async (email: string) => {
    setIsLoading(true);
    setError(null);

    try {
      const response = await fetch('/api/auth/forgot-password', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email }),
      });

      if (!response.ok) {
        throw new Error('Failed to send reset email');
      }
    } catch (err: any) {
      setError(err.message);
      throw err;
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <AuthContext.Provider
      value={{
        user,
        isLoading,
        isAuthenticated: !!user,
        signup,
        login,
        logout,
        forgotPassword,
        error,
      }}
    >
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}
```

### Login Component

**File: `components/auth/LoginForm.tsx`**

```typescript
import { useState } from 'react';
import { useRouter } from 'next/router';
import { useAuth } from '@/context/AuthContext';

export function LoginForm() {
  const router = useRouter();
  const { login, error: authError } = useAuth();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [isLoading, setIsLoading] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError('');
    setIsLoading(true);

    try {
      await login(email, password);
      router.push('/dashboard');
    } catch (err: any) {
      setError(err.message || 'Login failed');
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <label htmlFor="email" className="block text-sm font-medium mb-1">
          Email
        </label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          className="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
          required
          disabled={isLoading}
        />
      </div>

      <div>
        <label htmlFor="password" className="block text-sm font-medium mb-1">
          Password
        </label>
        <input
          id="password"
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          className="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
          required
          disabled={isLoading}
        />
      </div>

      {(error || authError) && (
        <div className="p-3 bg-red-50 border border-red-200 rounded-lg text-red-700 text-sm">
          {error || authError}
        </div>
      )}

      <button
        type="submit"
        disabled={isLoading}
        className="w-full bg-blue-600 text-white py-2 rounded-lg font-medium hover:bg-blue-700 disabled:opacity-50"
      >
        {isLoading ? 'Logging in...' : 'Login'}
      </button>

      <a
        href="/auth/forgot-password"
        className="block text-center text-blue-600 text-sm hover:underline"
      >
        Forgot password?
      </a>
    </form>
  );
}
```

### Sign Up Component

**File: `components/auth/SignupForm.tsx`**

```typescript
import { useState } from 'react';
import { useRouter } from 'next/router';
import { useAuth } from '@/context/AuthContext';

export function SignupForm() {
  const router = useRouter();
  const { signup, error: authError } = useAuth();
  const [formData, setFormData] = useState({
    full_name: '',
    email: '',
    password: '',
    password_confirm: '',
    age: '',
  });
  const [error, setError] = useState('');
  const [isLoading, setIsLoading] = useState(false);

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData((prev) => ({ ...prev, [name]: value }));
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError('');
    setIsLoading(true);

    // Validate
    if (formData.password !== formData.password_confirm) {
      setError('Passwords do not match');
      setIsLoading(false);
      return;
    }

    if (formData.password.length < 8) {
      setError('Password must be at least 8 characters');
      setIsLoading(false);
      return;
    }

    try {
      await signup(
        formData.email,
        formData.password,
        formData.full_name,
        parseInt(formData.age)
      );
      router.push('/onboarding');
    } catch (err: any) {
      setError(err.message || 'Signup failed');
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <label className="block text-sm font-medium mb-1">Full Name</label>
        <input
          type="text"
          name="full_name"
          value={formData.full_name}
          onChange={handleChange}
          className="w-full px-4 py-2 border rounded-lg"
          required
          disabled={isLoading}
        />
      </div>

      <div>
        <label className="block text-sm font-medium mb-1">Email</label>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          className="w-full px-4 py-2 border rounded-lg"
          required
          disabled={isLoading}
        />
      </div>

      <div>
        <label className="block text-sm font-medium mb-1">Age</label>
        <input
          type="number"
          name="age"
          value={formData.age}
          onChange={handleChange}
          className="w-full px-4 py-2 border rounded-lg"
          min="13"
          max="150"
          required
          disabled={isLoading}
        />
      </div>

      <div>
        <label className="block text-sm font-medium mb-1">Password</label>
        <input
          type="password"
          name="password"
          value={formData.password}
          onChange={handleChange}
          className="w-full px-4 py-2 border rounded-lg"
          minLength={8}
          required
          disabled={isLoading}
        />
        <p className="text-xs text-gray-500 mt-1">At least 8 characters</p>
      </div>

      <div>
        <label className="block text-sm font-medium mb-1">Confirm Password</label>
        <input
          type="password"
          name="password_confirm"
          value={formData.password_confirm}
          onChange={handleChange}
          className="w-full px-4 py-2 border rounded-lg"
          required
          disabled={isLoading}
        />
      </div>

      {(error || authError) && (
        <div className="p-3 bg-red-50 border border-red-200 rounded-lg text-red-700 text-sm">
          {error || authError}
        </div>
      )}

      <button
        type="submit"
        disabled={isLoading}
        className="w-full bg-blue-600 text-white py-2 rounded-lg font-medium hover:bg-blue-700 disabled:opacity-50"
      >
        {isLoading ? 'Creating account...' : 'Create Account'}
      </button>
    </form>
  );
}
```

### Protected Routes

**File: `components/ProtectedRoute.tsx`**

```typescript
import { ReactNode } from 'react';
import { useRouter } from 'next/router';
import { useAuth } from '@/context/AuthContext';

interface ProtectedRouteProps {
  children: ReactNode;
}

export function ProtectedRoute({ children }: ProtectedRouteProps) {
  const router = useRouter();
  const { isAuthenticated, isLoading } = useAuth();

  if (isLoading) {
    return <div className="flex items-center justify-center h-screen">Loading...</div>;
  }

  if (!isAuthenticated) {
    router.push('/auth/login');
    return null;
  }

  return <>{children}</>;
}
```

---

## SECURITY BEST PRACTICES

### ✅ Do's

```
✓ Always use HTTPS in production
✓ Store sensitive tokens in httpOnly cookies
✓ Validate input on both client and server
✓ Use strong password requirements (min 8 chars)
✓ Implement rate limiting
✓ Hash passwords with bcrypt (Supabase does this)
✓ Use JWT with short expiration (1 hour)
✓ Rotate refresh tokens
✓ Log authentication events
✓ Use environment variables for secrets
✓ Implement CORS properly
✓ Add CSRF protection for POST requests
```

### ❌ Don'ts

```
✗ Never store tokens in localStorage (vulnerable to XSS)
✗ Never expose secrets in client-side code
✗ Never log passwords or sensitive data
✗ Never trust client-side validation alone
✗ Never use weak passwords
✗ Never skip email verification for sensitive changes
✗ Never allow unlimited login attempts
✗ Don't expose whether email exists (info leak)
✗ Don't store unencrypted PII
✗ Don't allow sessions to last too long
```

### CSRF Protection

```typescript
// Add to API routes
import csrf from 'csrf';

const tokens = new csrf();

// Generate token in form:
const csrfToken = tokens.create(secret);

// Verify in POST request:
if (!tokens.verify(secret, token)) {
  return res.status(403).json({ error: 'CSRF token invalid' });
}
```

### CORS Configuration

```typescript
// In next.config.js or API middleware
const allowedOrigins = [
  'http://localhost:3000',
  'https://lifeos.ai',
  'https://www.lifeos.ai',
];

export function corsMiddleware(req: NextApiRequest, res: NextApiResponse) {
  const origin = req.headers.origin;
  
  if (allowedOrigins.includes(origin || '')) {
    res.setHeader('Access-Control-Allow-Origin', origin);
    res.setHeader('Access-Control-Allow-Methods', 'GET,POST,PUT,DELETE,OPTIONS');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type,Authorization');
    res.setHeader('Access-Control-Allow-Credentials', 'true');
  }

  if (req.method === 'OPTIONS') {
    return res.status(200).end();
  }
}
```

---

## TESTING CHECKLIST

### Unit Tests

```typescript
// Test signup with valid input
// Test signup with invalid email
// Test signup with weak password
// Test signup with duplicate email
// Test login with correct credentials
// Test login with wrong password
// Test password reset flow
// Test token refresh
// Test logout
```

### Integration Tests

```typescript
// Sign up → Verify email → Login → Logout
// Login → Access protected route → Logout
// Forgot password → Reset → Login with new password
// Social OAuth flow
// Session expiry & refresh
```

### Security Tests

```typescript
// SQL injection attempts
// XSS in email/name fields
// CSRF token validation
// Rate limiting (5+ rapid requests)
// Token tampering
// Expired token handling
// Invalid token handling
```

---

## DEPLOYMENT CHECKLIST

```
BEFORE GOING LIVE:

Backend:
[ ] Set all environment variables
[ ] Configure production Supabase project
[ ] Enable email verification (optional)
[ ] Set up email service (SendGrid)
[ ] Test OAuth with production URLs
[ ] Enable HTTPS everywhere
[ ] Set secure cookies (Secure, HttpOnly, SameSite)
[ ] Configure CORS properly
[ ] Set up error logging (Sentry)
[ ] Test rate limiting under load

Frontend:
[ ] Set NEXT_PUBLIC_APP_URL to production
[ ] Disable console logs in production
[ ] Test all auth flows in production
[ ] Verify redirect URLs
[ ] Test on mobile devices
[ ] Verify password reset emails

Security:
[ ] Security audit with OWASP checklist
[ ] Penetration testing (optional)
[ ] Check for exposed secrets
[ ] Enable WAF (Web Application Firewall)
[ ] Monitor for suspicious login attempts
```

---

## TROUBLESHOOTING

### Common Issues

```
ISSUE: "Invalid login credentials"
FIX: Check email exists & password is correct

ISSUE: "Token expired"
FIX: Implement token refresh logic

ISSUE: "CORS error"
FIX: Verify allowed origins in CORS config

ISSUE: "Email not received"
FIX: Check SendGrid quota, verify email in dashboard

ISSUE: "OAuth callback fails"
FIX: Verify redirect URIs match exactly (including protocol & domain)

ISSUE: "User profile not created after signup"
FIX: Check transaction logic, ensure user created before profile insert
```

---

## NEXT STEPS

### Immediate (This Week)
1. [ ] Set up Supabase project
2. [ ] Configure OAuth providers (Google, GitHub)
3. [ ] Implement backend auth endpoints
4. [ ] Implement frontend auth components
5. [ ] Test all auth flows locally

### Following Week
1. [ ] User acceptance testing with beta users
2. [ ] Security audit
3. [ ] Performance testing
4. [ ] Deploy to staging
5. [ ] Final security review

---

**Document Owner:** Backend Lead + Security Engineer  
**Last Updated:** July 2026  
**Next Review:** August 2026  
**Status:** Ready for Implementation
