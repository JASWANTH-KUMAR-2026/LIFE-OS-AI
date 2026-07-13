# LifeOS AI - Technical Architecture

## System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     Client Layer                             │
│              Next.js Web App (Vercel CDN)                   │
└────────────────────────┬────────────────────────────────────┘
                         │
         ┌───────────────┴───────────────┐
         │                               │
    ┌────▼────────┐          ┌──────────▼──────┐
    │  REST API   │          │  WebSocket      │
    │  /api/*     │          │  Real-time      │
    └────┬────────┘          └──────────┬──────┘
         │                              │
┌────────┴──────────────────────────────┴────────┐
│          Node.js API Server (Express)         │
│                                                 │
│  ├─ Authentication Routes                      │
│  ├─ User Profile Routes                        │
│  ├─ Goal Management Routes                     │
│  ├─ AI Mentor Routes                           │
│  ├─ Habit Tracking Routes                      │
│  ├─ Productivity Routes                        │
│  └─ Middleware (Auth, Logging, Validation)    │
└────────┬───────────────────────────────────────┘
         │
    ┌────┴─────────┬──────────────┬──────────────┐
    │              │              │              │
┌───▼────┐  ┌─────▼──┐  ┌────────▼─┐  ┌──────▼──┐
│PostgreSQL     │Redis  │  │OpenAI   │  │AWS S3   │
│Database      │Cache  │  │API      │  │Storage  │
└──────────┘  └────────┘  └─────────┘  └─────────┘
```

## Database Schema

### Core Models

#### Users
- Authentication & profile management
- Subscription tier
- Account settings

#### Profiles
- User's personal information
- Goals summary
- Progress overview

#### Goals
- Goal title, description, deadline
- Priority, status, progress
- Milestones & tasks

#### Milestones
- Sub-goals within a goal
- Timeline & progress tracking
- Task breakdown

#### Tasks
- Actionable items from milestones
- Daily/weekly organization
- Completion status

#### Habits
- Habit definition & frequency
- Tracking rules
- Category (fitness, learning, etc.)

#### HabitLogs
- Daily completion records
- Streak calculation
- Analytics data

#### Conversations
- AI mentor chat sessions
- Context & references
- Timestamps

#### Messages
- Individual chat messages
- Sender (user/ai)
- Embeddings for semantic search

#### Memories
- Long-term memory storage
- Preferences, insights, lessons
- Auto-generated summaries

#### Roadmaps
- AI-generated learning paths
- Monthly/weekly breakdown
- Resources & checkpoints

#### ProgressReports
- Weekly/monthly analytics
- Goal progress snapshots
- Habit streaks, insights

## API Design

### Authentication
```
POST /api/auth/register
POST /api/auth/login
POST /api/auth/logout
POST /api/auth/refresh
GET  /api/auth/session
```

### User Profile
```
GET    /api/users/profile
PUT    /api/users/profile
GET    /api/users/settings
PUT    /api/users/settings
```

### Goals
```
GET    /api/goals
POST   /api/goals
GET    /api/goals/:id
PUT    /api/goals/:id
DELETE /api/goals/:id
POST   /api/goals/:id/milestones
```

### AI Mentor
```
POST /api/mentor/chat
GET  /api/mentor/history
GET  /api/mentor/context
```

### Habits
```
GET    /api/habits
POST   /api/habits
POST   /api/habits/:id/log
GET    /api/habits/analytics
```

## Security Architecture

### Authentication
- NextAuth.js with JWT
- Secure session management
- CSRF protection
- Rate limiting

### Authorization
- Role-based access control (RBAC)
- User data isolation
- Subscription-based feature gates

### Data Protection
- Bcrypt password hashing
- Encrypted sensitive fields
- HTTPS only
- SQL injection prevention (Prisma)
- XSS protection via React

### API Security
- API rate limiting
- Request validation
- CORS configuration
- API key management

## Scalability Strategy

### Database
- PostgreSQL with connection pooling
- Indexes on frequently queried fields
- Read replicas for analytics

### Caching
- Redis for session storage
- Response caching for frequently accessed data
- OpenAI response caching

### CDN
- Vercel Edge Network for static assets
- Geo-distributed content delivery

### Background Jobs
- Job queue for AI mentor interactions
- Habit streak calculations
- Report generation
- Email notifications

## Deployment

### Development
- Local PostgreSQL + Redis
- Vercel local development
- Hot reload enabled

### Staging
- Vercel Preview Deployments
- Staging database replica
- Feature flag testing

### Production
- Vercel Production deployment
- AWS RDS PostgreSQL
- AWS ElastiCache Redis
- CloudFlare CDN
- Sentry error tracking

## Monitoring & Analytics

### Performance
- Vercel Analytics
- Core Web Vitals tracking
- API response time monitoring

### Errors
- Sentry integration
- Error grouping & alerting
- Stack trace analysis

### Usage Analytics
- Custom event tracking
- User behavior analysis
- Feature usage metrics

## Roadmap Integration

### Phase 1
- User auth & profile
- Database setup
- Basic API structure

### Phase 2
- OpenAI integration
- Chat interface
- Context management

### Phase 3
- Goal CRUD operations
- Progress tracking
- Milestone management

### Phase 4
- AI roadmap generation
- Learning path creation
- Resource recommendations

### Phase 5
- Habit creation & tracking
- Streak calculation
- Analytics dashboard

### Phase 6
- Dashboard UI
- Task calendar
- Progress visualizations

### Phase 7
- Note storage
- Knowledge organization
- Full-text search

### Phase 8
- Memory optimization
- Context-aware responses
- Personalization engine
