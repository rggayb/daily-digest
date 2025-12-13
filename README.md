# 📧 Daily Digest

> Turn Twitter noise into actionable insights. Get AI-curated daily digests from your favorite Twitter accounts, delivered straight to your inbox.

## ✨ What is Daily Digest?

Daily Digest is an intelligent automation tool that monitors Twitter accounts you care about, filters out the noise using AI, and sends you a beautifully formatted email summary on your schedule.

**Perfect for:**
- 📈 Staying updated on industry trends without endless scrolling
- 🎯 Following thought leaders efficiently
- 🚀 Getting actionable insights from multiple sources in one place
- ⏰ Saving hours of social media browsing time

## 🎯 Key Features

- **Smart AI Filtering** - Two-stage AI processing removes low-value content and highlights what matters
- **Automated Delivery** - Set your schedule and receive digests automatically via Gmail
- **Flexible Monitoring** - Track multiple Twitter accounts with customizable time windows
- **Beautiful Format** - Clean, readable HTML emails with direct links to source tweets
- **One-Click Testing** - Preview your digest before scheduling
- **Complete Privacy** - Run on your own infrastructure with your own API keys

## 🚀 Quick Start

### Prerequisites

You'll need:
- A Google account (for authentication & Gmail delivery)
- An OpenAI API key ([get one here](https://platform.openai.com/api-keys))
- A PostgreSQL database (free options: [Vercel Postgres](https://vercel.com/docs/storage/vercel-postgres) or [Supabase](https://supabase.com/))
- *(Optional)* A LangSmith account for AI monitoring ([free tier available](https://smith.langchain.com/))

### Installation

1. **Clone the repository**

```bash
git clone https://github.com/yourusername/daily-digest.git
cd daily-digest
npm install
```

2. **Set up Google OAuth**

   - Visit [Google Cloud Console](https://console.cloud.google.com/)
   - Create a new project
   - Enable the **Gmail API**
   - Create OAuth 2.0 credentials
   - Add authorized redirect URIs:
     - `http://localhost:3000/api/auth/callback/google` (development)
     - `https://your-domain.vercel.app/api/auth/callback/google` (production)
   - Save your Client ID and Client Secret

3. **Configure environment variables**

```bash
cp .env.example .env.local
```

Edit `.env.local` with your credentials:

```env
# Database
DATABASE_URL="your-postgresql-connection-string"

# Authentication
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="generate-with: openssl rand -base64 32"

# Google OAuth
GOOGLE_CLIENT_ID="your-google-client-id"
GOOGLE_CLIENT_SECRET="your-google-client-secret"

# OpenAI
OPENAI_API_KEY="your-openai-api-key"

# Twitter API (shared key - already configured in code)
# No action needed

# Cron security
CRON_SECRET="generate-with: openssl rand -base64 32"

# Optional: AI Monitoring
LANGCHAIN_TRACING_V2="true"
LANGCHAIN_API_KEY="your-langsmith-api-key"
LANGCHAIN_PROJECT="daily-digest"
```

4. **Initialize the database**

```bash
npx prisma migrate dev --name init
npx prisma generate
```

5. **Start the development server**

```bash
npm run dev
```

Visit [http://localhost:3000](http://localhost:3000) and sign in with Google!

## 🌐 Deploy to Production

### Option 1: Vercel (Recommended)

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new)

1. Push your code to GitHub
2. Import your repository in [Vercel](https://vercel.com/)
3. Add all environment variables from `.env.local`
4. Deploy!

The cron job is pre-configured in `vercel.json` and will automatically run hourly.

### Option 2: Other Platforms

Daily Digest works on any platform that supports:
- Next.js 14+
- Node.js 18+
- Cron jobs or scheduled tasks

## 📖 How to Use

1. **Sign In** - Use your Google account to authenticate
2. **Configure Settings** in the dashboard:
   - **Twitter Accounts**: Enter usernames to monitor (comma-separated, no @ symbol)
   - **Schedule**: Choose the hour to receive your digest (0-23)
   - **Time Window**: How far back to look for tweets (e.g., 24 hours)
   - **Email**: Where to send the digest (defaults to your Google email)
3. **Test It** - Click "Run Digest Now" to preview
4. **Relax** - Your digest will arrive automatically on schedule!

## 🛠 Tech Stack

| Category | Technology |
|----------|-----------|
| **Framework** | Next.js 14 (App Router) + TypeScript |
| **Database** | PostgreSQL with Prisma ORM |
| **Authentication** | NextAuth.js v5 + Google OAuth |
| **AI** | OpenAI (GPT-4o-mini & GPT-4o) |
| **Email** | Gmail API |
| **Styling** | Tailwind CSS + shadcn/ui |
| **Deployment** | Vercel (with Cron Jobs) |
| **Monitoring** | LangSmith (optional) |

## 📁 Project Structure

```
daily-digest/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   ├── auth/         # Authentication endpoints
│   │   ├── cron/         # Automated cron job
│   │   └── digest/       # Digest management
│   ├── dashboard/        # User dashboard
│   └── page.tsx          # Landing page
├── components/            # React components
│   ├── auth/            # Authentication UI
│   ├── dashboard/       # Dashboard UI
│   └── ui/              # Reusable UI components
├── lib/                  # Core business logic
│   ├── digest-processor.ts   # Main orchestrator
│   ├── gmail-service.ts      # Email delivery
│   ├── llm-service.ts        # AI processing
│   ├── twitter-service.ts    # Twitter integration
│   └── prisma.ts             # Database client
├── prisma/
│   └── schema.prisma     # Database schema
└── vercel.json          # Cron configuration
```

## 🔧 Configuration

### Email Scopes

During first sign-in, users must grant these permissions:
- `openid`, `email`, `profile` - Basic authentication
- `https://www.googleapis.com/auth/gmail.send` - Send emails on your behalf

### AI Processing

The digest uses a two-stage AI pipeline:

1. **Noise Filter (GPT-4o-mini)** - Fast, cost-effective filtering of irrelevant tweets
2. **Digest Formatter (GPT-4o)** - Creates polished, readable summaries with key insights

### Cron Schedule

By default, the cron runs **every hour** (configured in `vercel.json`). The system checks which users have digests scheduled for that hour and processes them automatically.

## ❓ FAQ

**Q: How much does it cost to run?**  
A: Main costs are OpenAI API usage (~$0.01-0.10 per digest depending on tweet volume) and database hosting (free tier available on Vercel/Supabase).

**Q: Can I monitor non-public Twitter accounts?**  
A: No, only public accounts are accessible.

**Q: What happens if Twitter API is down?**  
A: The system will log the error and retry on the next scheduled run.

**Q: Can I customize the AI prompts?**  
A: Yes! Edit `lib/llm-service.ts` to modify filtering criteria and formatting.

**Q: Is my data private?**  
A: Yes. Everything runs on your infrastructure. Your tweets and emails are never stored permanently.

## 🐛 Troubleshooting

### Gmail not sending emails

- Ensure Gmail API is enabled in Google Cloud Console
- Verify the `gmail.send` scope was granted during OAuth
- Check Vercel logs for error messages

### Cron job not running

- Confirm `CRON_SECRET` is set in production environment variables
- Check Vercel Cron logs in your dashboard
- Verify `vercel.json` is in the repository root

### Database connection errors

- Verify `DATABASE_URL` format is correct
- Run `npx prisma generate` after any schema changes
- For production: `npx prisma migrate deploy`

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

MIT License - feel free to use this project for personal or commercial purposes.

## 🙏 Acknowledgments

Built with:
- [Next.js](https://nextjs.org/)
- [OpenAI](https://openai.com/)
- [shadcn/ui](https://ui.shadcn.com/)
- [Prisma](https://www.prisma.io/)
- [NextAuth.js](https://next-auth.js.org/)

---

Made with ☕ by developers who value their time
