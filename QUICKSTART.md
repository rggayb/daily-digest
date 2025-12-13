# Quick Start Guide

## 🚀 Getting Started

### 1. Install Dependencies

```bash
npm install --legacy-peer-deps
```

### 2. Setup Environment Variables

Copy `.env.example` to `.env.local` and fill in the values:

```bash
cp .env.example .env.local
```

**Required variables:**

```env
# Database (use Vercel Postgres or Supabase)
DATABASE_URL="postgresql://..."

# NextAuth (generate with: openssl rand -base64 32)
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="your-secret-here"

# Google OAuth (get from Google Cloud Console)
GOOGLE_CLIENT_ID="your-client-id"
GOOGLE_CLIENT_SECRET="your-client-secret"

# Cron Secret (generate with: openssl rand -base64 32)
CRON_SECRET="your-cron-secret"
```

**Pre-configured (no change needed):**
- `TWITTER_API_KEY` - Already set
- `OPENAI_API_KEY` - Already set

### 3. Setup Google OAuth

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project
3. Enable **Gmail API** and **Google+ API**
4. Go to **Credentials** → **Create OAuth 2.0 Client ID**
5. Add redirect URIs:
   - `http://localhost:3000/api/auth/callback/google`
   - `https://your-domain.vercel.app/api/auth/callback/google`
6. Copy Client ID and Secret to `.env.local`

### 4. Setup Database

#### Option A: Vercel Postgres (Recommended)

```bash
npm i -g vercel
vercel link
vercel postgres create
vercel env pull .env.local
```

#### Option B: Supabase

1. Create project at [supabase.com](https://supabase.com/)
2. Copy connection string to `.env.local`

### 5. Run Migrations

```bash
npx prisma migrate dev --name init
```

### 6. Start Development Server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000)

## 📦 Deploy to Vercel

### 1. Push to GitHub

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin your-repo-url
git push -u origin main
```

### 2. Import to Vercel

1. Go to [vercel.com](https://vercel.com/)
2. Import your repository
3. Add all environment variables from `.env.local`
4. Deploy

### 3. Setup Production Database

```bash
vercel env pull
npx prisma migrate deploy
```

### 4. Verify Cron Job

The cron job will automatically run every hour. Check Vercel dashboard → Cron Logs.

## 🧪 Testing

1. Sign in with Google
2. Add Twitter usernames (e.g., `elonmusk, sama, karpathy`)
3. Set schedule time and email
4. Click **"Test Run Now"**
5. Check your email in 2-3 minutes

## 📝 Common Issues

### "No Google access token"
- Make sure you've granted Gmail permissions during OAuth
- Try signing out and signing in again

### Cron job not running
- Verify `CRON_SECRET` is set in Vercel
- Check `vercel.json` exists in root
- View logs in Vercel dashboard

### Database connection error
- Verify `DATABASE_URL` format
- Run `npx prisma generate`
- Run migrations with `npx prisma migrate deploy`

## 📚 More Help

See [README.md](./README.md) for full documentation.


