# LifeOS AI

AI-powered personal operating system for life transformation.

## Vision

LifeOS AI is a personal AI companion that understands a user's goals, personality, progress, habits, and challenges. It acts like a personal mentor, strategic planner, accountability partner, learning coach, and productivity assistant.

## Architecture

### Monorepo Structure

```
lifeos-ai/
├── apps/
│   ├── web/                 # Next.js frontend
│   └── api/                 # Node.js backend API
├── packages/
│   ├── database/            # Prisma schemas & migrations
│   ├── shared/              # Shared types & utilities
│   ├── config/              # Configuration management
│   └── email/               # Email service
├── docs/                    # Documentation
└── scripts/                 # Build & deployment scripts
```

## Tech Stack

- **Frontend**: Next.js 14, TypeScript, Tailwind CSS, React
- **Backend**: Node.js, Express.js, TypeScript
- **Database**: PostgreSQL, Prisma ORM
- **Auth**: NextAuth.js
- **AI**: OpenAI API
- **Deployment**: Vercel
- **Monitoring**: Sentry

## Getting Started

```bash
# Install dependencies
npm install

# Setup environment
cp .env.example .env.local

# Setup database
npm run db:migrate

# Run development
npm run dev
```

## Development Phases

- **Phase 1**: User System (Auth, Profile)
- **Phase 2**: AI Mentor System
- **Phase 3**: Goal Management
- **Phase 4**: AI Roadmap Generator
- **Phase 5**: Habit System
- **Phase 6**: Productivity Dashboard
- **Phase 7**: Knowledge Vault
- **Phase 8**: AI Memory System

## License

MIT
