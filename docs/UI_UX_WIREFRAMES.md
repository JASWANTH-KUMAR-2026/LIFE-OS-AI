# LifeOS AI — UI/UX WIREFRAMES & DESIGN SYSTEM (Phase 3)

**Version:** 1.0  
**Date:** July 2026  
**Status:** MVP Design Specification

---

## EXECUTIVE SUMMARY

This document defines the complete UI/UX design for LifeOS AI MVP. The design philosophy:
- **Apple Simplicity** – Clean, minimal, intuitive
- **Notion Organization** – Structured but flexible
- **ChatGPT Intelligence** – Conversational AI interactions
- **Accessibility First** – WCAG 2.1 AA compliant

---

## DESIGN SYSTEM

### Color Palette

```
PRIMARY COLORS:
- Primary Blue: #2563EB (Main actions, links, highlights)
- Primary Purple: #7C3AED (AI personality, special highlights)
- Accent Green: #10B981 (Success, completion, achievements)
- Accent Orange: #F59E0B (Warnings, streaks, energy)

NEUTRAL COLORS:
- Dark Gray: #1F2937 (Text, headings)
- Light Gray: #F3F4F6 (Backgrounds)
- Border Gray: #E5E7EB (Dividers, borders)
- White: #FFFFFF (Cards, surfaces)

SEMANTIC COLORS:
- Success: #10B981 (Green)
- Warning: #F59E0B (Orange)
- Error: #EF4444 (Red)
- Info: #3B82F6 (Blue)
```

### Typography

```
FONT FAMILY: Inter (open source, highly legible)

HEADING HIERARCHY:
- H1 (32px, 700 weight): Page titles
- H2 (24px, 700 weight): Section titles
- H3 (20px, 600 weight): Subsection titles
- H4 (16px, 600 weight): Card titles

BODY TEXT:
- Large: 16px, 400 weight (main body)
- Regular: 14px, 400 weight (secondary text)
- Small: 12px, 400 weight (captions, helper text)

LINE HEIGHT: 1.5 for body, 1.2 for headings
```

### Spacing System

```
Base Unit: 4px

Spacing Scale:
- XS: 4px
- SM: 8px
- MD: 12px
- LG: 16px
- XL: 24px
- XXL: 32px
- XXXL: 48px

Applied to: Padding, margins, gaps
```

### Component Library

```
BUTTONS:
- Primary (Blue): Main CTAs
- Secondary (Gray): Alternative actions
- Tertiary (Link): Minimal actions
- Sizes: SM (32px), MD (40px), LG (48px)

INPUT FIELDS:
- Text input with label
- Textarea for multi-line
- Dropdown select
- Radio buttons & checkboxes
- Date picker
- Slider

CARDS:
- Default card (white bg, border)
- Elevated card (shadow)
- Hover states: Lift up 2px, shadow increases

BADGES:
- Status badges (Active, Paused, Completed)
- Tag badges (Skills, Interests)
- Streak badges (15-day streak 🔥)

MODALS:
- Overlay with centered card
- 24px padding inside
- Close button top-right
```

### Icons

```
Icon Set: Feather Icons (24x24px)
- Consistent, minimal style
- Used for: Navigation, actions, status
- Always accompanied by text (for accessibility)
```

---

## PAGE WIREFRAMES

### 1. LANDING PAGE

**Purpose:** Convince visitors to sign up, explain value proposition

**Layout:**

```
┌─────────────────────────────────────────────────────┐
│                    NAVBAR                           │
│  LifeOS AI    [Pricing] [About] [Login] [Sign Up]   │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│                   HERO SECTION                      │
│                                                     │
│  "Your Personal AI Mentor"                          │
│  "Set goals. Build habits. Transform your life."    │
│                                                     │
│  [Get Started Free]      [Watch Demo]               │
│                                                     │
│  [Hero Image/Animation showing AI conversation]    │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│              FEATURES SECTION (3 COLUMNS)           │
│                                                     │
│  🎯 Goal Planning        💡 AI Mentor        📊 Progress
│  Create actionable      Get expert guidance    Track your
│  90-day plans          in career, business,    journey with
│                        learning, productivity  detailed metrics
│
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│            HOW IT WORKS SECTION                     │
│                                                     │
│  1️⃣ Create Profile → 2️⃣ Set Goals → 3️⃣ Chat AI → 4️⃣ Track Progress
│                                                     │
│  [Visual timeline with icons]                      │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│         PRICING SECTION (TOGGLE: MONTHLY/YEARLY)   │
│                                                     │
│  FREE          PRO                                 │
│  $0/month      $14.99/month  (or $149.99/year)    │
│  ✓ Feature 1   ✓ Feature 1                        │
│  ✓ Feature 2   ✓ Feature 2 (UNLIMITED)            │
│  ✗ Feature 3   ✓ Feature 3                        │
│                ✓ Feature 4                        │
│  [Sign Up]     [Start 7-Day Trial]                │
│                                                     │
└────────────────────────────────────────────��────────┘

┌─────────────────────────────────────────────────────┐
│            TESTIMONIALS SECTION                     │
│                                                     │
│  "LifeOS AI helped me..." ⭐⭐⭐⭐⭐               │
│  - Aria, Software Engineer                        │
│                                                     │
│  "I went from idea to..." ⭐⭐⭐⭐⭐               │
│  - Sameer, Entrepreneur                           │
│                                                     │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│              CTA SECTION                           │
│  "Ready to transform your life?"                   │
│  [Get Started Free]                                │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│              FOOTER                                │
│  Company | Privacy | Terms | Contact | Social      │
└─────────────────────────────────────────────────────┘
```

**Key Elements:**
- Clean hero with value prop
- Social proof (testimonials)
- Clear pricing comparison
- Strong CTA throughout

---

### 2. SIGN UP & LOGIN

#### Sign Up Page

**Layout:**

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│              LifeOS AI                              │
│                                                     │
│              CREATE ACCOUNT                         │
│                                                     │
│  [Full Name input field]                           │
│  [Email input field]                               │
│  [Password input field]                            │
│  [Confirm Password input field]                    │
│                                                     │
│  ☑ I agree to Terms & Privacy                     │
│                                                     │
│  [Sign Up with Email]                              │
│                                                     │
│              ─── OR ───                            │
│                                                     │
│  [Sign Up with Google]                             │
│  [Sign Up with GitHub]                             │
│                                                     │
│  Already have an account? [Login]                  │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Features:**
- Simple, minimal form
- Social login options
- Password validation
- Error states clearly visible

#### Login Page

**Layout:**

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│              LifeOS AI                              │
│                                                     │
│              WELCOME BACK                          │
│                                                     │
│  [Email input field]                               │
│  [Password input field]                            │
│                                                     │
│  [Forgot Password?]                                │
│                                                     │
│  [Login with Email]                                │
│                                                     │
│              ─── OR ───                            │
│                                                     │
│  [Login with Google]                               │
│  [Login with GitHub]                               │
│                                                     │
│  New here? [Create Account]                        │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

### 3. ONBOARDING WIZARD (5 Steps)

**Purpose:** Collect user info to personalize AI responses

#### Step 1/5: Basic Info

```
┌─────────────────────────────────────────────────────┐
│  LifeOS AI        [1️⃣ Profile] [2️⃣ Situation]...    │
├─────────────────────────────────────────────────────┤
│                                                     │
│              LET'S GET TO KNOW YOU                 │
│                                                     │
│  Full Name *                                        │
│  [____________]                                    │
│                                                     │
│  Age *                                              │
│  [________]                                        │
│                                                     │
│  Your Timezone *                                    │
│  [Dropdown: UTC, EST, IST, PST, etc.]             │
│                                                     │
│                                  [Skip] [Next ▶]   │
│                                                     │
└─────────────────────────────────────────────────────┘

Progress: ████░░░░░░ 20%
```

#### Step 2/5: Current Situation

```
┌─────────────────────────────────────────────────────┐
│  LifeOS AI        [1️⃣] [2️⃣ Situation] [3️⃣]...      │
├─────────────────────────────────────────────────────┤
│                                                     │
│              WHAT'S YOUR CURRENT SITUATION?        │
│                                                     │
│  Select one: *                                      │
│                                                     │
│  ⭕ Student - Learning & career building           │
│  ⭕ Employee - Growing in my current role          │
│  ⭕ Entrepreneur - Building my own business        │
│  ⭕ Freelancer - Building my independent career   │
│  ⭕ Other - Something else                         │
│                                                     │
│                                  [◀ Back] [Next ▶]  │
│                                                     │
└─────────────────────────────────────────────────────┘

Progress: ████████░░ 40%
```

#### Step 3/5: Goals & Challenges

```
┌─────────────────────────────────────────────────────┐
│  LifeOS AI        [...] [3️⃣ Goals] [4️⃣]...         │
├─────────────────────────────────────────────────────┤
│                                                     │
│              WHAT ARE YOUR GOALS?                  │
│                                                     │
│  Main Goal (What do you want to achieve?) *       │
│  [________________________________]                │
│                                                     │
│  Why is this important to you? *                   │
│  [________________________________]                │
│                                                     │
│  What's your biggest challenge right now? *       │
│  [________________________________]                │
│                                                     │
│  What have you already tried? *                    │
│  [________________________________]                │
│                                                     │
│                                  [◀ Back] [Next ▶]  │
│                                                     │
└─────────────────────────────────────────────────────┘

Progress: ██████████░░ 60%
```

#### Step 4/5: Skills & Interests

```
┌─────────────────────────────────────────────────────┐
│  LifeOS AI        [...] [4️⃣ Skills] [5️⃣]          │
├─────────────────────────────────────────────────────┤
│                                                     │
│              YOUR CURRENT SKILLS                   │
│                                                     │
│  Select your skills (choose at least 3): *        │
│                                                     │
│  [Programming]  [Design]  [Writing]  [Marketing]  │
│  [Sales]  [Leadership]  [Analytics]  [Project Mgmt]
│  [Communication]  [Problem Solving]  [Creativity] │
│                                                     │
│              YOUR INTERESTS                        │
│                                                     │
│  Select your interests: *                          │
│                                                     │
│  [Technology]  [Business]  [Health]  [Education]  │
│  [Travel]  [Art]  [Science]  [Entertainment]      │
│                                                     │
│                                  [◀ Back] [Next ▶]  │
│                                                     │
└─────────────────────────────────────────────────────┘

Progress: ████████████░░ 80%
```

#### Step 5/5: Learning Preferences

```
┌─────────────────────────────────────────────────────┐
│  LifeOS AI        [...] [5️⃣ Preferences]           │
├─────────────────────────────────────────────────────┤
│                                                     │
│              YOUR LEARNING STYLE                   │
│                                                     │
│  How do you learn best? *                          │
│  ⭕ Visual (Videos, diagrams, images)            │
│  ⭕ Hands-on (Practice, projects, building)      │
│  ⭕ Reading (Books, articles, documentation)     │
│  ⭕ Mixed (Combination of all)                   │
│                                                     │
│              HOW SHOULD I COMMUNICATE?            │
│                                                     │
│  ⭕ Direct & brief (Get to the point)            │
│  ⭕ Detailed & thorough (Explain everything)     │
│  ⭕ Conversational (Chat like friends)           │
│                                                     │
│  How many hours per day can you invest? *         │
│  [Slider: 0─────5────10────15────20────24] hours │
│                                                     │
│                            [◀ Back] [Get Started!] │
│                                                     │
└─────────────────────────────────────────────────────┘

Progress: ████████████████ 100%
```

**Design Notes:**
- Progress bar shows completion
- Step indicator at top
- Friendly, conversational language
- Skip option for optional fields
- Validation on submit

---

### 4. DASHBOARD (Main Hub)

**Purpose:** Central view of all progress and quick actions

**Layout:**

```
┌─────────────────────────────────────────────────────┐
│                   NAVBAR                           │
│  🔷 LifeOS AI  [Search] [Notifications] [Profile▼] │
└─────────────────────────────────────────────────────┘

┌──────────────────────────┬──────────────────────────┐
│    LEFT SIDEBAR          │   MAIN CONTENT           │
│                          │                          │
│ 🏠 Dashboard            │ ┌────────────────────────┐│
│ 💬 AI Mentors           │ │   WELCOME BACK, ARIA!  ││
│ 🎯 My Goals             │ │                        ││
│ 📊 Progress             │ │ 15-day streak 🔥       ││
│ ⚙️ Settings             │ │ You're crushing it!    ││
│ 📚 Resources            │ └────────────────────────┘│
│ ? Help & Feedback       │                          │
│                          │ ┌────────────────────────┐│
│                          │ │  TODAY'S FOCUS         ││
│                          │ │                        ││
│                          │ │ Complete 2hr Python    ││
│                          │ │ session (in progress) ││
│                          │ │ □ Learn Python         ││
│                          │ │   Milestone (3/10)     ││
│                          │ │ □ Build Weather App    ││
│                          │ │   Milestone (2/10)     ││
│                          │ │                        ││
│                          │ │ [Do Daily Check-in]    ││
│                          │ └────────────────────��───┘│
│                          │                          │
│                          │ ┌─────────┬─────────────┐│
│                          │ │ ACTIVE GOALS (3)       ││
│                          │ ├─────────┴─────────────┤│
│                          │ │ 🎯 Learn Python       ││
│                          │ │ 45% ███████░░░░░░     ││
│                          │ │ 45 days left          ││
│                          │ │                        ││
│                          │ │ 🎯 Build Portfolio    ││
│                          │ │ 20% ████░░░░░░░░░░   ││
│                          │ │ 70 days left          ││
│                          │ │                        ││
│                          │ │ 🎯 Networking Event   ││
│                          │ │ 5% █░░░░░░░░░░░░░    ││
│                          │ │ 8 days left           ││
│                          │ └────────────────────────┘│
│                          │                          │
│                          │ ┌────────────────────────┐│
│                          │ │  QUICK AI MENTORS      ││
│                          │ │                        ││
│                          │ │ [💼 Career] [👔Biz]    ││
│                          │ │ [⏱️ Productivity]      ││
│                          │ │ [📚 Learning Coach]    ││
│                          │ └────────────────────────┘│
│                          │                          │
│                          │ ┌────────────────────────┐│
│                          │ │  YOUR STATS            ││
│                          │ │                        ││
│                          │ │ Goals Started: 5       ││
│                          │ │ Goals Completed: 1 ✅  ││
│                          │ │ Total Hours: 45h       ││
│                          │ │ Skills Gained: 2       ││
│                          │ └────────────────────────┘│
│                          │                          │
└──────────────────────────┴──────────────────────────┘
```

**Key Sections:**
- Welcome message with streak
- Today's focus card
- Active goals list with progress bars
- Quick AI mentor access
- Stats at a glance

---

### 5. AI MENTOR CHAT INTERFACE

**Purpose:** Have intelligent conversations with AI mentors

**Layout:**

```
┌─────────────────────────────────────────────────────┐
│  🔷 LifeOS AI    Chat with Career Mentor    [X]     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Career Mentor | Entrepreneur | Productivity |...  │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  [Message 1 - Assistant - 3:45 PM]                 │
│  ┌─────────────────────────────────────────────┐   │
│  │ Hey Aria! 👋 I'm your Career Mentor. I can │   │
│  │ help you with career decisions, skill       │   │
│  │ development, interviews, and more.          │   │
│  │                                             │   │
│  │ I see you're interested in becoming a       │   │
│  │ Senior Developer. Let's build a roadmap.    │   │
│  │                                             │   │
│  │ First, tell me: What's your current level?  │   │
│  │                                             │   │
│  │ [Generate 90-Day Roadmap] [Save This]       │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  [Message 2 - User - 4:00 PM]                      │
│  ┌─────────────────────────────────────────────┐   │
│  │ I'm a mid-level developer with 3 years of   │   │
│  │ experience. I know React and Node.js but    │   │
│  │ want to expand to DevOps and system design. │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  [Message 3 - Assistant - 4:02 PM - Loading...]    │
│  ┌─────────────────────────────────────────────┐   │
│  │ 💬 Career Mentor is typing...               │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  [Input field: Type your message...]               │
│  [📎 Attach]  [😊 Emoji]     [Send ➤]            │
│                                                     │
│  💡 Tip: Ask about goals, skills, or challenges   │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Key Features:**
- Conversation history visible
- Mentor type selector at top
- AI responses with suggested actions
- Quick action buttons (Generate Roadmap, Save)
- Typing indicator
- Helpful tips

---

### 6. GOALS PAGE

**Purpose:** Manage all goals and view detailed progress

**Layout:**

```
┌─────────────────────────────────────────────────────┐
│  🔷 LifeOS AI    MY GOALS              [+ New Goal] │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Filters: [All] [Active] [Completed] [Paused]     │
│  Sort by: [Newest ▼] [Progress ▼] [Days Left ▼]  │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │ 🎯 LEARN PYTHON & GET FIRST FREELANCE CLIENT│  │
│  │                                              │  │
│  │ Started: 15 days ago  |  45 days left       │  │
│  │ Category: Skills  |  Timeline: 90 days      │  │
│  │                                              │  │
│  │ Progress: 45% ████████░░░░░░░░░░░░░░░░   │  │
│  │                                              │  │
│  │ Milestones: 4/10 completed                 │  │
│  │ Tasks this week: 3/3 ✅                     │  │
│  │ Streak: 7 days 🔥                          │  │
│  │                                              │  │
│  │ [View Details] [Edit] [Pause] [Archive]     │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │ 🎯 BUILD PROFESSIONAL PORTFOLIO              │  │
│  │                                              │  │
│  │ Started: 3 days ago  |  70 days left        │  │
│  │ Category: Career  |  Timeline: 90 days      │  │
│  │                                              │  │
│  │ Progress: 20% ████░░░░░░░░░░░░░░░░░░░░    │  │
│  │                                              │  │
│  │ Milestones: 1/10 completed                 │  │
│  │ Tasks this week: 1/3 ⏳                     │  │
│  │ Streak: 1 day 🔥                           │  │
│  │                                              │  │
│  │ [View Details] [Edit] [Pause] [Archive]     │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │ ✅ COMPLETE ONLINE COURSE (COMPLETED)        │  │
│  │                                              │  │
│  │ Started: 60 days ago  |  Completed 15 days  │  │
│  │ Category: Learning  |  Timeline: 90 days    │  │
│  │                                              │  │
│  │ Progress: 100% ███████████████████████████ │  │
│  │                                              │  │
│  │ Milestones: 9/9 completed ✅                │  │
│  │ Streak: Completed! (30 days maintained)     │  │
│  │                                              │  │
│  │ [View Details] [Archive] [Redo Goal]        │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Key Features:**
- Goal cards with status indicators
- Progress bars with percentage
- Milestones and tasks at a glance
- Streak tracking
- Filter and sort options
- Action buttons per goal

---

### 7. GOAL DETAILS PAGE

**Purpose:** Deep dive into a single goal with full tracking

**Layout:**

```
┌─────────────────────────────────────────────────────┐
│  🔷 LifeOS AI < My Goals    LEARN PYTHON           │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │ LEARN PYTHON & GET FIRST FREELANCE CLIENT   │  │
│  │                                              │  │
│  │ Started 15 days ago                          │  │
│  │ 45 days remaining (90-day goal)              │  │
│  │ Status: 🟢 ACTIVE                            │  │
│  │                                              │  │
│  │ Progress: 45% ████████░░░░░░░░░░░░░░░░   │  │
│  │                                              │  │
│  │ [Edit Goal] [Pause] [Archive] [...more]     │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  ┌─ 📋 ROADMAP ─────────────────────────────────┐  │
│  │                                              │  │
│  │ Phase 1: Python Fundamentals (Days 1-30)   │  │
│  │ ████████░░░░░░░░░░░░░░░░░░░░░░░░░░ 25%    │  │
│  │ ✓ Week 1: Variables, data types             │  │
│  │ ✓ Week 2: Control flow & loops              │  │
│  │ • Week 3: Functions & modules (in progress) │  │
│  │ ○ Week 4: Review & small project            │  │
│  │                                              │  │
│  │ Phase 2: Intermediate Concepts (Days 31-60) │  │
│  │ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 0%   │  │
│  │ ○ Week 5: OOP fundamentals                  │  │
│  │ ○ Week 6: Working with files & APIs         │  │
│  │ ○ Week 7: Database basics                   │  │
│  │ ○ Week 8: Project: Build a mini app         │  │
│  │                                              │  │
│  │ Phase 3: Freelance Ready (Days 61-90)      │  │
│  │ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 0%   │  │
│  │ ○ Week 9: Advanced topics                   │  │
│  │ ○ Week 10: Portfolio project                │  │
│  │ ○ Week 11: Freelance job hunting            │  │
│  │ ○ Week 12: Land first client                │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  ┌─ 📅 THIS WEEK'S TASKS ─────────────────────────┐ │
│  │                                              │  │
│  │ ✅ Mon: 2-hour Python session                │  │
│  │ ✅ Tue: Review functions concepts            │  │
│  │ ⏳ Wed: Start mini project (0/2h done)      │  │
│  │ ○ Thu: Code review session                  │  │
│  │ ○ Fri: Reflection & planning                │  │
│  │                                              │  │
│  │ Weekly Streak: 4 days 🔥                    │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  ┌─ 💭 REFLECTIONS ──────────────────────────────┐  │
│  │                                              │  │
│  │ Week 2 Reflection (June 30)                 │  │
│  │ "Things going great! Functions were tricky  │  │
│  │  at first but now making sense. Spending    │  │
│  │  3 hours/day. Feeling motivated!"           │  │
│  │                                              │  │
│  │ Week 1 Reflection (June 23)                 │  │
│  │ "Started strong. Variables concepts are     │  │
│  │  easy. Ready for loops!"                    │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Key Sections:**
- Goal overview with timeline
- Complete roadmap visualization
- Weekly tasks with completion
- Reflection log
- Edit & management options

---

### 8. PROGRESS & ANALYTICS PAGE

**Purpose:** See patterns and overall life progress

**Layout:**

```
┌─────────────────────────────────────────────────────┐
│  🔷 LifeOS AI    PROGRESS & INSIGHTS                │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Time Period: [This Week ▼] [This Month ▼]        │
│                                                     │
│  ┌────────���─────────────────────────────────────┐  │
│  │         📊 YOUR STATS                        │  │
│  ├──────────────────────────────────────────────┤  │
│  │                                              │  │
│  │  Total Hours Invested: 45h        📈 +5h    │  │
│  │  Goals Completed: 1 / 5           ✅ 20%   │  │
│  │  Check-in Streak: 15 days         🔥 Days  │  │
│  │  Skills Gained: 2                 📚 New    │  │
│  │  Conversations with AI: 42        💬 Total │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │         ⏱️ TIME BREAKDOWN                    │  │
│  ├─────────────────────────────────────��────────┤  │
│  │                                              │  │
│  │ Learning (🟦)    20 hours      44%          │  │
│  │ Career (🟩)       15 hours      33%          │  │
│  │ Productivity (🟪)  10 hours      23%          │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │         📈 GOAL PROGRESS                     │  │
│  ├──────────────────────────────────────────────┤  │
│  │                                              │  │
│  │ Learn Python:        45% ████████░░░░░░    │  │
│  │ Build Portfolio:     20% ████░░░░░░░░░░   │  │
│  │ Network Events:       5% █░░░░░░░░░░░░░   │  │
│  │                                              │  │
│  │ [View All Goals →]                          │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │         📅 DAILY ACTIVITY (Last 7 Days)     │  │
│  ├──────────────────────────────────────────────┤  │
│  │                                              │  │
│  │  Sat    Sun    Mon    Tue    Wed    Thu   Fri │  │
│  │  🟩    🟩    🟩    🟩    🟩    🟩    🟩   │  │
│  │  3h    2h    3h    4h    3h    2h    5h   │  │
│  │                                              │  │
│  │  Your most productive: Friday evenings ✨   │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │         🎯 INSIGHTS & RECOMMENDATIONS       │  │
│  ├──────────────────────────────────────────────┤  │
│  │                                              │  │
│  │ ✨ You've maintained a 15-day streak!      │  │
│  │    Keep it up! 🔥                           │  │
│  │                                              │  │
│  │ 📊 Learning is your strongest category     │  │
│  │    Consider taking on another course?      │  │
│  │                                              │  │
│  │ ⚠️ Portfolio project is falling behind      │  │
│  │    Let's chat with Career Mentor about it  │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
└─────────────────────────────────────────────────────��
```

**Key Elements:**
- High-level stats cards
- Time breakdown pie chart
- Goal progress bars
- Activity heatmap
- AI-generated insights

---

### 9. SUBSCRIPTION & BILLING PAGE

**Purpose:** Manage subscription and billing

**Layout:**

```
┌─────────────────────────────────────────────────────┐
│  🔷 LifeOS AI    SUBSCRIPTION & BILLING             │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │         YOUR CURRENT PLAN                    │  │
│  ├──────────────────────────────────────────────┤  │
│  │                                              │  │
│  │  FREE                                        │  │
│  │  $0/month                                    │  │
│  │                                              │  │
│  │  ✓ Basic AI mentor (1 specialization)       │  │
│  │  ✓ 20 conversations/month                   │  │
│  │  ✓ Up to 3 goals                            │  │
│  │  ✗ AI memory (limited 30 days)              │  │
│  │                                              │  │
│  │  [Upgrade to PRO for More]                  │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │         UPGRADE TO PRO                      │  │
│  ├──────────────────────────────────────────────┤  │
│  │                                              │  │
│  │  PRO                                         │  │
│  │  $14.99/month (or $149.99/year)             │  │
│  │                                              │  │
│  │  ✓ Unlimited AI conversations               │  │
│  │  ✓ All 4 mentors (Career, Biz, etc)        │  │
│  │  ✓ Unlimited goals                          │  │
│  │  ✓ Full AI memory (forever)                 │  │
│  │  ✓ Advanced analytics                       │  │
│  │  ✓ Priority support                         │  │
│  │  ✓ Export features                          │  │
│  │                                              │  │
│  │  You save $29.99/year with annual plan ✨  │  │
│  │                                              │  │
│  │  [Upgrade Now]  [Start 7-Day Free Trial]    │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │         BILLING HISTORY                      │  │
│  ├──────────────────────────────────────────────┤  │
│  │                                              │  │
│  │ No billing history (Free account)            │  │
│  │                                              │  │
│  │ Once you upgrade, all transactions will     │  │
│  │ appear here with receipts.                  │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │         PAYMENT METHOD                       │  │
│  ├──────────────────────────────────────────────┤  │
│  │                                              │  │
│  │ No payment method on file yet.               │  │
│  │ Add one when you upgrade to PRO.             │  │
│  │                                              │  │
│  │ Accepted: Visa, Mastercard, Amex, Google   │  │
│  │ Pay, Apple Pay                              │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

### 10. SETTINGS & PROFILE PAGE

**Layout:**

```
┌─────────────────────────────────────────────────────┐
│  🔷 LifeOS AI    SETTINGS & PROFILE                 │
├─────────────────────────────────────────────────────┤
│                                                     │
│  [Profile] [Preferences] [Notifications] [Security]│
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │         PROFILE SETTINGS                     │  │
│  ├──────────────────────────────────────────────┤  │
│  │                                              │  │
│  │  Full Name                                   │  │
│  │  [Aria Chen]                                 │  │
│  │                                              │  │
│  │  Email                                       │  │
│  │  [aria@example.com]  [Verified ✓]           │  │
│  │                                              │  │
│  │  Timezone                                    │  │
│  │  [America/New_York ▼]                       │  │
│  │                                              │  │
│  │  Avatar                                      │  │
│  │  [👤] [Upload New Photo]                    │  │
│  │                                              │  │
│  │  Situation                                   │  │
│  │  [Employee - Growing in current role]      │  │
│  │                                              │  │
│  │  [Save Changes] [Cancel]                    │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │         AI PREFERENCES                       │  │
│  ├──────────────────────────────────────────────┤  │
│  │                                              │  │
│  │  How should I communicate with you?         │  │
│  │  ⭕ Direct & brief                          │  │
│  │  ⭕ Detailed & thorough                     │  │
│  │  ⭕ Conversational                          │  │
│  │                                              │  │
│  │  Your learning style:                       │  │
│  │  ⭕ Visual                                   │  │
│  │  ⭕ Hands-on                                │  │
│  │  ⭕ Reading                                 │  │
│  │  ⭕ Mixed                                   │  │
│  │                                              │  │
│  │  Daily check-in time:                       │  │
│  │  [08:00 AM ▼]                               │  │
│  │                                              │  ��
│  │  [Save Preferences]                         │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │         DANGER ZONE                          │  │
│  ├──────────────────────────────────────────────┤  │
│  │                                              │  │
│  │  [Delete Account]  (Permanent, no recovery) │  │
│  │  [Logout from All Devices]                  │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## MOBILE RESPONSIVENESS

### Breakpoints

```
Mobile:    < 640px (full-width single column)
Tablet:    640px - 1024px (sidebar collapses, 2-column)
Desktop:   > 1024px (full layout with sidebars)
```

### Key Changes for Mobile

```
MOBILE LAYOUT:
- Hamburger menu instead of sidebar
- Single column layout
- Full-width cards
- Bottom navigation bar (Dashboard, Mentors, Goals, Profile)
- Larger touch targets (48px minimum)

TOUCH INTERACTIONS:
- Swipe to dismiss modals
- Long-press for context menus
- Pull-to-refresh on lists
```

---

## INTERACTION PATTERNS & MICRO-INTERACTIONS

### Buttons

```
States:
- Default: Full color, pointer cursor
- Hover: Slight shadow increase, color darken by 10%
- Active/Pressed: Color darken by 20%
- Disabled: 50% opacity, not-allowed cursor
- Loading: Show spinner inside button

Example:
┌─────────────┐
│ Save Changes│ → Hover → ┌─────────────┐
└─────────────┘           │ Save Changes│ (shadow)
                          └─────────────┘
```

### Form Validation

```
REAL-TIME VALIDATION:

Before interaction:
[Email field]

While typing (invalid):
[Email field] ❌ Invalid email format

While typing (valid):
[Email field] ✅ Email looks good!

On submit (error):
Email
[email@example.com] ❌
Error: This email is already registered
```

### Loading States

```
SKELETON LOADING:
┌─────────────────────────┐
│ ▓▓▓▓▓▓▓▓▓▓ ▓▓▓▓▓▓     │  
│ ▓▓▓▓▓▓▓▓▓▓ ▓▓▓▓        │
│ ▓▓▓▓▓ ▓▓▓▓▓            │
└─────────────────────────┘
(Animates with wave effect)

SPINNER:
    ⟲ Loading...
    (Rotates)

PROGRESS:
████░░░░░░ 40%
(Animates smoothly)
```

### Success Notifications

```
TOAST NOTIFICATION (bottom-right):
┌───────────────────────────────┐
│ ✅ Goal saved successfully!    │  (Auto-closes in 3s)
└───────────────────────────────┘

INLINE SUCCESS:
[Save Changes] → ✅ Saved!  (Green checkmark appears)
```

---

## ACCESSIBILITY FEATURES

### Keyboard Navigation

```
TAB: Move to next interactive element
SHIFT + TAB: Move to previous
ENTER: Activate button
SPACE: Toggle checkbox/radio
ARROW KEYS: Navigate lists/sliders
ESC: Close modals
```

### Screen Reader Support

```
✓ Semantic HTML (<button>, <nav>, <main>, etc.)
✓ ARIA labels for icons: <button aria-label="Close chat">X</button>
✓ Form labels linked to inputs: <label for="email">Email</label>
✓ Skip navigation links: <a href="#main-content">Skip to main</a>
✓ Alt text for images
✓ Focus indicators always visible (4px outline)
```

### Color Contrast

```
WCAG AA (minimum):
- Normal text: 4.5:1 contrast ratio
- Large text: 3:1 ratio

WCAG AAA (enhanced):
- Normal text: 7:1 contrast ratio
- Large text: 4.5:1 ratio

We meet WCAG AA throughout.
```

---

## DESIGN TOKENS (CSS VARIABLES)

```css
/* COLORS */
--color-primary: #2563EB;
--color-primary-light: #DBEAFE;
--color-primary-dark: #1E40AF;

--color-success: #10B981;
--color-warning: #F59E0B;
--color-error: #EF4444;

--color-text-primary: #1F2937;
--color-text-secondary: #6B7280;
--color-bg-light: #F3F4F6;
--color-bg-white: #FFFFFF;

/* SPACING */
--spacing-xs: 4px;
--spacing-sm: 8px;
--spacing-md: 12px;
--spacing-lg: 16px;
--spacing-xl: 24px;

/* TYPOGRAPHY */
--font-family: 'Inter', -apple-system, sans-serif;
--font-size-sm: 12px;
--font-size-base: 14px;
--font-size-lg: 16px;

/* BORDER RADIUS */
--radius-sm: 4px;
--radius-md: 8px;
--radius-lg: 12px;

/* SHADOWS */
--shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
--shadow-md: 0 4px 6px rgba(0,0,0,0.1);
--shadow-lg: 0 10px 15px rgba(0,0,0,0.1);

/* TRANSITIONS */
--transition-fast: 150ms ease-in-out;
--transition-base: 300ms ease-in-out;
```

---

## COPY & TONE OF VOICE

### Brand Voice
- **Friendly** – Like talking to a friend
- **Expert** – But never condescending
- **Motivating** – Celebrate wins
- **Honest** – Acknowledge challenges

### Copy Examples

```
❌ BAD: "System Error 42001"
✅ GOOD: "We couldn't save that. Try again in a moment?"

❌ BAD: "Enter personal information"
✅ GOOD: "Let's get to know you"

❌ BAD: "Subscription upgraded"
✅ GOOD: "Awesome! Welcome to PRO. You now have unlimited mentor access 🎉"

❌ BAD: "Goal creation failed"
✅ GOOD: "That goal isn't quite there yet. Need help refining it?"
```

---

## PROTOTYPING & TESTING TOOLS

**Recommended Tools:**

```
DESIGN:
- Figma (design system, prototyping)
- Adobe XD (prototyping)

TESTING:
- Maze (user testing)
- Useberry (feedback)
- Hotjar (heatmaps, session recordings)

HANDOFF:
- Figma Dev Mode (specs for developers)
- Abstract (design versioning)
```

---

## NEXT STEPS

### Immediate Actions
1. [ ] Create Figma design system
2. [ ] Build component library
3. [ ] Create high-fidelity mockups for each page
4. [ ] User test with 5 beta users
5. [ ] Refine based on feedback

### Component Development Priority

**Phase 1 (MVP):**
1. Button, Input, Textarea
2. Card, Modal
3. Progress bar, Badge
4. Sidebar navigation

**Phase 2:**
5. Datepicker, Dropdown
6. Toast notifications
7. Skeletons
8. Charts/graphs

---

**Design Lead:** Product Designer  
**Last Updated:** July 2026  
**Design File:** Figma (link coming soon)  
**Accessibility Audit:** WCAG 2.1 AA Compliant
