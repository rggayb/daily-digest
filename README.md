# Daily Digest - Twitter to Email

Automated daily digest application that fetches tweets from specified Twitter accounts, filters them using AI, and sends a beautifully formatted digest to your Gmail.

## Features

- ✅ **Google OAuth Authentication** - Secure login with Google
- ✅ **Twitter Integration** - Fetch tweets by username (no Twitter API key needed for users)
- ✅ **AI-Powered Filtering** - Two-stage LLM processing to filter noise and format digest
- ✅ **LangSmith Integration** - LLM observability, monitoring, and debugging
- ✅ **Gmail Delivery** - Send digests via Gmail OAuth
- ✅ **Customizable Settings** - Configure Twitter accounts, schedule time, and time window
- ✅ **Daily Automated Execution** - Vercel Cron Jobs
- ✅ **Execution History** - View logs of past digests

## Tech Stack

- **Frontend & Backend**: Next.js 14 (App Router) + TypeScript
- **Database**: PostgreSQL (Vercel Postgres or Supabase)
- **Authentication**: NextAuth.js v5 with Google OAuth
- **Styling**: TailwindCSS + shadcn/ui
- **AI**: OpenAI (GPT-4o-mini for filtering, GPT-4o for formatting)
- **LLM Monitoring**: LangSmith (tracing, evaluation, debugging)
- **Email**: Gmail API via OAuth
- **Deployment**: Vercel (with Cron Jobs)

## Prerequisites

1. **Google Cloud Console** account for OAuth setup
2. **PostgreSQL** database (Vercel Postgres or Supabase)
3. **OpenAI API** key
4. **Twitter API** key (shared, already configured)
5. **LangSmith API** key (optional, for LLM monitoring)

## Setup Instructions

### 1. Clone and Install Dependencies

```bash
cd daily-digest
npm install
```

### 2. Setup Google OAuth

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select existing one
3. Enable **Gmail API** and **Google+ API**
4. Go to **Credentials** → **Create Credentials** → **OAuth 2.0 Client ID**
5. Configure OAuth consent screen
6. Add Authorized redirect URIs:
   - `http://localhost:3000/api/auth/callback/google` (for dev)
   - `https://your-domain.vercel.app/api/auth/callback/google` (for production)
7. Copy **Client ID** and **Client Secret**

### 3. Setup Database

#### Option A: Vercel Postgres (Recommended for Vercel deployment)

```bash
# Install Vercel CLI
npm i -g vercel

# Link to your Vercel project
vercel link

# Create Postgres database
vercel postgres create

# Get connection string
vercel env pull .env.local
```

#### Option B: Supabase

1. Go to [Supabase](https://supabase.com/)
2. Create new project
3. Copy connection string from Settings → Database

### 4. Configure Environment Variables

Copy `.env.example` to `.env.local`:

```bash
cp .env.example .env.local
```

Update the following variables in `.env.local`:

```env
# Database
DATABASE_URL="your-postgres-connection-string"

# NextAuth
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="run: openssl rand -base64 32"

# Google OAuth
GOOGLE_CLIENT_ID="your-google-client-id"
GOOGLE_CLIENT_SECRET="your-google-client-secret"

# Cron Secret (for production)
CRON_SECRET="run: openssl rand -base64 32"

# LangSmith (optional - for LLM monitoring)
LANGCHAIN_TRACING_V2="true"
LANGCHAIN_API_KEY="your-langsmith-api-key"
LANGCHAIN_PROJECT="daily-digest-prod"
```

### 5. Setup Database Schema

```bash
npx prisma migrate dev --name init
npx prisma generate
```

### 6. Run Development Server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

## Deployment to Vercel

### 1. Push to GitHub

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin your-repo-url
git push -u origin main
```

### 2. Deploy to Vercel

1. Go to [Vercel](https://vercel.com/)
2. Import your GitHub repository
3. Add environment variables (same as `.env.local`)
4. Deploy

### 3. Setup Vercel Postgres

```bash
vercel env pull
```

Then run migrations on production:

```bash
npx prisma migrate deploy
```

### 4. Setup Cron Job

The cron job is already configured in `vercel.json`. It will run every hour and check for digests scheduled at that hour.

Make sure to add `CRON_SECRET` to your Vercel environment variables.

## How It Works

### User Flow

1. User signs in with Google (OAuth)
2. User configures digest settings:
   - Twitter usernames to monitor (comma-separated)
   - Schedule time (0-23 hour)
   - Time window (how far back to fetch tweets)
   - Recipient email
3. User can test the digest immediately or wait for daily scheduled execution

### Automated Flow (Cron Job)

1. **Trigger**: Vercel Cron runs every hour (`/api/cron/digest`)
2. **Fetch**: Get tweets from all configured Twitter usernames
3. **Filter Time**: Keep only tweets within the time window (e.g., last 24 hours)
4. **Filter Noise (LLM #1)**: Use GPT-4o-mini to filter out low-value tweets
5. **Format Digest (LLM #2)**: Use GPT-4o to create a clean HTML digest
6. **Send Email**: Send formatted digest via Gmail OAuth
7. **Log**: Save execution log to database

## API Routes

### Public Routes

- `GET /` - Landing page
- `GET /api/auth/signin` - Sign in page
- `GET /api/auth/signout` - Sign out

### Protected Routes

- `GET /dashboard` - User dashboard
- `PUT /api/digest` - Update digest settings
- `POST /api/digest/run` - Manually trigger digest execution

### Cron Route

- `GET /api/cron/digest` - Automated cron job (secured with CRON_SECRET)

## Project Structure

```
daily-digest/
├── app/
│   ├── api/
│   │   ├── auth/[...nextauth]/route.ts  # NextAuth handler
│   │   ├── cron/digest/route.ts         # Cron job endpoint
│   │   └── digest/
│   │       ├── route.ts                 # Update digest
│   │       └── run/route.ts             # Manual trigger
│   ├── dashboard/page.tsx               # Dashboard page
│   ├── globals.css                      # Global styles
│   ├── layout.tsx                       # Root layout
│   └── page.tsx                         # Landing page
├── components/
│   ├── auth/signin-button.tsx           # Sign in button
│   ├── dashboard/dashboard-content.tsx  # Dashboard UI
│   └── ui/                              # shadcn/ui components
├── lib/
│   ├── digest-processor.ts              # Main digest logic
│   ├── gmail-service.ts                 # Gmail API integration
│   ├── llm-service.ts                   # OpenAI integration
│   ├── prisma.ts                        # Prisma client
│   ├── twitter-service.ts               # Twitter API integration
│   └── utils.ts                         # Utility functions
├── prisma/
│   └── schema.prisma                    # Database schema
├── auth.ts                              # NextAuth configuration
├── package.json
├── tailwind.config.ts
├── tsconfig.json
└── vercel.json                          # Vercel cron configuration
```

## Troubleshooting

### Gmail Not Sending

- Make sure you've enabled **Gmail API** in Google Cloud Console
- Check that the user has granted **gmail.send** scope during OAuth
- Verify that `access_token` is not expired (NextAuth should refresh automatically)

### Cron Job Not Running

- Verify `CRON_SECRET` is set in Vercel environment variables
- Check Vercel Cron logs in Vercel dashboard
- Ensure `vercel.json` is in the root directory

### Database Connection Issues

- Verify `DATABASE_URL` is correct
- Run `npx prisma generate` after schema changes
- Run `npx prisma migrate deploy` on production

### OpenAI Rate Limits

- Monitor your OpenAI usage in OpenAI dashboard
- Consider reducing the number of tweets processed
- Add retry logic if needed

### LangSmith Not Showing Traces

- Check `LANGCHAIN_TRACING_V2=true` is set
- Verify API key is correct
- Wait 10-30 seconds for traces to appear
- See `LANGSMITH_SETUP.md` for detailed troubleshooting

## Future Enhancements

- [ ] Support for multiple digests per user
- [ ] Custom AI prompts for formatting
- [ ] Slack integration (alternative to email)
- [ ] RSS feed output
- [ ] Tweet sentiment analysis
- [ ] Weekly/monthly digest options
- [ ] Export digest as PDF
- [ ] Analytics dashboard

## License

MIT

## Support

For issues or questions, please create an issue in the GitHub repository.


