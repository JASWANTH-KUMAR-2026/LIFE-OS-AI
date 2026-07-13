# LifeOS AI - Database Schema

## Schema Design Philosophy

- **Normalized structure** for consistency and performance
- **Audit trails** for user actions
- **Soft deletes** for data recovery
- **Timestamps** for all entities
- **Indexing** on foreign keys and frequently queried fields

## Models

### User
```prisma
model User {
  id                String    @id @default(cuid())
  email             String    @unique
  password          String    // Bcrypt hashed
  name              String
  
  // Subscription
  plan              Plan      @default(FREE)
  subscriptionId    String?
  subscriptionEnds  DateTime?
  
  // Auth
  emailVerified     DateTime?
  lastLogin         DateTime?
  
  // Relations
  profile           Profile?
  goals             Goal[]
  habits            Habit[]
  conversations     Conversation[]
  memories          Memory[]
  progressReports   ProgressReport[]
  
  // Timestamps
  createdAt         DateTime  @default(now())
  updatedAt         DateTime  @updatedAt
  deletedAt         DateTime?
  
  @@index([email])
  @@index([plan])
}

enum Plan {
  FREE
  PRO
  ENTERPRISE
}
```

### Profile
```prisma
model Profile {
  id                String    @id @default(cuid())
  userId            String    @unique
  user              User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  // Personal Info
  age               Int?
  location          String?
  bio               String?
  avatar            String?
  
  // Life Context
  currentSituation  String?   // Markdown
  interests         String[]  // Array of interests
  skills            String[]  // Array of skills
  strengths         String[]  // Array of strengths
  weaknesses        String[]  // Array of weaknesses
  
  // Preferences
  timezone          String    @default("UTC")
  availableHours    Int       @default(8)  // Daily hours available
  learningStyle     String    @default("mixed") // visual, auditory, kinesthetic, mixed
  preferredLanguage String    @default("en")
  
  // Timestamps
  createdAt         DateTime  @default(now())
  updatedAt         DateTime  @updatedAt
  
  @@index([userId])
}
```

### Goal
```prisma
model Goal {
  id                String       @id @default(cuid())
  userId            String
  user              User         @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  // Goal Info
  title             String
  description       String?      // Markdown
  category          String       // career, business, fitness, learning, finance, personal
  
  // Timeline
  deadline          DateTime?
  startDate         DateTime     @default(now())
  targetDate        DateTime?
  
  // Progress
  status            GoalStatus   @default(ACTIVE)  // ACTIVE, PAUSED, COMPLETED, ABANDONED
  priority          Int          @default(3)  // 1-5, 1 is highest
  progress          Int          @default(0)  // 0-100%
  
  // Relations
  milestones        Milestone[]
  tasks             Task[]
  roadmap           Roadmap?
  progressReports   ProgressReport[]
  memories          Memory[]     // Related insights
  
  // Timestamps
  createdAt         DateTime     @default(now())
  updatedAt         DateTime     @updatedAt
  completedAt       DateTime?
  deletedAt         DateTime?
  
  @@index([userId])
  @@index([status])
  @@index([deadline])
}

enum GoalStatus {
  ACTIVE
  PAUSED
  COMPLETED
  ABANDONED
}
```

### Milestone
```prisma
model Milestone {
  id                String    @id @default(cuid())
  goalId            String
  goal              Goal      @relation(fields: [goalId], references: [id], onDelete: Cascade)
  
  // Info
  title             String
  description       String?
  
  // Timeline
  targetDate        DateTime?
  
  // Progress
  status            GoalStatus @default(ACTIVE)
  progress          Int        @default(0)
  
  // Relations
  tasks             Task[]
  
  // Timestamps
  createdAt         DateTime   @default(now())
  updatedAt         DateTime   @updatedAt
  completedAt       DateTime?
  
  @@index([goalId])
  @@index([status])
}
```

### Task
```prisma
model Task {
  id                String     @id @default(cuid())
  milestoneId       String?
  milestone         Milestone? @relation(fields: [milestoneId], references: [id], onDelete: SetNull)
  goalId            String
  goal              Goal       @relation(fields: [goalId], references: [id], onDelete: Cascade)
  
  // Info
  title             String
  description       String?
  
  // Timeline
  dueDate           DateTime?
  scheduledDate     DateTime?
  
  // Progress
  status            TaskStatus @default(TODO)  // TODO, IN_PROGRESS, COMPLETED, BLOCKED
  priority          Int        @default(3)
  estimatedHours    Float?
  actualHours       Float?
  
  // Relations
  subtasks          Task[]     @relation("subtasks")
  parentTask        Task?      @relation("subtasks", fields: [parentTaskId], references: [id])
  parentTaskId      String?
  
  // Timestamps
  createdAt         DateTime   @default(now())
  updatedAt         DateTime   @updatedAt
  completedAt       DateTime?
  
  @@index([goalId])
  @@index([milestoneId])
  @@index([status])
  @@index([dueDate])
}

enum TaskStatus {
  TODO
  IN_PROGRESS
  COMPLETED
  BLOCKED
}
```

### Habit
```prisma
model Habit {
  id                String       @id @default(cuid())
  userId            String
  user              User         @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  // Info
  name              String
  description       String?
  category          String       // fitness, learning, health, productivity, personal
  
  // Tracking
  frequency         String       // daily, weekly, custom
  targetDays        String[]     // ["Mon", "Tue", ...] for weekly habits
  status            HabitStatus  @default(ACTIVE)
  
  // Streak
  currentStreak     Int          @default(0)
  longestStreak     Int          @default(0)
  lastCompletedAt   DateTime?
  
  // Relations
  logs              HabitLog[]
  
  // Timestamps
  createdAt         DateTime     @default(now())
  updatedAt         DateTime     @updatedAt
  archivedAt        DateTime?
  
  @@index([userId])
  @@index([status])
}

enum HabitStatus {
  ACTIVE
  PAUSED
  ARCHIVED
}
```

### HabitLog
```prisma
model HabitLog {
  id                String    @id @default(cuid())
  habitId           String
  habit             Habit     @relation(fields: [habitId], references: [id], onDelete: Cascade)
  
  // Tracking
  date              DateTime
  completed         Boolean   @default(false)
  notes             String?
  
  // Timestamps
  createdAt         DateTime  @default(now())
  updatedAt         DateTime  @updatedAt
  
  @@unique([habitId, date])
  @@index([habitId])
  @@index([date])
}
```

### Conversation
```prisma
model Conversation {
  id                String     @id @default(cuid())
  userId            String
  user              User       @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  // Context
  title             String?
  context           String?    // JSON context about goals, progress, etc.
  goalId            String?    // Related goal if any
  
  // Messages
  messages          Message[]
  
  // Timestamps
  createdAt         DateTime   @default(now())
  updatedAt         DateTime   @updatedAt
  
  @@index([userId])
  @@index([createdAt])
}
```

### Message
```prisma
model Message {
  id                String        @id @default(cuid())
  conversationId    String
  conversation      Conversation  @relation(fields: [conversationId], references: [id], onDelete: Cascade)
  
  // Content
  role              String        // "user" or "assistant"
  content           String        // Message text
  
  // Metadata
  tokens            Int?          // Token count for cost tracking
  embedding         Float[]?      // Vector embedding for semantic search
  
  // Timestamps
  createdAt         DateTime      @default(now())
  
  @@index([conversationId])
  @@index([role])
}
```

### Memory
```prisma
model Memory {
  id                String    @id @default(cuid())
  userId            String
  user              User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  // Content
  type              String    // preference, insight, lesson, achievement
  title             String
  content           String    // Markdown
  importance        Int       @default(1) // 1-5
  
  // Relations
  relatedGoals      Goal[]    // Many-to-many
  relatedHabits     String[]  // Habit IDs
  
  // Timestamps
  createdAt         DateTime  @default(now())
  updatedAt         DateTime  @updatedAt
  
  @@index([userId])
  @@index([type])
}
```

### Roadmap
```prisma
model Roadmap {
  id                String    @id @default(cuid())
  goalId            String    @unique
  goal              Goal      @relation(fields: [goalId], references: [id], onDelete: Cascade)
  
  // Content
  title             String
  description       String?
  duration          Int       // in months
  
  // Phases (JSON structure)
  phases            String    // JSON array of phases
  // Each phase: { month: 1, title: "", description: "", resources: [], tasks: [] }
  
  // Timestamps
  createdAt         DateTime  @default(now())
  updatedAt         DateTime  @updatedAt
  
  @@index([goalId])
}
```

### ProgressReport
```prisma
model ProgressReport {
  id                String    @id @default(cuid())
  userId            String
  user              User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  goalId            String?
  goal              Goal?     @relation(fields: [goalId], references: [id], onDelete: SetNull)
  
  // Report Info
  period            String    // "weekly", "monthly"
  startDate         DateTime
  endDate           DateTime
  
  // Data (JSON)
  summary           String    // Markdown summary
  metrics           String    // JSON of metrics
  habits            String    // JSON of habit data
  achievements      String[]  // Array of achievements
  challenges        String[]  // Array of challenges faced
  nextActions       String[]  // Array of next actions
  
  // Timestamps
  createdAt         DateTime  @default(now())
  
  @@index([userId])
  @@index([period])
  @@index([startDate])
}
```

## Indexes

```sql
-- Performance-critical indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_goals_user_status ON goals(user_id, status);
CREATE INDEX idx_goals_deadline ON goals(deadline);
CREATE INDEX idx_tasks_goal_status ON tasks(goal_id, status);
CREATE INDEX idx_habits_user_status ON habits(user_id, status);
CREATE INDEX idx_habit_logs_habit_date ON habit_logs(habit_id, date);
CREATE INDEX idx_messages_conversation ON messages(conversation_id);
CREATE INDEX idx_memories_user_type ON memories(user_id, type);
```
