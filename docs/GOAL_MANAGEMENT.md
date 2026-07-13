# LifeOS AI — GOAL MANAGEMENT SYSTEM IMPLEMENTATION (Phase 6)

**Version:** 1.0  
**Date:** July 2026  
**Status:** Ready for Development

---

## EXECUTIVE SUMMARY

This document provides complete implementation guidance for LifeOS AI's core feature: the Goal Management System. It covers:
- **Goal Creation** – AI-powered 90-day roadmap generation
- **Milestone Tracking** – Weekly progress milestones
- **Daily Tasks** – Actionable daily work
- **Progress Tracking** – Real-time completion metrics
- **Reflection System** – Weekly learning & adjustment
- **Implementation** – Complete backend & frontend code

---

## GOAL MANAGEMENT ARCHITECTURE

```
┌──────────────────────────────────────────────────────┐
│               GOAL LIFECYCLE                          │
└────────────────────────────────────────────���─────────┘

STEP 1: Goal Creation
┌────────────────────────────────────────────────────┐
│ User enters: "Learn Python & get first client"    │
│ AI generates: 90-day roadmap with phases          │
│ User confirms: "Yes, let's do this!"              │
└────────────────────────────────────────────────────┘
              │
              ▼
STEP 2: Milestone Setup
┌────────────────────────────────────────────────────┐
│ AI breaks goal into 12 weekly milestones           │
│ Week 1: "Learn Python basics"                      │
│ Week 2: "Master functions & OOP"                   │
│ ... etc                                            │
└────────────────────────────────────────────────────┘
              │
              ▼
STEP 3: Daily Tasks
┌────────────────────────────────────────────────────┐
│ AI generates daily tasks per milestone             │
│ Mon: 2-hour Python session                         │
│ Tue: Review functions concepts                     │
│ Wed: Start mini project                            │
│ ... etc                                            │
└────────────────────────────────────────────────────┘
              │
              ▼
STEP 4: Daily Execution
┌────────────────────────────────────────────────────┐
│ User completes tasks daily                         │
│ Logs completion status                             │
│ Sees streak counter 🔥                             │
└────────────────────────────────────────────────────┘
              │
              ▼
STEP 5: Weekly Reflection
┌────────────────────────────────────────────��───────┐
│ User reflects: What went well? What was hard?      │
│ AI provides feedback & adjustments                 │
│ Next milestone unlocked                            │
└────────────────────────────────────────────────────┘
              │
              ▼
STEP 6: Progress Tracking
┌────────────────────────────────────────────────────┐
│ Dashboard shows: 45% complete, 45 days left        │
│ Milestone progress: 4/10 done                      │
│ Streak: 15 days 🔥                                │
└────────────────────────────────────────────────────┘
              │
              ▼
STEP 7: Goal Completion
┌────────────────────────────────────────────────────┐
│ User hits 100% completion                          │
│ Goal archived with celebration 🎉                  │
│ Insights: "You invested 45 hours"                  │
└────────────────────────────────────────────────────┘
```

---

## AI ROADMAP GENERATION

### Prompt for Roadmap Generation

**File: `lib/prompts/goalRoadmap.ts`**

```typescript
export const generateRoadmapPrompt = (goal: {
  title: string;
  description: string;
  timeline_days: number;
  user_context: string;
}): string => {
  return `You are an expert goal coach creating a detailed 90-day roadmap.

USER GOAL: "${goal.title}"
CONTEXT: ${goal.description}
TIMELINE: ${goal.timeline_days} days
USER INFO: ${goal.user_context}

CREATE A STRUCTURED ROADMAP with this exact JSON format:

{
  "phases": [
    {
      "phase_number": 1,
      "phase_name": "Foundation Building",
      "duration_days": 30,
      "description": "Build core skills and foundational knowledge",
      "key_milestones": [
        "Complete fundamentals course",
        "Build first project",
        "Document learnings"
      ],
      "success_metrics": [
        "Completed all lessons",
        "Project deployed",
        "README written"
      ]
    },
    {
      "phase_number": 2,
      "phase_name": "Skill Development",
      "duration_days": 30,
      "description": "Develop intermediate skills",
      "key_milestones": [...],
      "success_metrics": [...]
    },
    {
      "phase_number": 3,
      "phase_name": "Goal Achievement",
      "duration_days": 30,
      "description": "Execute final push to achieve goal",
      "key_milestones": [...],
      "success_metrics": [...]
    }
  ],
  "weekly_milestones": [
    {
      "week": 1,
      "title": "Learn Python basics",
      "description": "Master variables, data types, and control flow",
      "tasks": [
        { "day": "Monday", "task": "2hr Python session: syntax & variables" },
        { "day": "Tuesday", "task": "Practice: 20 coding problems" },
        { "day": "Wednesday", "task": "Learn: loops and conditionals" },
        { "day": "Thursday", "task": "Build: simple calculator app" },
        { "day": "Friday", "task": "Review and reflection" }
      ]
    },
    ...13 more weeks
  ],
  "resources": [
    {
      "name": "Real Python",
      "type": "course",
      "url": "https://...",
      "estimated_time": "10 hours",
      "priority": "high"
    }
  ],
  "success_indicators": [
    "Can explain core concepts",
    "Completed all milestones",
    "Built working projects"
  ],
  "potential_obstacles": [
    "Concept confusion at week 3-4",
    "Time management challenges",
    "Motivation dips at week 6"
  ],
  "mitigation_strategies": [
    "Schedule office hours to clarify concepts",
    "Use time blocking for 2-hour sessions",
    "Join community groups for accountability"
  ]
}

REQUIREMENTS:
- Be SPECIFIC (not vague)
- Break down into 12-13 weekly milestones
- Each week has 5 daily tasks (Mon-Fri)
- Include real, named resources
- Address common obstacles
- Be REALISTIC about timeline
- Include rest days for sustainability

Return ONLY valid JSON, no markdown or explanation.`;
};
```

### Roadmap Generation Service

**File: `lib/goalService.ts`**

```typescript
import { aiService } from './openai';
import { supabaseAdmin } from './supabase';

interface RoadmapResponse {
  phases: Phase[];
  weekly_milestones: WeeklyMilestone[];
  resources: Resource[];
  success_indicators: string[];
  potential_obstacles: string[];
  mitigation_strategies: string[];
}

interface Phase {
  phase_number: number;
  phase_name: string;
  duration_days: number;
  description: string;
  key_milestones: string[];
  success_metrics: string[];
}

interface WeeklyMilestone {
  week: number;
  title: string;
  description: string;
  tasks: DailyTask[];
}

interface DailyTask {
  day: string;
  task: string;
}

interface Resource {
  name: string;
  type: string;
  url: string;
  estimated_time: string;
  priority: string;
}

export const goalService = {
  // Generate AI roadmap for new goal
  async generateRoadmap(
    userId: string,
    goalData: {
      title: string;
      description: string;
      timeline_days: number;
      category: string;
    }
  ): Promise<RoadmapResponse> {
    try {
      // Get user context
      const { data: userProfile } = await supabaseAdmin
        .from('user_profiles')
        .select('*')
        .eq('id', userId)
        .single();

      const userContext = `
Current Situation: ${userProfile.current_situation}
Available Time: ${userProfile.available_time_per_day} hours/day
Learning Style: ${userProfile.learning_style}
Skills: ${userProfile.skills?.map((s: any) => s.skill_name).join(', ') || 'Not specified'}
      `.trim();

      // Build prompt
      const prompt = generateRoadmapPrompt({
        title: goalData.title,
        description: goalData.description,
        timeline_days: goalData.timeline_days,
        user_context: userContext,
      });

      // Call AI
      const response = await aiService.chat(
        prompt,
        'learning', // Use learning coach for roadmap generation
        []
      );

      // Parse JSON response
      let roadmap: RoadmapResponse;
      try {
        roadmap = JSON.parse(response.text);
      } catch (parseError) {
        // Try to extract JSON from response
        const jsonMatch = response.text.match(/\{[\s\S]*\}/);
        if (jsonMatch) {
          roadmap = JSON.parse(jsonMatch[0]);
        } else {
          throw parseError;
        }
      }

      return roadmap;
    } catch (error) {
      console.error('Roadmap generation error:', error);
      throw error;
    }
  },

  // Save generated roadmap to database
  async saveGoalWithRoadmap(
    userId: string,
    goalData: {
      title: string;
      description: string;
      category: string;
      timeline_days: number;
      why_important: string;
    },
    roadmap: RoadmapResponse
  ) {
    try {
      const startDate = new Date();
      const targetDate = new Date();
      targetDate.setDate(targetDate.getDate() + goalData.timeline_days);

      // 1. Create goal
      const { data: goal, error: goalError } = await supabaseAdmin
        .from('goals')
        .insert({
          user_id: userId,
          goal_title: goalData.title,
          goal_description: goalData.description,
          category: goalData.category,
          timeline_days: goalData.timeline_days,
          start_date: startDate.toISOString().split('T')[0],
          target_date: targetDate.toISOString().split('T')[0],
          status: 'active',
          roadmap_json: roadmap,
          completion_percentage: 0,
        })
        .select()
        .single();

      if (goalError) throw goalError;

      // 2. Create milestones
      const milestones = roadmap.weekly_milestones.map((wm, index) => {
        const milestoneDate = new Date(startDate);
        milestoneDate.setDate(milestoneDate.getDate() + (wm.week * 7));

        return {
          goal_id: goal.id,
          milestone_number: wm.week,
          milestone_title: wm.title,
          milestone_description: wm.description,
          target_date: milestoneDate.toISOString().split('T')[0],
          is_completed: false,
        };
      });

      const { error: milestonesError } = await supabaseAdmin
        .from('milestones')
        .insert(milestones);

      if (milestonesError) throw milestonesError;

      // 3. Create daily tasks
      const dailyTasks: any[] = [];

      roadmap.weekly_milestones.forEach((wm) => {
        const dayOfWeekMap: { [key: string]: number } = {
          'Monday': 1,
          'Tuesday': 2,
          'Wednesday': 3,
          'Thursday': 4,
          'Friday': 5,
          'Saturday': 6,
          'Sunday': 0,
        };

        wm.tasks.forEach((task) => {
          dailyTasks.push({
            goal_id: goal.id,
            task_title: task.task,
            task_description: '',
            frequency: 'weekly',
            days_of_week: [dayOfWeekMap[task.day]],
            preferred_time_of_day: 'morning',
            estimated_duration_minutes: 120,
            is_active: true,
          });
        });
      });

      const { error: tasksError } = await supabaseAdmin
        .from('daily_tasks')
        .insert(dailyTasks);

      if (tasksError) throw tasksError;

      // 4. Create user streak
      const { error: streakError } = await supabaseAdmin
        .from('user_streaks')
        .upsert({
          user_id: userId,
          current_streak_days: 0,
          longest_streak_days: 0,
        });

      if (streakError) throw streakError;

      return { success: true, goal };
    } catch (error) {
      console.error('Save goal error:', error);
      throw error;
    }
  },

  // Create goal from user input
  async createGoal(
    userId: string,
    goalInput: {
      title: string;
      description: string;
      category: string;
      timeline_days: number;
      why_important: string;
    }
  ) {
    try {
      // Generate roadmap
      const roadmap = await this.generateRoadmap(userId, goalInput);

      // Save to database
      const result = await this.saveGoalWithRoadmap(userId, goalInput, roadmap);

      return result;
    } catch (error) {
      console.error('Create goal error:', error);
      throw error;
    }
  },

  // Get goal with all related data
  async getGoalWithDetails(goalId: string) {
    try {
      // Get goal
      const { data: goal } = await supabaseAdmin
        .from('goals')
        .select('*')
        .eq('id', goalId)
        .single();

      // Get milestones
      const { data: milestones } = await supabaseAdmin
        .from('milestones')
        .select('*')
        .eq('goal_id', goalId)
        .order('milestone_number', { ascending: true });

      // Get daily tasks
      const { data: tasks } = await supabaseAdmin
        .from('daily_tasks')
        .select('*')
        .eq('goal_id', goalId);

      // Get today's task completions
      const today = new Date().toISOString().split('T')[0];
      const { data: todaysCompletions } = await supabaseAdmin
        .from('task_completions')
        .select('*')
        .eq('completion_date', today)
        .in('task_id', tasks?.map((t: any) => t.id) || []);

      // Get reflections
      const { data: reflections } = await supabaseAdmin
        .from('reflections')
        .select('*')
        .eq('goal_id', goalId)
        .order('reflection_week', { ascending: false });

      return {
        goal,
        milestones,
        tasks,
        todaysCompletions,
        reflections,
      };
    } catch (error) {
      console.error('Get goal details error:', error);
      throw error;
    }
  },

  // Update task completion
  async completeTask(
    taskId: string,
    completionDate: string = new Date().toISOString().split('T')[0]
  ) {
    try {
      const { data, error } = await supabaseAdmin
        .from('task_completions')
        .upsert({
          task_id: taskId,
          completion_date: completionDate,
          was_completed: true,
        })
        .select()
        .single();

      if (error) throw error;

      return { success: true, completion: data };
    } catch (error) {
      console.error('Complete task error:', error);
      throw error;
    }
  },

  // Mark milestone as complete
  async completeMilestone(milestoneId: string) {
    try {
      const { data: milestone } = await supabaseAdmin
        .from('milestones')
        .select('goal_id')
        .eq('id', milestoneId)
        .single();

      const { data, error } = await supabaseAdmin
        .from('milestones')
        .update({
          is_completed: true,
          completed_at: new Date().toISOString(),
        })
        .eq('id', milestoneId)
        .select()
        .single();

      if (error) throw error;

      // Update goal completion percentage
      const { data: allMilestones } = await supabaseAdmin
        .from('milestones')
        .select('id, is_completed')
        .eq('goal_id', milestone.goal_id);

      const completedCount = allMilestones?.filter(m => m.is_completed).length || 0;
      const totalCount = allMilestones?.length || 1;
      const completionPercentage = Math.round((completedCount / totalCount) * 100);

      await supabaseAdmin
        .from('goals')
        .update({ completion_percentage: completionPercentage })
        .eq('id', milestone.goal_id);

      return { success: true, milestone: data };
    } catch (error) {
      console.error('Complete milestone error:', error);
      throw error;
    }
  },

  // Add weekly reflection
  async addReflection(
    goalId: string,
    reflectionWeek: number,
    reflection: {
      what_went_well: string;
      what_was_hard: string;
      what_to_improve: string;
    }
  ) {
    try {
      const { data, error } = await supabaseAdmin
        .from('reflections')
        .upsert({
          goal_id: goalId,
          reflection_week: reflectionWeek,
          reflection_date: new Date().toISOString().split('T')[0],
          ...reflection,
        })
        .select()
        .single();

      if (error) throw error;

      return { success: true, reflection: data };
    } catch (error) {
      console.error('Add reflection error:', error);
      throw error;
    }
  },

  // Update streak
  async updateStreak(userId: string, completedToday: boolean) {
    try {
      if (!completedToday) {
        // Reset streak
        await supabaseAdmin
          .from('user_streaks')
          .update({ current_streak_days: 0 })
          .eq('user_id', userId);
      } else {
        // Increment streak
        const { data: current } = await supabaseAdmin
          .from('user_streaks')
          .select('*')
          .eq('user_id', userId)
          .single();

        const newStreak = (current?.current_streak_days || 0) + 1;
        const longestStreak = Math.max(
          newStreak,
          current?.longest_streak_days || 0
        );

        await supabaseAdmin
          .from('user_streaks')
          .update({
            current_streak_days: newStreak,
            longest_streak_days: longestStreak,
            last_check_in_date: new Date().toISOString().split('T')[0],
          })
          .eq('user_id', userId);
      }

      return { success: true };
    } catch (error) {
      console.error('Update streak error:', error);
      throw error;
    }
  },

  // Get goal analytics
  async getGoalAnalytics(goalId: string) {
    try {
      // Get all task completions
      const { data: completions } = await supabaseAdmin
        .from('task_completions')
        .select('*')
        .in(
          'task_id',
          (
            await supabaseAdmin
              .from('daily_tasks')
              .select('id')
              .eq('goal_id', goalId)
          ).data?.map((t: any) => t.id) || []
        );

      const totalTasks = completions?.length || 0;
      const completedTasks =
        completions?.filter((c: any) => c.was_completed).length || 0;
      const completionRate =
        totalTasks > 0 ? Math.round((completedTasks / totalTasks) * 100) : 0;

      // Get daily breakdown
      const dailyBreakdown: { [key: string]: number } = {};
      completions?.forEach((c: any) => {
        dailyBreakdown[c.completion_date] =
          (dailyBreakdown[c.completion_date] || 0) +
          (c.was_completed ? 1 : 0);
      });

      return {
        total_tasks: totalTasks,
        completed_tasks: completedTasks,
        completion_rate: completionRate,
        daily_breakdown: dailyBreakdown,
      };
    } catch (error) {
      console.error('Get analytics error:', error);
      throw error;
    }
  },
};
```

---

## API ENDPOINTS

### Create Goal

**File: `pages/api/goals/create.ts`**

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';
import { withAuth } from '@/middleware/auth';
import { goalService } from '@/lib/goalService';

interface CreateGoalRequest {
  title: string;
  description: string;
  category: string;
  timeline_days: number;
  why_important: string;
}

async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const userId = (req as any).userId;
    const {
      title,
      description,
      category,
      timeline_days,
      why_important,
    }: CreateGoalRequest = req.body;

    // Validate
    if (!title || !category || !timeline_days) {
      return res.status(400).json({ error: 'Missing required fields' });
    }

    if (timeline_days < 7 || timeline_days > 365) {
      return res.status(400).json({
        error: 'Timeline must be between 7 and 365 days',
      });
    }

    // Check user's subscription tier for goal limits
    const { data: subscription } = await supabaseAdmin
      .from('subscriptions')
      .select('tier')
      .eq('user_id', userId)
      .single();

    const { data: existingGoals } = await supabaseAdmin
      .from('goals')
      .select('id')
      .eq('user_id', userId)
      .eq('status', 'active');

    const maxGoals = subscription?.tier === 'pro' ? 99 : 3;
    if ((existingGoals?.length || 0) >= maxGoals) {
      return res.status(403).json({
        error: `You've reached your goal limit. ${
          subscription?.tier === 'pro'
            ? 'Contact support'
            : 'Upgrade to PRO for unlimited goals'
        }`,
      });
    }

    // Create goal with AI-generated roadmap
    const result = await goalService.createGoal(userId, {
      title,
      description,
      category,
      timeline_days,
      why_important,
    });

    return res.status(201).json({
      success: true,
      goal: result.goal,
      message: 'Goal created with AI-generated roadmap!',
    });
  } catch (error: any) {
    console.error('Create goal error:', error);
    return res.status(500).json({ error: 'Failed to create goal' });
  }
}

export default withAuth(handler);
```

### Get Goal Details

**File: `pages/api/goals/[goalId].ts`**

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';
import { withAuth } from '@/middleware/auth';
import { goalService } from '@/lib/goalService';

async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'GET') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const userId = (req as any).userId;
    const { goalId } = req.query;

    if (!goalId || typeof goalId !== 'string') {
      return res.status(400).json({ error: 'Goal ID required' });
    }

    // Verify user owns this goal
    const { data: goal } = await supabaseAdmin
      .from('goals')
      .select('user_id')
      .eq('id', goalId)
      .single();

    if (!goal || goal.user_id !== userId) {
      return res.status(403).json({ error: 'Unauthorized' });
    }

    const goalDetails = await goalService.getGoalWithDetails(goalId);

    return res.status(200).json({
      success: true,
      data: goalDetails,
    });
  } catch (error: any) {
    console.error('Get goal error:', error);
    return res.status(500).json({ error: 'Failed to get goal' });
  }
}

export default withAuth(handler);
```

### Complete Task

**File: `pages/api/tasks/[taskId]/complete.ts`**

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';
import { withAuth } from '@/middleware/auth';
import { goalService } from '@/lib/goalService';

async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const userId = (req as any).userId;
    const { taskId } = req.query;
    const { completionDate } = req.body;

    if (!taskId || typeof taskId !== 'string') {
      return res.status(400).json({ error: 'Task ID required' });
    }

    // Verify user owns this task (through goal)
    const { data: task } = await supabaseAdmin
      .from('daily_tasks')
      .select('goal_id')
      .eq('id', taskId)
      .single();

    const { data: goal } = await supabaseAdmin
      .from('goals')
      .select('user_id')
      .eq('id', task.goal_id)
      .single();

    if (goal.user_id !== userId) {
      return res.status(403).json({ error: 'Unauthorized' });
    }

    // Complete task
    const result = await goalService.completeTask(
      taskId,
      completionDate || new Date().toISOString().split('T')[0]
    );

    // Check if all tasks completed today for streak update
    const today = new Date().toISOString().split('T')[0];
    const { data: todaysTasks } = await supabaseAdmin
      .from('daily_tasks')
      .select('id')
      .eq('goal_id', task.goal_id);

    const { data: completedToday } = await supabaseAdmin
      .from('task_completions')
      .select('*')
      .eq('completion_date', today)
      .in('task_id', todaysTasks?.map((t: any) => t.id) || []);

    const allCompleted =
      completedToday?.length === todaysTasks?.length &&
      completedToday?.every((c: any) => c.was_completed);

    if (allCompleted) {
      await goalService.updateStreak(userId, true);
    }

    return res.status(200).json({
      success: true,
      completion: result.completion,
      streak_updated: allCompleted,
    });
  } catch (error: any) {
    console.error('Complete task error:', error);
    return res.status(500).json({ error: 'Failed to complete task' });
  }
}

export default withAuth(handler);
```

### Add Reflection

**File: `pages/api/goals/[goalId]/reflection.ts`**

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';
import { withAuth } from '@/middleware/auth';
import { goalService } from '@/lib/goalService';
import { aiService } from '@/lib/openai';

async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const userId = (req as any).userId;
    const { goalId } = req.query;
    const {
      reflection_week,
      what_went_well,
      what_was_hard,
      what_to_improve,
    } = req.body;

    // Verify user owns this goal
    const { data: goal } = await supabaseAdmin
      .from('goals')
      .select('user_id')
      .eq('id', goalId)
      .single();

    if (goal.user_id !== userId) {
      return res.status(403).json({ error: 'Unauthorized' });
    }

    // Save reflection
    const result = await goalService.addReflection(goalId, reflection_week, {
      what_went_well,
      what_was_hard,
      what_to_improve,
    });

    // Get AI feedback on reflection
    const feedbackPrompt = `
User is reflecting on Week ${reflection_week} of their goal.

What went well:
${what_went_well}

What was hard:
${what_was_hard}

What to improve:
${what_to_improve}

Provide 2-3 specific, actionable suggestions based on their reflection.
Keep it to 2-3 sentences.
Be encouraging but honest.
`;

    const feedback = await aiService.chat(
      feedbackPrompt,
      'productivity',
      []
    );

    return res.status(200).json({
      success: true,
      reflection: result.reflection,
      ai_feedback: feedback.text,
    });
  } catch (error: any) {
    console.error('Add reflection error:', error);
    return res.status(500).json({ error: 'Failed to add reflection' });
  }
}

export default withAuth(handler);
```

### Get User's Goals

**File: `pages/api/goals/list.ts`**

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';
import { withAuth } from '@/middleware/auth';
import { supabaseAdmin } from '@/lib/supabase';

async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'GET') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const userId = (req as any).userId;
    const { status = 'active', sort = 'created_at' } = req.query;

    let query = supabaseAdmin
      .from('goals')
      .select('*')
      .eq('user_id', userId);

    if (status && status !== 'all') {
      query = query.eq('status', status);
    }

    const { data: goals, error } = await query.order(sort as string, {
      ascending: false,
    });

    if (error) throw error;

    // Enrich with milestone counts
    const enrichedGoals = await Promise.all(
      (goals || []).map(async (goal) => {
        const { data: milestones } = await supabaseAdmin
          .from('milestones')
          .select('id, is_completed')
          .eq('goal_id', goal.id);

        const totalMilestones = milestones?.length || 0;
        const completedMilestones =
          milestones?.filter((m: any) => m.is_completed).length || 0;

        return {
          ...goal,
          milestones: {
            total: totalMilestones,
            completed: completedMilestones,
          },
        };
      })
    );

    return res.status(200).json({
      success: true,
      goals: enrichedGoals,
    });
  } catch (error: any) {
    console.error('List goals error:', error);
    return res.status(500).json({ error: 'Failed to list goals' });
  }
}

export default withAuth(handler);
```

---

## FRONTEND COMPONENTS

### Goal Creation Form

**File: `components/goals/CreateGoalForm.tsx`**

```typescript
import { useState } from 'react';
import { useRouter } from 'next/router';

export function CreateGoalForm() {
  const router = useRouter();
  const [formData, setFormData] = useState({
    title: '',
    description: '',
    category: 'career',
    timeline_days: 90,
    why_important: '',
  });
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState('');

  const categories = [
    'Career',
    'Skills',
    'Productivity',
    'Business',
    'Health',
    'Personal',
  ];

  const timelineOptions = [30, 60, 90, 180];

  const handleChange = (
    e: React.ChangeEvent<
      HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement
    >
  ) => {
    const { name, value } = e.target;
    setFormData((prev) => ({
      ...prev,
      [name]:
        name === 'timeline_days' ? parseInt(value) : value.toLowerCase(),
    }));
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError('');
    setIsLoading(true);

    try {
      const response = await fetch('/api/goals/create', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData),
      });

      if (!response.ok) {
        const { error: errorMsg } = await response.json();
        throw new Error(errorMsg);
      }

      const { goal } = await response.json();
      router.push(`/goals/${goal.id}`);
    } catch (err: any) {
      setError(err.message || 'Failed to create goal');
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="max-w-2xl mx-auto space-y-6">
      <div>
        <label className="block text-sm font-medium mb-2">Goal Title *</label>
        <input
          type="text"
          name="title"
          value={formData.title}
          onChange={handleChange}
          placeholder="e.g., Learn Python & get first freelance client"
          className="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
          required
          disabled={isLoading}
        />
        <p className="text-xs text-gray-500 mt-1">Be specific and measurable</p>
      </div>

      <div>
        <label className="block text-sm font-medium mb-2">Description</label>
        <textarea
          name="description"
          value={formData.description}
          onChange={handleChange}
          placeholder="Why do you want to achieve this? What will it enable?"
          className="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 h-32"
          disabled={isLoading}
        />
      </div>

      <div className="grid grid-cols-2 gap-4">
        <div>
          <label className="block text-sm font-medium mb-2">Category *</label>
          <select
            name="category"
            value={formData.category}
            onChange={handleChange}
            className="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
            disabled={isLoading}
          >
            {categories.map((cat) => (
              <option key={cat} value={cat.toLowerCase()}>
                {cat}
              </option>
            ))}
          </select>
        </div>

        <div>
          <label className="block text-sm font-medium mb-2">
            Timeline *
          </label>
          <select
            name="timeline_days"
            value={formData.timeline_days}
            onChange={handleChange}
            className="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
            disabled={isLoading}
          >
            {timelineOptions.map((days) => (
              <option key={days} value={days}>
                {days} days
              </option>
            ))}
          </select>
        </div>
      </div>

      <div>
        <label className="block text-sm font-medium mb-2">
          Why is this important? *
        </label>
        <textarea
          name="why_important"
          value={formData.why_important}
          onChange={handleChange}
          placeholder="How will achieving this goal improve your life?"
          className="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 h-24"
          required
          disabled={isLoading}
        />
      </div>

      {error && (
        <div className="p-3 bg-red-50 border border-red-200 rounded-lg text-red-700 text-sm">
          {error}
        </div>
      )}

      <button
        type="submit"
        disabled={isLoading}
        className="w-full bg-blue-600 text-white py-3 rounded-lg font-medium hover:bg-blue-700 disabled:opacity-50"
      >
        {isLoading ? 'Creating AI-powered roadmap...' : 'Create Goal'}
      </button>
    </form>
  );
}
```

### Goal Details Page

**File: `components/goals/GoalDetails.tsx`**

```typescript
import { useState, useEffect } from 'react';
import { useRouter } from 'next/router';

interface GoalData {
  goal: any;
  milestones: any[];
  tasks: any[];
  todaysCompletions: any[];
  reflections: any[];
}

export function GoalDetails({ goalId }: { goalId: string }) {
  const [goalData, setGoalData] = useState<GoalData | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState('');

  useEffect(() => {
    const fetchGoal = async () => {
      try {
        const response = await fetch(`/api/goals/${goalId}`);
        if (!response.ok) throw new Error('Failed to load goal');
        const { data } = await response.json();
        setGoalData(data);
      } catch (err: any) {
        setError(err.message);
      } finally {
        setIsLoading(false);
      }
    };

    fetchGoal();
  }, [goalId]);

  if (isLoading) {
    return <div className="text-center py-12">Loading...</div>;
  }

  if (error || !goalData) {
    return (
      <div className="text-center py-12 text-red-600">
        Error: {error || 'Could not load goal'}
      </div>
    );
  }

  const { goal, milestones, tasks, reflections } = goalData;
  const daysLeft = Math.ceil(
    (new Date(goal.target_date).getTime() - Date.now()) / (1000 * 60 * 60 * 24)
  );

  return (
    <div className="max-w-4xl mx-auto space-y-8">
      {/* Header */}
      <div className="bg-white rounded-lg border p-6">
        <h1 className="text-3xl font-bold mb-2">{goal.goal_title}</h1>
        <p className="text-gray-600 mb-4">{goal.goal_description}</p>

        <div className="flex items-center justify-between mb-6">
          <div>
            <p className="text-sm text-gray-600">Status: {goal.status}</p>
            <p className="text-sm text-gray-600">{daysLeft} days remaining</p>
          </div>
          <div className="text-right">
            <p className="text-3xl font-bold text-blue-600">
              {goal.completion_percentage}%
            </p>
            <p className="text-sm text-gray-600">Complete</p>
          </div>
        </div>

        <div className="w-full bg-gray-200 rounded-full h-2">
          <div
            className="bg-blue-600 h-2 rounded-full transition-all"
            style={{ width: `${goal.completion_percentage}%` }}
          ></div>
        </div>
      </div>

      {/* Roadmap */}
      <div className="bg-white rounded-lg border p-6">
        <h2 className="text-2xl font-bold mb-4">📋 Your Roadmap</h2>

        {goal.roadmap_json?.phases?.map((phase: any) => (
          <div key={phase.phase_number} className="mb-6 pb-6 border-b last:border-b-0">
            <h3 className="text-lg font-semibold mb-2">
              Phase {phase.phase_number}: {phase.phase_name}
            </h3>
            <p className="text-gray-600 text-sm mb-3">{phase.description}</p>
            <ul className="list-disc list-inside space-y-1">
              {phase.key_milestones.map((milestone: string, i: number) => (
                <li key={i} className="text-sm text-gray-700">
                  {milestone}
                </li>
              ))}
            </ul>
          </div>
        ))}
      </div>

      {/* Milestones */}
      <div className="bg-white rounded-lg border p-6">
        <h2 className="text-2xl font-bold mb-4">🎯 Weekly Milestones</h2>
        <div className="space-y-3">
          {milestones?.map((milestone: any) => (
            <div
              key={milestone.id}
              className="flex items-center p-3 border rounded-lg hover:bg-gray-50"
            >
              <input
                type="checkbox"
                checked={milestone.is_completed}
                onChange={() => {
                  // Handle milestone completion
                }}
                className="mr-3 w-5 h-5"
              />
              <div className="flex-1">
                <p className="font-medium">
                  Week {milestone.milestone_number}: {milestone.milestone_title}
                </p>
                <p className="text-sm text-gray-600">
                  {milestone.milestone_description}
                </p>
              </div>
              {milestone.is_completed && <span className="text-green-600">✓</span>}
            </div>
          ))}
        </div>
      </div>

      {/* Today's Tasks */}
      <div className="bg-white rounded-lg border p-6">
        <h2 className="text-2xl font-bold mb-4">📅 Today's Tasks</h2>
        <div className="space-y-3">
          {tasks?.map((task: any) => (
            <div
              key={task.id}
              className="flex items-center p-3 border rounded-lg hover:bg-gray-50"
            >
              <input
                type="checkbox"
                onChange={async (e) => {
                  if (e.target.checked) {
                    await fetch(`/api/tasks/${task.id}/complete`, {
                      method: 'POST',
                    });
                  }
                }}
                className="mr-3 w-5 h-5"
              />
              <div className="flex-1">
                <p className="font-medium">{task.task_title}</p>
                <p className="text-sm text-gray-600">
                  {task.estimated_duration_minutes} minutes
                </p>
              </div>
            </div>
          ))}
        </div>
      </div>

      {/* Reflections */}
      {reflections && reflections.length > 0 && (
        <div className="bg-white rounded-lg border p-6">
          <h2 className="text-2xl font-bold mb-4">💭 Recent Reflections</h2>
          <div className="space-y-4">
            {reflections.slice(0, 3).map((reflection: any) => (
              <div key={reflection.id} className="p-4 border rounded-lg bg-gray-50">
                <p className="font-semibold mb-2">
                  Week {reflection.reflection_week} Reflection
                </p>
                <p className="text-sm mb-2">
                  <strong>✓ What went well:</strong> {reflection.what_went_well}
                </p>
                <p className="text-sm mb-2">
                  <strong>⚠️ What was hard:</strong> {reflection.what_was_hard}
                </p>
                <p className="text-sm">
                  <strong>🔧 To improve:</strong> {reflection.what_to_improve}
                </p>
              </div>
            ))}
          </div>
        </div>
      )}
    </div>
  );
}
```

---

## DAILY CHECK-IN INTEGRATION

### Daily Check-in With Goal Context

**File: `components/checkin/DailyCheckIn.tsx`**

```typescript
import { useState } from 'react';

export function DailyCheckIn({ activeGoals }: { activeGoals: any[] }) {
  const [answers, setAnswers] = useState({
    todays_main_focus: '',
    yesterdays_accomplishment: '',
    yesterdays_challenges: '',
    todays_priorities: '',
  });
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsSubmitting(true);

    try {
      const response = await fetch('/api/checkins/create', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          ...answers,
          check_in_date: new Date().toISOString().split('T')[0],
        }),
      });

      if (!response.ok) throw new Error('Failed to save check-in');

      // Show success
      alert('Check-in saved! Keep up the great work 🔥');
      setAnswers({
        todays_main_focus: '',
        yesterdays_accomplishment: '',
        yesterdays_challenges: '',
        todays_priorities: '',
      });
    } catch (error: any) {
      alert(error.message);
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-6 max-w-2xl mx-auto">
      <h2 className="text-2xl font-bold">Daily Check-in</h2>

      <div>
        <label className="block text-sm font-medium mb-2">
          What's your main focus today?
        </label>
        <select
          value={answers.todays_main_focus}
          onChange={(e) =>
            setAnswers((prev) => ({
              ...prev,
              todays_main_focus: e.target.value,
            }))
          }
          className="w-full px-4 py-2 border rounded-lg"
          required
        >
          <option value="">Select a goal...</option>
          {activeGoals.map((goal: any) => (
            <option key={goal.id} value={goal.id}>
              {goal.goal_title}
            </option>
          ))}
        </select>
      </div>

      <div>
        <label className="block text-sm font-medium mb-2">
          What did you accomplish yesterday?
        </label>
        <textarea
          value={answers.yesterdays_accomplishment}
          onChange={(e) =>
            setAnswers((prev) => ({
              ...prev,
              yesterdays_accomplishment: e.target.value,
            }))
          }
          placeholder="Tell me what you got done..."
          className="w-full px-4 py-2 border rounded-lg h-24"
          required
        />
      </div>

      <div>
        <label className="block text-sm font-medium mb-2">
          What challenges did you face?
        </label>
        <textarea
          value={answers.yesterdays_challenges}
          onChange={(e) =>
            setAnswers((prev) => ({
              ...prev,
              yesterdays_challenges: e.target.value,
            }))
          }
          placeholder="Any blockers or difficulties?"
          className="w-full px-4 py-2 border rounded-lg h-24"
        />
      </div>

      <div>
        <label className="block text-sm font-medium mb-2">
          What are your priorities for today?
        </label>
        <textarea
          value={answers.todays_priorities}
          onChange={(e) =>
            setAnswers((prev) => ({
              ...prev,
              todays_priorities: e.target.value,
            }))
          }
          placeholder="Your top 3 priorities..."
          className="w-full px-4 py-2 border rounded-lg h-24"
          required
        />
      </div>

      <button
        type="submit"
        disabled={isSubmitting}
        className="w-full bg-blue-600 text-white py-3 rounded-lg font-medium hover:bg-blue-700 disabled:opacity-50"
      >
        {isSubmitting ? 'Saving...' : 'Submit Check-in'}
      </button>
    </form>
  );
}
```

---

## ANALYTICS & INSIGHTS

### Goal Progress Analytics

**File: `pages/api/goals/[goalId]/analytics.ts`**

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';
import { withAuth } from '@/middleware/auth';
import { goalService } from '@/lib/goalService';

async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'GET') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const userId = (req as any).userId;
    const { goalId } = req.query;

    // Verify ownership
    const { data: goal } = await supabaseAdmin
      .from('goals')
      .select('user_id')
      .eq('id', goalId)
      .single();

    if (goal.user_id !== userId) {
      return res.status(403).json({ error: 'Unauthorized' });
    }

    const analytics = await goalService.getGoalAnalytics(goalId as string);

    return res.status(200).json({
      success: true,
      analytics,
    });
  } catch (error: any) {
    console.error('Analytics error:', error);
    return res.status(500).json({ error: 'Failed to get analytics' });
  }
}

export default withAuth(handler);
```

---

## TESTING CHECKLIST

```
Unit Tests:
[ ] generateRoadmap() returns valid structure
[ ] createGoal() saves to database
[ ] completeTask() updates completion
[ ] completeMilestone() updates goal percentage
[ ] addReflection() stores reflection
[ ] updateStreak() increments/resets correctly

Integration Tests:
[ ] Create goal → Get goal → Complete task → Verify
[ ] Create goal → Complete all tasks → Verify streak update
[ ] Create goal → Add reflection → Verify saved
[ ] Free tier limit enforced (3 goals max)

UI Tests:
[ ] CreateGoalForm validates input
[ ] GoalDetails loads and displays correctly
[ ] Tasks can be marked complete
[ ] Reflections can be added
[ ] Progress bar updates
```

---

## DEPLOYMENT CHECKLIST

```
Before Launch:
[ ] All endpoints tested with real users
[ ] AI roadmap generation tested extensively
[ ] Roadmap quality verified by coaches
[ ] Task completion logic working
[ ] Streak system working correctly
[ ] Reflection system working
[ ] Analytics accurate
[ ] Performance tested with 1000+ goals

Monitoring:
[ ] Track roadmap generation times
[ ] Monitor task completion rates
[ ] Track user goal completion rates
[ ] Monitor reflection submission rates
[ ] Alert if <50% of goals completed
```

---

## NEXT STEPS

### Immediate (This Week)
1. [ ] Implement `goalService.ts` with all methods
2. [ ] Create API endpoints for goal management
3. [ ] Build goal creation form component
4. [ ] Build goal details component
5. [ ] Test goal creation to completion flow

### Following Week
1. [ ] Refine AI roadmap generation based on feedback
2. [ ] Optimize roadmap quality
3. [ ] Add analytics dashboard
4. [ ] Performance optimization
5. [ ] Beta testing with 20 users

---

**Document Owner:** Product Lead + Backend Engineer  
**Last Updated:** July 2026  
**Next Review:** August 2026  
**Status:** Ready for Implementation
