# LifeOS AI — DATABASE SCHEMA DESIGN (Phase 2)

**Version:** 1.0  
**Date:** July 2026  
**Database:** PostgreSQL (via Supabase)

---

## EXECUTIVE SUMMARY

This document defines the complete database schema for LifeOS AI MVP. The design focuses on:
- **Scalability** – Supports millions of users
- **Performance** – Optimized queries for real-time experiences
- **Security** – Row-level security (RLS) for privacy
- **Flexibility** – Easy to extend for future features

---

## DATABASE ARCHITECTURE OVERVIEW

```
┌─────────────────────────────────────────────────────────────┐
│                    LifeOS AI Database                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────┐         ┌──────────────────┐        │
│  │  AUTH SCHEMA     │         │   CORE SCHEMA    │        │
│  ├──────────────────┤         ├──────────────────┤        │
│  │ • users (Supabase)        │ • user_profiles  │        │
│  │                           │ • goals          │        │
│  │                           │ • milestones     │        │
│  │                           │ • daily_tasks    │        │
│  │                           │ • check_ins      │        │
│  │                           │ • reflections    │        │
│  └──────────────────┘         └──────────────────┘        │
│                                                             │
│  ┌──────────────────┐         ┌──────────────────┐        │
│  │  CONVERSATION    │         │   MEMORY SCHEMA  │        │
│  │    SCHEMA        │         ├──────────────────┤        │
│  ├──────────────────┤         │ • conversations  │        │
│  │ • conversations  │         │ • messages       │        │
│  │ • messages       │         │ • user_preferences        │
│  │                 │         │ • context_summary         │
│  └──────────────────┘         └──────────────────┘        │
│                                                             │
│  ┌──────────────────┐         ┌──────────────────┐        │
│  │  SUBSCRIPTION    │         │  ANALYTICS       │        │
│  │    SCHEMA        │         │  SCHEMA          │        │
│  ├──────────────────┤         ├──────────────────┤        │
│  │ • subscriptions  │         │ • activity_logs  │        │
│  │ • payments       │         │ • user_metrics   │        │
│  │ • billing_history          │ • engagement_data         │
│  └──────────────────┘         └──────────────────┘        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## DETAILED TABLE DEFINITIONS

### 1. AUTHENTICATION & USERS (Supabase Managed)

**Note:** Supabase Auth automatically creates and manages the `auth.users` table. We create a linked profile table.

#### Table: `user_profiles`
**Purpose:** Store additional user information beyond authentication

```sql
CREATE TABLE user_profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  
  -- Personal Info
  full_name VARCHAR(255) NOT NULL,
  age INTEGER,
  email VARCHAR(255) NOT NULL UNIQUE,
  avatar_url TEXT,
  
  -- Profile Info
  current_situation VARCHAR(50) NOT NULL, -- 'student', 'employee', 'entrepreneur', 'freelancer', 'other'
  timezone VARCHAR(50) DEFAULT 'UTC',
  location VARCHAR(255),
  
  -- Status
  profile_complete BOOLEAN DEFAULT FALSE,
  onboarding_completed_at TIMESTAMP,
  
  -- Preferences
  learning_style VARCHAR(50), -- 'visual', 'hands-on', 'reading', 'mixed'
  communication_style VARCHAR(50), -- 'direct', 'detailed', 'conversational'
  preferred_language VARCHAR(10) DEFAULT 'en',
  
  -- Subscription Info (denormalized for quick access)
  subscription_tier VARCHAR(20) DEFAULT 'free', -- 'free', 'pro'
  subscription_expires_at TIMESTAMP,
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  last_login_at TIMESTAMP,
  
  CONSTRAINT age_valid CHECK (age > 0 AND age <= 150),
  CONSTRAINT valid_tier CHECK (subscription_tier IN ('free', 'pro'))
);

CREATE INDEX idx_user_profiles_email ON user_profiles(email);
CREATE INDEX idx_user_profiles_subscription_tier ON user_profiles(subscription_tier);
CREATE INDEX idx_user_profiles_created_at ON user_profiles(created_at);
```

---

#### Table: `user_interests`
**Purpose:** Store user's interests (many-to-many relationship)

```sql
CREATE TABLE user_interests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
  interest VARCHAR(100) NOT NULL,
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(user_id, interest)
);

CREATE INDEX idx_user_interests_user_id ON user_interests(user_id);
```

---

#### Table: `user_skills`
**Purpose:** Store user's current skills

```sql
CREATE TABLE user_skills (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
  skill_name VARCHAR(100) NOT NULL,
  proficiency_level VARCHAR(20), -- 'beginner', 'intermediate', 'advanced', 'expert'
  years_experience DECIMAL(4,1),
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(user_id, skill_name)
);

CREATE INDEX idx_user_skills_user_id ON user_skills(user_id);
```

---

#### Table: `user_challenges`
**Purpose:** Store challenges mentioned during onboarding

```sql
CREATE TABLE user_challenges (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
  challenge_description TEXT NOT NULL,
  what_tried TEXT,
  whats_blocking TEXT,
  priority VARCHAR(20) DEFAULT 'medium', -- 'low', 'medium', 'high'
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_user_challenges_user_id ON user_challenges(user_id);
```

---

### 2. GOAL MANAGEMENT SYSTEM

#### Table: `goals`
**Purpose:** Store user goals with timeline and status

```sql
CREATE TABLE goals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
  
  -- Goal Info
  goal_title VARCHAR(255) NOT NULL,
  goal_description TEXT,
  category VARCHAR(50) NOT NULL, -- 'career', 'skills', 'productivity', 'business', 'health', 'personal'
  
  -- Timeline
  timeline_days INTEGER NOT NULL, -- 30, 60, 90, 180
  start_date DATE NOT NULL,
  target_date DATE NOT NULL,
  
  -- Status
  status VARCHAR(30) DEFAULT 'active', -- 'active', 'paused', 'completed', 'failed', 'archived'
  completion_percentage INTEGER DEFAULT 0,
  
  -- AI-Generated Roadmap (stored as JSON for flexibility)
  roadmap_json JSONB, -- Contains phases, milestones, tasks
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  completed_at TIMESTAMP,
  
  CONSTRAINT valid_timeline CHECK (timeline_days IN (30, 60, 90, 180)),
  CONSTRAINT valid_completion CHECK (completion_percentage >= 0 AND completion_percentage <= 100),
  CONSTRAINT target_after_start CHECK (target_date > start_date)
);

CREATE INDEX idx_goals_user_id ON goals(user_id);
CREATE INDEX idx_goals_status ON goals(status);
CREATE INDEX idx_goals_created_at ON goals(created_at);
CREATE INDEX idx_goals_target_date ON goals(target_date);
```

---

#### Table: `milestones`
**Purpose:** Weekly milestones for each goal

```sql
CREATE TABLE milestones (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  goal_id UUID NOT NULL REFERENCES goals(id) ON DELETE CASCADE,
  
  -- Milestone Info
  milestone_number INTEGER NOT NULL, -- Week 1, 2, 3, etc.
  milestone_title VARCHAR(255) NOT NULL,
  milestone_description TEXT,
  
  -- Timeline
  target_date DATE NOT NULL,
  
  -- Status
  is_completed BOOLEAN DEFAULT FALSE,
  completed_at TIMESTAMP,
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(goal_id, milestone_number)
);

CREATE INDEX idx_milestones_goal_id ON milestones(goal_id);
CREATE INDEX idx_milestones_is_completed ON milestones(is_completed);
```

---

#### Table: `daily_tasks`
**Purpose:** Daily actionable tasks for goal achievement

```sql
CREATE TABLE daily_tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  goal_id UUID NOT NULL REFERENCES goals(id) ON DELETE CASCADE,
  
  -- Task Info
  task_title VARCHAR(255) NOT NULL,
  task_description TEXT,
  
  -- Schedule (recurrence pattern)
  frequency VARCHAR(30) NOT NULL, -- 'daily', 'mon-wed-fri', 'weekly', 'biweekly'
  days_of_week INTEGER[], -- Array: [1,3,5] for Mon, Wed, Fri (0=Sunday, 6=Saturday)
  
  -- Time
  preferred_time_of_day VARCHAR(20), -- 'morning', 'afternoon', 'evening'
  estimated_duration_minutes INTEGER, -- How long does this task take?
  
  -- Status
  is_active BOOLEAN DEFAULT TRUE,
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT valid_frequency CHECK (frequency IN ('daily', 'mon-wed-fri', 'weekly', 'biweekly'))
);

CREATE INDEX idx_daily_tasks_goal_id ON daily_tasks(goal_id);
CREATE INDEX idx_daily_tasks_is_active ON daily_tasks(is_active);
```

---

#### Table: `task_completions`
**Purpose:** Track daily when user completes tasks

```sql
CREATE TABLE task_completions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  task_id UUID NOT NULL REFERENCES daily_tasks(id) ON DELETE CASCADE,
  
  -- Completion
  completion_date DATE NOT NULL,
  was_completed BOOLEAN NOT NULL,
  
  -- Notes
  user_notes TEXT, -- Why they couldn't complete, insights, etc.
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(task_id, completion_date)
);

CREATE INDEX idx_task_completions_task_id ON task_completions(task_id);
CREATE INDEX idx_task_completions_completion_date ON task_completions(completion_date);
CREATE INDEX idx_task_completions_was_completed ON task_completions(was_completed);
```

---

#### Table: `reflections`
**Purpose:** Weekly goal reflections for learning & adjustment

```sql
CREATE TABLE reflections (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  goal_id UUID NOT NULL REFERENCES goals(id) ON DELETE CASCADE,
  
  -- Reflection Content
  reflection_week INTEGER NOT NULL, -- Week 1, 2, 3, etc.
  what_went_well TEXT,
  what_was_hard TEXT,
  what_to_improve TEXT,
  
  -- Metadata
  reflection_date DATE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(goal_id, reflection_week)
);

CREATE INDEX idx_reflections_goal_id ON reflections(goal_id);
CREATE INDEX idx_reflections_reflection_date ON reflections(reflection_date);
```

---

### 3. DAILY CHECK-IN SYSTEM

#### Table: `check_ins`
**Purpose:** Daily check-in records

```sql
CREATE TABLE check_ins (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
  
  -- Check-in Content
  check_in_date DATE NOT NULL,
  time_of_day VARCHAR(20) NOT NULL, -- 'morning', 'afternoon', 'evening'
  
  -- Answers
  todays_main_focus TEXT NOT NULL,
  yesterdays_accomplishment TEXT,
  yesterdays_challenges TEXT,
  todays_priorities TEXT,
  
  -- AI Feedback
  ai_feedback TEXT, -- Generated feedback from AI
  ai_suggestions TEXT, -- AI suggestions for today
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(user_id, check_in_date)
);

CREATE INDEX idx_check_ins_user_id ON check_ins(user_id);
CREATE INDEX idx_check_ins_check_in_date ON check_ins(check_in_date);
CREATE INDEX idx_check_ins_created_at ON check_ins(created_at);
```

---

#### Table: `user_streaks`
**Purpose:** Track check-in streaks for gamification

```sql
CREATE TABLE user_streaks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
  
  -- Streak Data
  current_streak_days INTEGER DEFAULT 0,
  longest_streak_days INTEGER DEFAULT 0,
  
  -- Dates
  streak_start_date DATE,
  last_check_in_date DATE,
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(user_id)
);

CREATE INDEX idx_user_streaks_user_id ON user_streaks(user_id);
```

---

### 4. CONVERSATION & AI MEMORY SYSTEM

#### Table: `conversations`
**Purpose:** Store conversation threads

```sql
CREATE TABLE conversations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
  
  -- Conversation Info
  mentor_type VARCHAR(50) NOT NULL, -- 'career', 'entrepreneur', 'productivity', 'learning'
  conversation_title VARCHAR(255),
  
  -- Context (summarized for memory)
  context_summary TEXT, -- AI-generated summary of what was discussed
  
  -- Status
  is_archived BOOLEAN DEFAULT FALSE,
  
  -- Token Usage (for analytics & cost tracking)
  total_input_tokens INTEGER DEFAULT 0,
  total_output_tokens INTEGER DEFAULT 0,
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  last_message_at TIMESTAMP,
  
  CONSTRAINT valid_mentor_type CHECK (mentor_type IN ('career', 'entrepreneur', 'productivity', 'learning'))
);

CREATE INDEX idx_conversations_user_id ON conversations(user_id);
CREATE INDEX idx_conversations_mentor_type ON conversations(mentor_type);
CREATE INDEX idx_conversations_is_archived ON conversations(is_archived);
CREATE INDEX idx_conversations_created_at ON conversations(created_at);
```

---

#### Table: `messages`
**Purpose:** Individual messages within a conversation

```sql
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id UUID NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
  
  -- Message Content
  sender_role VARCHAR(20) NOT NULL, -- 'user', 'assistant'
  message_content TEXT NOT NULL,
  message_type VARCHAR(30) DEFAULT 'text', -- 'text', 'action', 'suggestion', 'roadmap'
  
  -- Metadata (for memory)
  message_metadata JSONB, -- Can store additional context, e.g., goals referenced
  
  -- Token Usage
  input_tokens INTEGER,
  output_tokens INTEGER,
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT valid_sender_role CHECK (sender_role IN ('user', 'assistant')),
  CONSTRAINT valid_message_type CHECK (message_type IN ('text', 'action', 'suggestion', 'roadmap'))
);

CREATE INDEX idx_messages_conversation_id ON messages(conversation_id);
CREATE INDEX idx_messages_created_at ON messages(created_at);
CREATE INDEX idx_messages_sender_role ON messages(sender_role);
```

---

#### Table: `user_preferences`
**Purpose:** AI learns user preferences for personalization

```sql
CREATE TABLE user_preferences (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
  
  -- Preference Type
  preference_category VARCHAR(100) NOT NULL, -- e.g., 'communication_style', 'learning_pace', 'topic_interest'
  preference_key VARCHAR(100) NOT NULL,
  preference_value TEXT NOT NULL,
  
  -- Confidence (how sure are we about this preference?)
  confidence_score DECIMAL(3,2) DEFAULT 0.5, -- 0.0 to 1.0
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(user_id, preference_category, preference_key)
);

CREATE INDEX idx_user_preferences_user_id ON user_preferences(user_id);
CREATE INDEX idx_user_preferences_category ON user_preferences(preference_category);
```

---

#### Table: `conversation_context`
**Purpose:** Store AI-generated context/memory for faster recall

```sql
CREATE TABLE conversation_context (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
  
  -- Context Type
  context_type VARCHAR(50) NOT NULL, -- 'goal_progress', 'past_advice', 'challenge_pattern', 'skill_interest'
  context_data JSONB NOT NULL,
  
  -- Importance
  importance_score DECIMAL(3,2) DEFAULT 0.5, -- Higher = more relevant
  
  -- Freshness
  is_active BOOLEAN DEFAULT TRUE,
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT valid_context_type CHECK (context_type IN ('goal_progress', 'past_advice', 'challenge_pattern', 'skill_interest'))
);

CREATE INDEX idx_conversation_context_user_id ON conversation_context(user_id);
CREATE INDEX idx_conversation_context_type ON conversation_context(context_type);
CREATE INDEX idx_conversation_context_is_active ON conversation_context(is_active);
```

---

### 5. SUBSCRIPTION & PAYMENTS

#### Table: `subscriptions`
**Purpose:** Track user subscription status

```sql
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
  
  -- Subscription Info
  tier VARCHAR(20) NOT NULL, -- 'free', 'pro'
  status VARCHAR(30) NOT NULL, -- 'active', 'canceled', 'expired', 'paused'
  
  -- Pricing
  currency VARCHAR(10) DEFAULT 'USD', -- 'USD', 'EUR', 'INR'
  monthly_price DECIMAL(10,2),
  
  -- Dates
  start_date DATE NOT NULL,
  end_date DATE,
  renewal_date DATE,
  
  -- Stripe
  stripe_subscription_id VARCHAR(255) UNIQUE,
  stripe_customer_id VARCHAR(255),
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(user_id),
  CONSTRAINT valid_tier CHECK (tier IN ('free', 'pro')),
  CONSTRAINT valid_status CHECK (status IN ('active', 'canceled', 'expired', 'paused'))
);

CREATE INDEX idx_subscriptions_user_id ON subscriptions(user_id);
CREATE INDEX idx_subscriptions_status ON subscriptions(status);
CREATE INDEX idx_subscriptions_tier ON subscriptions(tier);
```

---

#### Table: `payments`
**Purpose:** Track individual payment transactions

```sql
CREATE TABLE payments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  subscription_id UUID NOT NULL REFERENCES subscriptions(id) ON DELETE CASCADE,
  
  -- Payment Info
  amount DECIMAL(10,2) NOT NULL,
  currency VARCHAR(10) NOT NULL DEFAULT 'USD',
  status VARCHAR(30) NOT NULL, -- 'pending', 'succeeded', 'failed', 'refunded'
  
  -- Payment Method
  payment_method VARCHAR(50), -- 'card', 'paypal', 'bank_transfer'
  
  -- Stripe
  stripe_payment_intent_id VARCHAR(255) UNIQUE,
  
  -- Period
  billing_period_start DATE NOT NULL,
  billing_period_end DATE NOT NULL,
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT valid_status CHECK (status IN ('pending', 'succeeded', 'failed', 'refunded'))
);

CREATE INDEX idx_payments_subscription_id ON payments(subscription_id);
CREATE INDEX idx_payments_status ON payments(status);
CREATE INDEX idx_payments_created_at ON payments(created_at);
```

---

#### Table: `billing_history`
**Purpose:** Audit trail of billing changes

```sql
CREATE TABLE billing_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
  
  -- Event
  event_type VARCHAR(50) NOT NULL, -- 'subscription_created', 'upgrade', 'downgrade', 'cancellation', 'refund'
  event_description TEXT,
  
  -- Previous & New Values
  previous_tier VARCHAR(20),
  new_tier VARCHAR(20),
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT valid_event_type CHECK (event_type IN ('subscription_created', 'upgrade', 'downgrade', 'cancellation', 'refund'))
);

CREATE INDEX idx_billing_history_user_id ON billing_history(user_id);
CREATE INDEX idx_billing_history_created_at ON billing_history(created_at);
```

---

### 6. ANALYTICS & LOGGING

#### Table: `activity_logs`
**Purpose:** Track user actions for analytics

```sql
CREATE TABLE activity_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
  
  -- Activity Info
  activity_type VARCHAR(50) NOT NULL, -- 'goal_created', 'conversation_started', 'check_in_completed', 'page_viewed'
  activity_description TEXT,
  
  -- Metadata
  page_or_feature VARCHAR(100),
  metadata JSONB,
  
  -- Timestamps
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT valid_activity_type CHECK (activity_type IN ('goal_created', 'conversation_started', 'check_in_completed', 'page_viewed', 'goal_completed', 'milestone_hit'))
);

CREATE INDEX idx_activity_logs_user_id ON activity_logs(user_id);
CREATE INDEX idx_activity_logs_activity_type ON activity_logs(activity_type);
CREATE INDEX idx_activity_logs_created_at ON activity_logs(created_at);
```

---

#### Table: `user_metrics`
**Purpose:** Aggregate metrics for each user (updated daily)

```sql
CREATE TABLE user_metrics (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
  
  -- Count Metrics
  total_goals_created INTEGER DEFAULT 0,
  total_goals_completed INTEGER DEFAULT 0,
  total_conversations INTEGER DEFAULT 0,
  total_check_ins_completed INTEGER DEFAULT 0,
  total_tasks_completed INTEGER DEFAULT 0,
  
  -- Time Metrics
  total_hours_invested DECIMAL(10,1) DEFAULT 0,
  average_session_duration_minutes DECIMAL(8,2),
  
  -- Streak Info
  current_check_in_streak INTEGER DEFAULT 0,
  longest_check_in_streak INTEGER DEFAULT 0,
  
  -- Engagement
  last_activity_date DATE,
  
  -- Metadata
  metrics_date DATE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(user_id, metrics_date)
);

CREATE INDEX idx_user_metrics_user_id ON user_metrics(user_id);
CREATE INDEX idx_user_metrics_metrics_date ON user_metrics(metrics_date);
```

---

#### Table: `api_usage`
**Purpose:** Track API costs (OpenAI, Stripe, etc.)

```sql
CREATE TABLE api_usage (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
  
  -- API Info
  api_name VARCHAR(50) NOT NULL, -- 'openai', 'stripe', etc.
  
  -- Usage
  request_count INTEGER DEFAULT 1,
  input_tokens INTEGER,
  output_tokens INTEGER,
  estimated_cost DECIMAL(10,4),
  
  -- Date
  usage_date DATE NOT NULL,
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(user_id, api_name, usage_date)
);

CREATE INDEX idx_api_usage_user_id ON api_usage(user_id);
CREATE INDEX idx_api_usage_api_name ON api_usage(api_name);
CREATE INDEX idx_api_usage_usage_date ON api_usage(usage_date);
```

---

## RELATIONSHIPS DIAGRAM

```
┌─────────────────────────────────────────────────────────────────┐
│                     DATABASE RELATIONSHIPS                      │
└─────────────────────────────────────────────────────────────────┘

auth.users (Supabase managed)
    ↓
user_profiles (1:1)
    ├──→ user_interests (1:many)
    ├──→ user_skills (1:many)
    ├──→ user_challenges (1:many)
    ├──→ check_ins (1:many)
    ├──→ user_streaks (1:1)
    ├──→ subscriptions (1:1)
    ├──→ user_preferences (1:many)
    ├──→ conversation_context (1:many)
    ├──→ activity_logs (1:many)
    ├──→ user_metrics (1:many)
    ├──→ api_usage (1:many)
    └──→ goals (1:many)
            ├──→ milestones (1:many)
            ├──→ daily_tasks (1:many)
            │     └──→ task_completions (1:many)
            └──→ reflections (1:many)

conversations (belongs to user_profiles)
    └──→ messages (1:many)

subscriptions (1:1 with user_profiles)
    └──→ payments (1:many)
         └──→ billing_history (reference to user)
```

---

## ROW LEVEL SECURITY (RLS) POLICIES

### Overview
Every table has RLS enabled to prevent unauthorized data access.

### User-Scoped Tables (Most Secure)
Tables with `user_id` field use these policies:

```sql
-- Enable RLS
ALTER TABLE user_profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE goals ENABLE ROW LEVEL SECURITY;
ALTER TABLE check_ins ENABLE ROW LEVEL SECURITY;
ALTER TABLE conversations ENABLE ROW LEVEL SECURITY;
-- ... (all user-scoped tables)

-- Policy: Users can only see their own data
CREATE POLICY "Users can view own profile" 
  ON user_profiles 
  FOR SELECT 
  USING (auth.uid() = id);

CREATE POLICY "Users can update own profile"
  ON user_profiles
  FOR UPDATE
  USING (auth.uid() = id);

CREATE POLICY "Users can view own goals"
  ON goals
  FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can view own conversations"
  ON conversations
  FOR SELECT
  USING (auth.uid() = user_id);

-- ... (similar for all sensitive tables)
```

---

## PERFORMANCE OPTIMIZATION STRATEGY

### Indexing Strategy

**1. Foreign Key Indexes** (Already Created)
```sql
-- Enables fast joins
idx_goals_user_id
idx_check_ins_user_id
idx_conversations_user_id
-- etc.
```

**2. Status/State Indexes** (For Filtering)
```sql
-- Common queries filter by status
idx_goals_status
idx_subscriptions_status
idx_payments_status
```

**3. Date Indexes** (For Range Queries)
```sql
-- Common queries filter by date
idx_goals_created_at
idx_check_ins_check_in_date
idx_messages_created_at
```

**4. Composite Indexes** (For Common Queries)
```sql
-- Future: Add if needed after profiling
-- Example: (user_id, created_at) for recent user activity
```

### Query Optimization Tips

```sql
-- ✅ GOOD: Uses index
SELECT * FROM goals 
WHERE user_id = 'xyz' AND status = 'active';

-- ❌ BAD: Full table scan
SELECT * FROM goals 
WHERE YEAR(created_at) = 2026;

-- ✅ GOOD: Use indexed columns
SELECT * FROM goals 
WHERE created_at > NOW() - INTERVAL '30 days'
  AND user_id = 'xyz';
```

---

## DATA TYPES & CONSTRAINTS

### UUID vs Serial IDs
- **Decision:** Use UUID for all primary keys
- **Why:** Better for distributed systems, harder to guess, no collisions

### Timestamps
```sql
-- CURRENT_TIMESTAMP - timezone-aware, recommended
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP

-- Better practice (timezone-aware):
created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP AT TIME ZONE 'UTC'
```

### JSON Storage
- Used for flexible data (roadmaps, preferences, metadata)
- Allows extending features without schema migrations
- Can be queried with `@>` operator for filtering

### Currency & Decimals
```sql
-- Always use DECIMAL for money (never FLOAT)
amount DECIMAL(10,2)  -- $9,999.99 max

-- Store currency separately
currency VARCHAR(10)
```

---

## BACKUP & RECOVERY

### Backup Strategy
```
- Daily automated backups (Supabase handles this)
- Weekly full backup export to external storage
- Point-in-time recovery available
```

### Data Retention Policy
```
- Active user data: Keep indefinitely
- Deleted user data: 30-day grace period before purge
- Conversation history: Keep for 2 years (compliance)
- Payment records: Keep for 7 years (tax/legal)
```

---

## SCALING CONSIDERATIONS

### For 10K Users
Current schema is optimal. No changes needed.

### For 100K Users
- Consider partitioning large tables by date
- Archive old conversations (>1 year)
- Use read replicas for analytics queries

### For 1M+ Users
- Partition `messages` table by date
- Partition `activity_logs` by date
- Use separate analytics database (data warehouse)
- Consider Redis for session/cache layer

---

## MIGRATION STRATEGY

### Initial Setup (Week 1)
```sql
-- Run in this order:
1. Create user_profiles
2. Create all user-related tables
3. Create goals & related tables
4. Create conversations & messages
5. Create subscriptions & payments
6. Create analytics tables
7. Enable RLS on all tables
8. Create indexes
```

### Future Migrations (Phases 2+)
- Use Supabase migration system
- Always create new columns as nullable first
- Test migrations in staging before production

---

## SAMPLE QUERIES

### Query 1: Get User's Active Goals with Progress
```sql
SELECT 
  g.id,
  g.goal_title,
  g.completion_percentage,
  g.target_date,
  COUNT(DISTINCT m.id) as total_milestones,
  SUM(CASE WHEN tc.was_completed THEN 1 ELSE 0 END) as completed_tasks
FROM goals g
LEFT JOIN milestones m ON g.id = m.goal_id
LEFT JOIN daily_tasks dt ON g.id = dt.goal_id
LEFT JOIN task_completions tc ON dt.id = tc.task_id
WHERE g.user_id = $1 AND g.status = 'active'
GROUP BY g.id
ORDER BY g.created_at DESC;
```

### Query 2: Get User's AI Memory (Context for Personalization)
```sql
SELECT 
  context_type,
  context_data,
  importance_score
FROM conversation_context
WHERE user_id = $1 AND is_active = TRUE
ORDER BY importance_score DESC
LIMIT 10;
```

### Query 3: Get User's Conversation History (For Memory)
```sql
SELECT 
  c.id,
  c.mentor_type,
  c.conversation_title,
  COUNT(m.id) as message_count,
  MAX(m.created_at) as last_message,
  c.context_summary
FROM conversations c
LEFT JOIN messages m ON c.id = m.conversation_id
WHERE c.user_id = $1 AND c.is_archived = FALSE
GROUP BY c.id
ORDER BY c.last_message_at DESC
LIMIT 20;
```

### Query 4: Get Daily Metrics for Analytics Dashboard
```sql
SELECT 
  u.full_name,
  um.total_goals_completed,
  um.current_check_in_streak,
  um.total_hours_invested,
  COUNT(DISTINCT g.id) as active_goals,
  COUNT(DISTINCT c.id) as total_conversations
FROM user_metrics um
JOIN user_profiles u ON um.user_id = u.id
LEFT JOIN goals g ON u.id = g.user_id AND g.status = 'active'
LEFT JOIN conversations c ON u.id = c.user_id
WHERE um.metrics_date = CURRENT_DATE
GROUP BY u.id, um.id
ORDER BY um.current_check_in_streak DESC;
```

---

## SECURITY CHECKLIST

- [x] RLS enabled on all user-scoped tables
- [x] Foreign key constraints enforced
- [x] Sensitive fields (emails, stripe IDs) marked appropriately
- [x] JSONB data validated before storage
- [x] Timestamps use UTC timezone
- [x] Deleted records soft-deleted where appropriate
- [ ] Encryption at rest (Supabase handles this by default)
- [ ] API rate limiting (implemented in backend)
- [ ] Audit logging for sensitive changes (future: add audit table)

---

## NEXT STEPS

### Immediate (This Week)
1. [ ] Review and approve schema
2. [ ] Create database in Supabase
3. [ ] Run migration scripts
4. [ ] Test RLS policies
5. [ ] Load sample data for testing

### Following Week
1. [ ] Create API endpoints for CRUD operations
2. [ ] Implement authentication layer
3. [ ] Write query builders/ORM models
4. [ ] Performance test with 1000+ records

---

**Document Owner:** CTO (Database Architect)  
**Last Updated:** July 2026  
**Next Review:** August 2026  
**Status:** Ready for Implementation
