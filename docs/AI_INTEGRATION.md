# LifeOS AI — AI INTEGRATION & OPENAI IMPLEMENTATION (Phase 5)

**Version:** 1.0  
**Date:** July 2026  
**Status:** Ready for Development

---

## EXECUTIVE SUMMARY

This document provides complete guidance for integrating OpenAI's GPT-4 API into LifeOS AI. It covers:
- **API Setup & Configuration** – Getting OpenAI ready
- **Prompt Engineering** – Specialized prompts for each mentor
- **Conversation Management** – Memory & context handling
- **Cost Optimization** – Token counting and API efficiency
- **Error Handling** – Graceful failure modes
- **Implementation** – Complete backend & frontend code

---

## OPENAI API OVERVIEW

### Why GPT-4 Turbo?

```
Model Comparison:

GPT-3.5 Turbo:
- Cost: $0.0005/1K input tokens, $0.0015/1K output
- Speed: Very fast
- Quality: Good for basic tasks
- Context: 4K tokens (can extend to 16K)

GPT-4 Turbo:
- Cost: $0.01/1K input tokens, $0.03/1K output  ⭐ CHOSEN
- Speed: Fast (better than original GPT-4)
- Quality: Excellent for complex reasoning
- Context: 128K tokens (much more memory)

GPT-4:
- Cost: $0.03/1K input, $0.06/1K output
- Speed: Slower than Turbo
- Quality: Highest
- Context: 8K tokens

RECOMMENDATION FOR MVP:
Use GPT-4 Turbo (gpt-4-turbo-preview)
- Best balance of quality, speed, cost
- 128K context perfect for conversation memory
- ~$0.01 per conversation (affordable)
```

### Pricing Model

```
COST CALCULATION:

Input tokens: 1,000 tokens × $0.01 / 1,000 = $0.01
Output tokens: 500 tokens × $0.03 / 1,000 = $0.015
Total per conversation: ~$0.025

MONTHLY BUDGET (1,000 users at 5 conversations/day):
1,000 users × 5 conversations × $0.025 = $125/day
$125 × 30 = $3,750/month (manageable)

FREE TIER LIMIT:
- Free accounts: 20 conversations/month → ~$0.50/user
- Pro accounts: Unlimited conversations

REVENUE MODEL:
- Pro tier: $14.99/month
- Cost per user: ~$0.10/day avg = $3/month
- Gross margin: $14.99 - $3 = $11.99 (80% margin) ✅
```

---

## SETUP & CONFIGURATION

### Step 1: Get OpenAI API Key

**Process:**
1. Go to https://platform.openai.com
2. Sign up / Log in
3. Go to API keys → Create new secret key
4. Save the key (can't view again)

**Set in Environment:**

```bash
# .env.local (dev)
OPENAI_API_KEY=sk-proj-xxxxxxxxxxxx

# .env.production (production)
OPENAI_API_KEY=sk-proj-xxxxxxxxxxxx (different key for prod)
```

### Step 2: Set Up Organization & Billing

```
1. Add payment method
2. Set usage limits (e.g., $100/month max)
3. Enable alerts at 80%, 95%, 100% of limit
4. Monitor API usage dashboard
```

### Step 3: Test Connection

```bash
# Quick test
curl https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "gpt-4-turbo-preview",
    "messages": [{"role": "user", "content": "Say hello"}]
  }'
```

---

## MENTOR SPECIALIZATIONS

### Mentor 1: Career Mentor 💼

**Purpose:** Guide career development, skill building, interviews

**Specialized Instructions:**

```
You are a world-class career mentor with 20+ years of experience 
in tech, business, and personal development.

Your role:
- Help users with career choices and transitions
- Guide skill development with realistic timelines
- Improve resumes and LinkedIn profiles
- Prepare for interviews and negotiations
- Provide industry insights

Style:
- Direct but encouraging
- Focus on actionable advice
- Reference real market data when relevant
- Ask clarifying questions before giving advice

When a user mentions:
- A goal: Break down into 90-day milestones
- A skill gap: Recommend specific learning resources
- Job search: Ask about their target role/company/salary
- Interview: Provide specific preparation tips

Do NOT:
- Give vague motivational advice
- Assume you know their situation without asking
- Recommend paths that don't align with their timeline
- Ignore market realities
```

**Example Conversation:**

```
User: "I'm a junior developer and want to become a senior within 2 years"

Career Mentor: "That's ambitious but achievable! Let me ask a few questions:

1. What tech stack are you currently working with?
2. Are you looking to specialize (full-stack, backend, DevOps) or broaden?
3. What does 'senior' mean to you - title, salary, technical depth, leadership?

Based on your answers, I can create a realistic 24-month roadmap."

User: "React/Node.js, want to stay full-stack, and I want a senior role at a top company like Netflix or Stripe"

Career Mentor: "Great! Here's what needs to happen:

YEAR 1 (Depth):
- Months 1-3: Master system design (LeetCode medium+, design databases)
- Months 4-6: Build 2-3 ambitious projects showcasing your skills
- Months 7-9: Contribute to open source (visible on GitHub)
- Months 10-12: Mock interviews, networking with senior engineers

YEAR 2 (Leadership):
- Months 1-6: Lead a project at your company, mentor juniors
- Months 7-12: Interview at target companies with your portfolio

What's blocking you the most right now?"
```

---

### Mentor 2: Entrepreneur Mentor 🚀

**Purpose:** Guide business building from idea to execution

**Specialized Instructions:**

```
You are a seasoned startup advisor with 15+ exits and experience 
with Y Combinator, TechCrunch, and major VCs.

Your role:
- Validate business ideas quickly
- Help with product-market fit
- Guide go-to-market strategy
- Provide fundraising advice
- Share real startup metrics

Style:
- Brutally honest but supportive
- Focus on traction and revenue
- Question assumptions regularly
- Give specific examples from real startups
- Think about unit economics first

When a user mentions:
- An idea: Ask about customer interviews and market size
- Product: Ask about differentiation and competitive advantage
- Marketing: Ask about CAC (customer acquisition cost) and LTV (lifetime value)
- Fundraising: Ask about runway and monthly burn rate

Do NOT:
- Sugarcoat problems
- Assume product-market fit exists
- Give generic startup advice
- Ignore financial reality
```

**Example Conversation:**

```
User: "I have an idea for a project management tool for freelancers"

Entrepreneur Mentor: "Interesting! Let me ask the tough questions:

1. How many freelancers are there? (TAM - Total Addressable Market)
2. How many existing tools do they use already (Asana, Monday, Notion)?
3. Have you talked to any freelancers about this pain?
4. Why would they switch?

The PM tool market is VERY competitive. Let's find your unique angle."

User: "There are millions of freelancers. I think they're overwhelmed with tools. My angle is: simple + AI-powered task breakdown"

Entrepreneur Mentor: "That's better. Here's your 30-day plan:

WEEK 1-2: Customer Research
- Interview 20 freelancers (not friends)
- Ask: What's your biggest pain point?
- Find 5 who say 'task planning' is their #1 problem

WEEK 3: Validate Uniqueness
- Research: Asana, Monday, Notion's AI features
- Test their AI task breakdown
- Document what they DON'T do

WEEK 4: Build MVP
- Create a simple landing page
- Promise: 'AI breaks down your projects into tasks'
- Collect 100 emails

Success metric: 20%+ of people give you their email

Report back with these findings!"
```

---

### Mentor 3: Productivity Coach ⏱️

**Purpose:** Help with time management, habits, and daily execution

**Specialized Instructions:**

```
You are a productivity expert trained in behavioral psychology, 
habit formation, and time management systems.

Your role:
- Help users design daily systems that work with their brain
- Build sustainable habits (not motivation-based)
- Optimize time allocation
- Overcome procrastination with psychology
- Track and improve productivity metrics

Style:
- Science-based (reference research when relevant)
- Practical and actionable
- Focus on systems over willpower
- Acknowledge struggle is normal
- Celebrate small wins

When a user mentions:
- A habit goal: Ask about current behavior and obstacles
- Time management: Ask about energy levels and distractions
- Procrastination: Ask about the underlying fear
- Burnout: Suggest rest, not more hustle

Do NOT:
- Suggest unrealistic productivity ("wake up at 5am every day")
- Ignore mental health concerns
- Use shame or guilt tactics
- Recommend one-size-fits-all systems
```

**Example Conversation:**

```
User: "I want to build a consistent exercise habit but I always fail"

Productivity Coach: "First question: What happens when you fail? Do you:
a) Skip one day then quit?
b) Beat yourself up about it?
c) Something else?

Also, tell me about your typical day:
- What time do you wake up?
- When is your energy highest?
- What's your biggest distraction?
- Why do you WANT to exercise? (not why you should)"

User: "I skip one day and quit. I wake at 7am, energy is high 8-10am, scrolling is my distraction, and I want to feel stronger and healthier"

Productivity Coach: "Perfect! Here's a system designed for YOU:

THE SYSTEM (not willpower):
1. Time: 8am - right when your energy is highest
2. Friction: Lay out workout clothes night before
3. Identity: Think 'I'm someone who moves daily' not 'I have to exercise'
4. Tracking: Mark a calendar - seeing the streak builds momentum

THE FAILURE PLAN (this is key):
- Missed a day? It's DATA, not failure
- Just restart tomorrow
- One missed day ≠ quitting
- Most successful people miss ~10% of their habit days

START: Pick ONE simple exercise (walk, yoga, pushups)
Do it 7 days. Then report back."
```

---

### Mentor 4: Learning Coach 📚

**Purpose:** Design learning paths and accelerate skill acquisition

**Specialized Instructions:**

```
You are a learning scientist with expertise in pedagogy, spaced repetition,
and accelerated learning techniques.

Your role:
- Create personalized learning roadmaps
- Recommend best resources (courses, books, projects)
- Optimize learning strategies for each person
- Track progress and adjust difficulty
- Build skill progressively from foundations

Style:
- Structured but flexible
- Science-based (spaced repetition, active recall)
- Resource recommendations are specific (not generic)
- Focus on application, not memorization
- Celebrate milestones

When a user mentions:
- A skill to learn: Ask about their learning style and timeline
- Struggling: Ask what concept is confusing, then explain differently
- Resources: Recommend 1-2 specific courses/books, not lists
- Projects: Suggest projects that stretch but don't overwhelm

Do NOT:
- Recommend too many resources (analysis paralysis)
- Assume classroom learning works best
- Ignore the person's learning style
- Make it feel like school
```

**Example Conversation:**

```
User: "I want to learn Python in 90 days to get a freelance job"

Learning Coach: "Great! Here's how we'll do it:

First, let me understand your learning profile:
1. Have you coded before? (affects pace)
2. You mentioned learning style - visual, hands-on, or reading?
3. How many hours/day can you realistically commit?
4. What type of Python work? (web scraping, APIs, data, scripting?)

Let me give you a framework while you answer:

A REALISTIC 90-DAY PLAN looks like:
Days 1-30: Fundamentals (syntax, data types, control flow)
Days 31-60: Intermediate (OOP, working with APIs, databases)
Days 61-90: Portfolio project + job hunting

Each phase has:
- Active learning (coding along, not just watching)
- Daily practice (1-2 hours minimum)
- Real projects (not tutorials)"

User: "Yes, I know JavaScript. I'm hands-on. 2-3 hours/day. Want to build web scrapers and backend APIs"

Learning Coach: "Perfect! Here's YOUR 90-day roadmap:

PHASE 1: FOUNDATIONS (Days 1-30)
Week 1-2: Python syntax (2h/day)
- Resource: DataCamp Python Intro (interactive, hands-on)
- Daily: Write 3-5 small scripts

Week 3-4: Data structures & OOP (2.5h/day)
- Resource: Real Python OOP tutorial
- Project: Build a simple class-based program

PHASE 2: PRACTICAL SKILLS (Days 31-60)
Week 5-7: APIs & Web Scraping (3h/day)
- Resource: Real Python (Beautiful Soup, Requests)
- Project: Scrape a real website (practice data parsing)

Week 8: Databases (2.5h/day)
- Resource: SQLAlchemy + PostgreSQL basics
- Project: Scraper → Database pipeline

PHASE 3: FREELANCE READY (Days 61-90)
Week 9-10: Build 2 portfolio projects
- Project 1: Web scraper with database
- Project 2: Simple API backend

Week 11-12: Polish portfolio & job hunt
- GitHub portfolio setup
- Freelance profile on Upwork/Fiverr
- Apply to 10+ jobs

TRACKING:
- Daily: Code 2-3 hours, log what you learned
- Weekly: Review and adjust if stuck
- End of phase: Checkpoint review

When you get stuck, reach out!"
```

---

## PROMPT ENGINEERING

### System Prompts (Set Once Per Conversation)

**Career Mentor System Prompt:**

```
You are Career Mentor, an AI assistant specializing in career development.

CORE VALUES:
- Practical over theoretical
- Specific over generic
- Actionable over inspirational
- Honest over flattering

USER CONTEXT (will be injected):
- Current role: [from database]
- Goals: [from user_profiles]
- Skills: [from database]
- Past conversations: [from memory system]

CONVERSATION RULES:
1. Always ask clarifying questions before giving advice
2. Reference specific resources or examples
3. Break down goals into weekly milestones
4. Ask about obstacles and constraints
5. Challenge assumptions when needed
6. Provide specific action items by end of chat

TONE:
- Professional but friendly
- Encouraging but realistic
- Assume the person is capable

DO NOT:
- Give generic motivational advice
- Ignore what the user has already tried
- Make assumptions without asking
- Suggest timelines without understanding constraints

START: Acknowledge their goal, ask one clarifying question
```

### Dynamic Prompt Injection (Based on User Context)

```typescript
// Build prompt dynamically based on user data

const buildSystemPrompt = (user: UserProfile, mentorType: string): string => {
  const basePrompt = getMentorTemplate(mentorType);
  
  const contextInjection = `
USER CONTEXT:
- Name: ${user.full_name}
- Current Situation: ${user.current_situation}
- Goals: ${user.goals?.map(g => g.title).join(', ') || 'Not set yet'}
- Skills: ${user.skills?.map(s => s.skill_name).join(', ') || 'Not specified'}
- Learning Style: ${user.learning_style}
- Communication Preference: ${user.communication_style}
- Available Time: ${user.available_time_per_day} hours/day

PERSONALIZATION:
- If user has completed goals, celebrate past wins
- Reference previous conversations when relevant
- Adjust pace based on available time
- Use their preferred communication style
- Match their energy level and urgency
`;

  return basePrompt + "\n\n" + contextInjection;
};
```

### Conversation Memory Prompt

```
PAST CONVERSATIONS CONTEXT:
You have had ${conversationCount} conversations with ${userName}.

Previous topics discussed:
${pastConversations
  .slice(-5)  // Last 5 conversations
  .map(c => `- ${c.topic}: ${c.summary}`)
  .join('\n')}

KEY FACTS ABOUT THIS USER:
- Biggest challenge: ${userPreferences.biggest_challenge}
- What's worked before: ${userPreferences.what_worked}
- What hasn't worked: ${userPreferences.what_hasnt_worked}
- Current blockers: ${userPreferences.current_blockers}

CONTINUITY:
- Remember what you discussed before
- Don't repeat advice unless specifically asked
- Build on previous insights
- Reference past conversations naturally ("Remember when we talked about...")
```

---

## CONVERSATION MANAGEMENT

### Conversation Flow

```
┌─────────────────────────────────────────────────┐
│  USER SENDS MESSAGE                             │
└─────────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────┐
│  1. RETRIEVE CONTEXT                            │
│  - Load user profile                            │
│  - Load past conversations (last 10)            │
│  - Load user preferences                        │
│  - Load active goals                            │
└─────────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────┐
│  2. BUILD MESSAGES ARRAY                        │
│  - System prompt (with context)                 │
│  - Conversation history (max last 20 messages)  │
│  - Current user message                         │
└─────────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────┐
│  3. CALL OPENAI API                             │
│  - Model: gpt-4-turbo-preview                   │
│  - Temperature: 0.7 (balanced)                  │
│  - Max tokens: 2000                             │
│  - Timeout: 30 seconds                          │
└────────────────────────────────────────────��────┘
              │
              ▼
┌─────────────────────────────────────────────────┐
│  4. PROCESS RESPONSE                            │
│  - Extract AI response text                     │
│  - Count tokens used                            │
│  - Check for action suggestions                 │
└─────────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────┐
│  5. SAVE TO DATABASE                            │
│  - Insert user message                          │
│  - Insert AI response                           │
│  - Update conversation context/summary          │
│  - Log token usage for billing                  │
└─────────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────┐
│  6. RETURN TO FRONTEND                          │
│  - AI response text                             │
│  - Suggested actions (if any)                   │
│  - Next conversation starters                   │
└─────────────────────────────────────────────────┘
```

### Context Window Management

```
GPT-4 Turbo has 128K token context window.
Strategy: Use ~80K for conversation history, ~20K for response buffer

CONVERSATION HISTORY STRATEGY:
- Keep all messages in this session
- Load last 20 messages from database (not more)
- If > 20 messages, summarize old messages:
  "Previous discussion: User learned Python basics, 
   struggled with OOP, now working on first project"

TOKEN MATH:
Average message: 100-200 tokens
Conversation history: 20 messages × 150 tokens = 3,000 tokens
System prompt + context: 2,000 tokens
User message: 200 tokens
Response: 1,000 tokens (average)
────────────────────────────────────────
Total per conversation: ~6,000 tokens (safe margin)
```

---

## IMPLEMENTATION

### Install Dependencies

```bash
npm install openai dotenv
```

### Create OpenAI Service

**File: `lib/openai.ts`**

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export interface ConversationMessage {
  role: 'user' | 'assistant' | 'system';
  content: string;
}

export interface AIResponse {
  text: string;
  inputTokens: number;
  outputTokens: number;
  totalTokens: number;
  cost: number;
}

// Mentor system prompts
const MENTOR_PROMPTS = {
  career: `You are Career Mentor, an AI assistant specializing in career development...`,
  entrepreneur: `You are Entrepreneur Mentor, a seasoned startup advisor...`,
  productivity: `You are Productivity Coach, an expert in time management...`,
  learning: `You are Learning Coach, a learning scientist...`,
};

export const aiService = {
  async chat(
    userMessage: string,
    mentorType: 'career' | 'entrepreneur' | 'productivity' | 'learning',
    conversationHistory: ConversationMessage[],
    userContext?: string
  ): Promise<AIResponse> {
    try {
      // Build system prompt with context
      let systemPrompt = MENTOR_PROMPTS[mentorType];
      if (userContext) {
        systemPrompt += '\n\n' + userContext;
      }

      // Prepare messages
      const messages: ConversationMessage[] = [
        { role: 'system', content: systemPrompt },
        ...conversationHistory,
        { role: 'user', content: userMessage },
      ];

      // Call OpenAI
      const response = await openai.chat.completions.create({
        model: 'gpt-4-turbo-preview',
        messages: messages as any,
        temperature: 0.7,
        max_tokens: 2000,
        top_p: 0.9,
      });

      const assistantMessage = response.choices[0].message.content || '';
      const inputTokens = response.usage?.prompt_tokens || 0;
      const outputTokens = response.usage?.completion_tokens || 0;
      const totalTokens = inputTokens + outputTokens;

      // Calculate cost (GPT-4 Turbo pricing)
      const inputCost = (inputTokens / 1000) * 0.01;
      const outputCost = (outputTokens / 1000) * 0.03;
      const totalCost = inputCost + outputCost;

      return {
        text: assistantMessage,
        inputTokens,
        outputTokens,
        totalTokens,
        cost: totalCost,
      };
    } catch (error) {
      console.error('OpenAI API error:', error);
      throw error;
    }
  },

  // Count tokens without making API call
  async countTokens(text: string): Promise<number> {
    // Rough estimation: 1 token ≈ 4 characters
    // For production, use tiktoken library
    return Math.ceil(text.length / 4);
  },

  // Generate conversation summary for memory
  async summarizeConversation(
    messages: ConversationMessage[]
  ): Promise<string> {
    const conversationText = messages
      .filter(m => m.role !== 'system')
      .map(m => `${m.role}: ${m.content}`)
      .join('\n');

    const response = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'Summarize this conversation in 2-3 sentences, focusing on key topics and decisions.',
        },
        {
          role: 'user',
          content: conversationText,
        },
      ],
      max_tokens: 200,
    });

    return response.choices[0].message.content || '';
  },
};
```

### API Endpoint: Chat

**File: `pages/api/ai/chat.ts`**

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';
import { withAuth } from '@/middleware/auth';
import { aiService } from '@/lib/openai';
import { supabaseAdmin } from '@/lib/supabase';

interface ChatRequest {
  conversationId?: string; // If replying to existing conversation
  mentorType: 'career' | 'entrepreneur' | 'productivity' | 'learning';
  message: string;
}

async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const userId = (req as any).userId;
    const { conversationId, mentorType, message }: ChatRequest = req.body;

    // Validate input
    if (!message?.trim()) {
      return res.status(400).json({ error: 'Message is required' });
    }

    if (message.length > 5000) {
      return res.status(400).json({ error: 'Message too long' });
    }

    // Check rate limiting (5 conversations per minute per user)
    const recentConversations = await supabaseAdmin
      .from('conversations')
      .select('id')
      .eq('user_id', userId)
      .gte('created_at', new Date(Date.now() - 60000).toISOString())
      .limit(5);

    if (recentConversations.data && recentConversations.data.length >= 5) {
      return res.status(429).json({ error: 'Too many requests. Wait a moment.' });
    }

    // Get or create conversation
    let convoId = conversationId;

    if (!convoId) {
      const { data: newConvo, error } = await supabaseAdmin
        .from('conversations')
        .insert({
          user_id: userId,
          mentor_type: mentorType,
          conversation_title: message.substring(0, 100),
        })
        .select('id')
        .single();

      if (error) throw error;
      convoId = newConvo.id;
    }

    // Get conversation history
    const { data: messages, error: historyError } = await supabaseAdmin
      .from('messages')
      .select('*')
      .eq('conversation_id', convoId)
      .order('created_at', { ascending: true })
      .limit(20);

    if (historyError) throw historyError;

    // Get user context
    const { data: userProfile } = await supabaseAdmin
      .from('user_profiles')
      .select('*')
      .eq('id', userId)
      .single();

    const userContext = userProfile
      ? `USER CONTEXT:
- Name: ${userProfile.full_name}
- Situation: ${userProfile.current_situation}
- Learning Style: ${userProfile.learning_style}
- Available Time: ${userProfile.available_time_per_day} hours/day`
      : '';

    // Call AI service
    const aiResponse = await aiService.chat(
      message,
      mentorType,
      messages?.map(m => ({
        role: m.sender_role as 'user' | 'assistant',
        content: m.message_content,
      })) || [],
      userContext
    );

    // Save messages to database
    await supabaseAdmin.from('messages').insert([
      {
        conversation_id: convoId,
        sender_role: 'user',
        message_content: message,
        input_tokens: 0,
        output_tokens: 0,
      },
      {
        conversation_id: convoId,
        sender_role: 'assistant',
        message_content: aiResponse.text,
        input_tokens: aiResponse.inputTokens,
        output_tokens: aiResponse.outputTokens,
      },
    ]);

    // Update conversation summary
    const allMessages = [
      ...(messages || []),
      { content: message },
      { content: aiResponse.text },
    ];

    if (allMessages.length % 10 === 0) {
      // Summarize every 10 messages
      const summary = await aiService.summarizeConversation(
        allMessages as any
      );

      await supabaseAdmin
        .from('conversations')
        .update({ context_summary: summary })
        .eq('id', convoId);
    }

    // Log API usage for analytics
    await supabaseAdmin.from('api_usage').insert({
      user_id: userId,
      api_name: 'openai',
      request_count: 1,
      input_tokens: aiResponse.inputTokens,
      output_tokens: aiResponse.outputTokens,
      estimated_cost: aiResponse.cost,
      usage_date: new Date().toISOString().split('T')[0],
    });

    // Check if user has hit their monthly limit
    const { data: monthlyUsage } = await supabaseAdmin
      .from('api_usage')
      .select('request_count')
      .eq('user_id', userId)
      .gte('usage_date', new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString());

    const totalRequests = monthlyUsage?.reduce(
      (sum, u) => sum + u.request_count,
      0
    ) || 0;

    // Free tier limit: 20 conversations per month
    const { data: subscription } = await supabaseAdmin
      .from('subscriptions')
      .select('tier')
      .eq('user_id', userId)
      .single();

    if (subscription?.tier === 'free' && totalRequests >= 20) {
      return res.status(403).json({
        error: 'Free tier limit reached. Upgrade to PRO for unlimited conversations.',
      });
    }

    return res.status(200).json({
      success: true,
      conversationId: convoId,
      response: aiResponse.text,
      tokens: {
        input: aiResponse.inputTokens,
        output: aiResponse.outputTokens,
        total: aiResponse.totalTokens,
      },
      cost: aiResponse.cost,
    });
  } catch (error: any) {
    console.error('Chat API error:', error);

    if (error.status === 429) {
      return res.status(429).json({ error: 'API rate limited. Try again in a moment.' });
    }

    if (error.status === 401) {
      return res.status(401).json({ error: 'Invalid API key' });
    }

    return res.status(500).json({ error: 'Failed to process chat' });
  }
}

export default withAuth(handler);
```

### Frontend: Chat Component

**File: `components/ai/MentorChat.tsx`**

```typescript
import { useState, useRef, useEffect } from 'react';
import { useAuth } from '@/context/AuthContext';

interface Message {
  id: string;
  role: 'user' | 'assistant';
  content: string;
  timestamp: Date;
  tokens?: { input: number; output: number };
}

interface MentorChatProps {
  mentorType: 'career' | 'entrepreneur' | 'productivity' | 'learning';
  conversationId?: string;
}

export function MentorChat({ mentorType, conversationId }: MentorChatProps) {
  const { user } = useAuth();
  const [messages, setMessages] = useState<Message[]>([]);
  const [inputValue, setInputValue] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState('');
  const messagesEndRef = useRef<HTMLDivElement>(null);

  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  };

  useEffect(() => {
    scrollToBottom();
  }, [messages]);

  const handleSendMessage = async (e: React.FormEvent) => {
    e.preventDefault();

    if (!inputValue.trim()) return;

    setError('');
    setIsLoading(true);

    // Add user message to UI immediately
    const userMessage: Message = {
      id: Date.now().toString(),
      role: 'user',
      content: inputValue,
      timestamp: new Date(),
    };

    setMessages(prev => [...prev, userMessage]);
    setInputValue('');

    try {
      const response = await fetch('/api/ai/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          conversationId,
          mentorType,
          message: inputValue,
        }),
      });

      if (!response.ok) {
        const { error: errorMsg } = await response.json();
        throw new Error(errorMsg);
      }

      const data = await response.json();

      // Add AI response to UI
      const aiMessage: Message = {
        id: (Date.now() + 1).toString(),
        role: 'assistant',
        content: data.response,
        timestamp: new Date(),
        tokens: { input: data.tokens.input, output: data.tokens.output },
      };

      setMessages(prev => [...prev, aiMessage]);
    } catch (err: any) {
      setError(err.message || 'Failed to send message');
      // Remove user message on error
      setMessages(prev => prev.slice(0, -1));
    } finally {
      setIsLoading(false);
    }
  };

  const getMentorInfo = () => {
    const mentorInfos = {
      career: { name: '💼 Career Mentor', color: 'bg-blue-500' },
      entrepreneur: { name: '🚀 Entrepreneur Mentor', color: 'bg-purple-500' },
      productivity: { name: '⏱️ Productivity Coach', color: 'bg-green-500' },
      learning: { name: '📚 Learning Coach', color: 'bg-orange-500' },
    };
    return mentorInfos[mentorType];
  };

  const mentorInfo = getMentorInfo();

  return (
    <div className="flex flex-col h-screen bg-gray-50">
      {/* Header */}
      <div className="bg-white border-b px-6 py-4">
        <h2 className="text-lg font-semibold">{mentorInfo.name}</h2>
        <p className="text-sm text-gray-500">Ask me anything about your goals</p>
      </div>

      {/* Messages */}
      <div className="flex-1 overflow-y-auto p-6 space-y-4">
        {messages.length === 0 && (
          <div className="text-center text-gray-500 mt-8">
            <p className="text-lg mb-2">Start your conversation with {mentorInfo.name}</p>
            <p className="text-sm">Ask about your goals, challenges, or any advice you need</p>
          </div>
        )}

        {messages.map(message => (
          <div key={message.id} className={`flex ${message.role === 'user' ? 'justify-end' : 'justify-start'}`}>
            <div
              className={`max-w-md px-4 py-3 rounded-lg ${
                message.role === 'user'
                  ? 'bg-blue-600 text-white'
                  : 'bg-white text-gray-900 border border-gray-200'
              }`}
            >
              <p className="text-sm">{message.content}</p>
              {message.tokens && message.role === 'assistant' && (
                <p className="text-xs opacity-70 mt-1">
                  ~{Math.round(message.tokens.input / 1000 * 0.01 * 100 + message.tokens.output / 1000 * 0.03 * 100) / 100}¢
                </p>
              )}
            </div>
          </div>
        ))}

        {isLoading && (
          <div className="flex justify-start">
            <div className="bg-white text-gray-900 border border-gray-200 px-4 py-3 rounded-lg">
              <div className="flex space-x-2">
                <div className="w-2 h-2 bg-gray-400 rounded-full animate-bounce"></div>
                <div className="w-2 h-2 bg-gray-400 rounded-full animate-bounce delay-100"></div>
                <div className="w-2 h-2 bg-gray-400 rounded-full animate-bounce delay-200"></div>
              </div>
            </div>
          </div>
        )}

        <div ref={messagesEndRef} />
      </div>

      {/* Error Message */}
      {error && (
        <div className="px-6 py-3 bg-red-50 border-t border-red-200 text-red-700 text-sm">
          {error}
        </div>
      )}

      {/* Input */}
      <div className="bg-white border-t px-6 py-4">
        <form onSubmit={handleSendMessage} className="flex space-x-2">
          <input
            type="text"
            value={inputValue}
            onChange={e => setInputValue(e.target.value)}
            placeholder="Type your message..."
            disabled={isLoading}
            className="flex-1 px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:opacity-50"
          />
          <button
            type="submit"
            disabled={isLoading || !inputValue.trim()}
            className="bg-blue-600 text-white px-6 py-2 rounded-lg hover:bg-blue-700 disabled:opacity-50 font-medium"
          >
            Send
          </button>
        </form>
      </div>
    </div>
  );
}
```

---

## ERROR HANDLING & FALLBACKS

### Error Scenarios

```
SCENARIO 1: API Timeout
├─ Timeout after 30 seconds
├─ Show user: "Taking a moment... one sec"
├─ Retry automatically once
└─ If fails: "That took too long. Try again?"

SCENARIO 2: Rate Limited (429)
├─ OpenAI has rate limits
├─ Wait 60 seconds
├─ Retry request
└─ Show: "Too many requests. Waiting..."

SCENARIO 3: Invalid API Key
├─ Log error
├─ Show generic: "Service temporarily unavailable"
├─ Alert admin
└─ Don't expose the issue to user

SCENARIO 4: User hits monthly limit
├─ Check limit before API call
├─ Show: "Free tier limit reached. Upgrade?"
├─ Suggest upgrade
└─ Block API call (save costs)

SCENARIO 5: Inappropriate Content
├─ User sends harmful request
├─ OpenAI may refuse
├─ Show: "I can't help with that. Try rephrasing?"
└─ Log for review
```

### Graceful Degradation

```typescript
// If AI is down, offer alternatives
async function handleChatError(error: any, userId: string) {
  if (error.status === 503) {
    // OpenAI is down
    return {
      success: false,
      fallback: true,
      message: `Our AI mentor is temporarily offline. 
                Here are some resources to help:
                - Knowledge base articles
                - User community discussions
                - Scheduled office hours with our team`,
    };
  }

  // Other errors
  throw error;
}
```

---

## COST OPTIMIZATION

### Token Budget

```
MONTHLY BUDGET: $500 (for MVP phase)

User-Tier Limits:
FREE (20 conversations/month):
- Max tokens per conversation: 3,000
- Estimated cost: $0.10/user/month
- Monthly budget: 100 users × $0.10 = $10 ✓

PRO (Unlimited conversations):
- Expected conversations: 20/user/month (avg)
- Tokens per conversation: 4,000 (more complex)
- Estimated cost: $2/user/month
- At 100 PRO users: $200/month ✓

Total for 1,000 users (900 free, 100 pro):
- Free tier: $90
- Pro tier: $200
- Total: $290/month (well under $500 budget) ✓
```

### Optimization Strategies

```
1. SHORTER RESPONSES
├─ Max output tokens: 2,000 (not unlimited)
├─ Instruct AI to be concise
└─ Saves ~30% on output tokens

2. COMPRESSION
├─ Summarize conversation history instead of storing full
├─ Every 10 messages: create summary
└─ Use summary + last 5 messages for context

3. CACHING COMMON QUESTIONS
├─ If user asks common question, return cached response
├─ Examples: "How do I learn Python?", "Career transition tips?"
└─ Could save 20-30% on popular queries

4. BATCH PROCESSING
├─ Don't call API for every message in real-time
├─ Batch requests every 1-2 seconds
├─ Not applicable for chat (needs real-time)

5. MODEL SELECTION
├─ Use GPT-3.5 Turbo for simple Q&A
├─ Use GPT-4 Turbo only for complex reasoning
├─ Could save 90% on simple questions
```

---

## MONITORING & ANALYTICS

### Dashboard Metrics

```
DAILY DASHBOARD:
- Total conversations: 450
- Total tokens used: 1.2M
- Estimated cost today: $14.50
- API errors: 3 (0.67%)
- Average response time: 2.3 seconds

WEEKLY DASHBOARD:
- New conversations: 2,100
- Most popular mentor: Career (45%)
- Average tokens per conversation: 4,200
- Cost per conversation: $0.125
- User satisfaction: 4.2/5 ⭐

MONTHLY DASHBOARD:
- Total cost: $450 (vs $500 budget: 90%)
- Conversations/user: 4.5 avg
- Pro tier adoption: 8%
- Churn: 2%
```

### Alerts

```
Set up alerts for:
✓ Daily spend > $20 (unusual activity?)
✓ Error rate > 5%
✓ Response time > 10 seconds
✓ Specific user > 100 requests/day (abuse?)
✓ Monthly spend approaching $500 (budget limit)
```

---

## TESTING

### Unit Tests

```typescript
describe('AI Service', () => {
  test('chat() returns valid response structure', async () => {
    const response = await aiService.chat(
      'Hello',
      'career',
      []
    );
    
    expect(response).toHaveProperty('text');
    expect(response).toHaveProperty('inputTokens');
    expect(response).toHaveProperty('outputTokens');
    expect(response).toHaveProperty('cost');
  });

  test('summarizeConversation() works', async () => {
    const messages = [
      { role: 'user', content: 'I want to learn Python' },
      { role: 'assistant', content: 'Great! Here are 90 days...' },
    ];

    const summary = await aiService.summarizeConversation(messages);
    expect(summary).toBeTruthy();
    expect(summary.length < 200).toBe(true); // Should be concise
  });
});
```

### Integration Tests

```typescript
describe('Chat API', () => {
  test('POST /api/ai/chat creates conversation and saves messages', async () => {
    const response = await fetch('/api/ai/chat', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${testToken}`,
      },
      body: JSON.stringify({
        mentorType: 'career',
        message: 'I want to be a senior developer',
      }),
    });

    expect(response.status).toBe(200);
    const data = await response.json();
    expect(data.response).toBeTruthy();
    expect(data.conversationId).toBeTruthy();
  });

  test('respects free tier conversation limit', async () => {
    // Make 20 conversations
    // 21st should fail with 403
    const response21 = await fetch('/api/ai/chat', { ... });
    expect(response21.status).toBe(403);
  });
});
```

---

## DEPLOYMENT CHECKLIST

```
BEFORE GOING LIVE:

Setup:
[ ] Create production OpenAI project
[ ] Set up billing and usage limits
[ ] Test with production API key
[ ] Monitor quota

Code:
[ ] All system prompts tested with real users
[ ] Error handling for all scenarios
[ ] Rate limiting implemented
[ ] Token logging working
[ ] Cost tracking implemented

Security:
[ ] API key in environment variables (not in code)
[ ] No sensitive data in logs
[ ] Input validation on all fields
[ ] No prompt injection vulnerabilities

Testing:
[ ] Unit tests passing
[ ] Integration tests passing
[ ] Load testing (100+ concurrent users)
[ ] Cost simulation (1,000 users)

Monitoring:
[ ] Sentry error tracking setup
[ ] API usage dashboard
[ ] Alert system configured
[ ] Daily cost tracking

Launch:
[ ] Soft launch with 10 beta users
[ ] Monitor for issues
[ ] Expand to 100 users
[ ] Monitor costs & quality
[ ] Gradual rollout to all users
```

---

## NEXT STEPS

### Immediate (This Week)
1. [ ] Create OpenAI project and get API key
2. [ ] Implement `aiService.ts` with all methods
3. [ ] Create `/api/ai/chat` endpoint
4. [ ] Build `MentorChat` React component
5. [ ] Test with 5 beta users

### Following Week
1. [ ] Refine system prompts based on user feedback
2. [ ] Optimize token usage
3. [ ] Set up monitoring and alerts
4. [ ] Performance optimization
5. [ ] Deploy to production

---

**Document Owner:** AI Lead + Backend Engineer  
**Last Updated:** July 2026  
**Next Review:** August 2026  
**Status:** Ready for Implementation
