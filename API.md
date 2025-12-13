# API Documentation

## Authentication

All protected routes require authentication via NextAuth session cookie.

---

## Public Routes

### `GET /`
Landing page with sign-in option.

**Response**: HTML page

---

### `GET /api/auth/signin`
Sign in page (redirects to Google OAuth).

**Response**: Redirect to Google OAuth

---

### `GET /api/auth/signout`
Sign out current user.

**Response**: Redirect to landing page

---

## Protected Routes

### `GET /dashboard`
User dashboard for managing digest settings.

**Authentication**: Required

**Response**: HTML page with:
- Digest configuration form
- Recent execution logs
- Quick info

---

### `PUT /api/digest`
Update digest configuration.

**Authentication**: Required

**Request Body**:
```json
{
  "name": "My Daily Digest",
  "twitterUsernames": ["elonmusk", "sama", "karpathy"],
  "scheduleHour": 6,
  "timeWindowHours": 24,
  "recipientEmail": "user@example.com",
  "isActive": true
}
```

**Response**:
```json
{
  "success": true
}
```

**Error Responses**:
- `401 Unauthorized` - Not authenticated
- `404 Not Found` - Digest not found
- `500 Internal Server Error` - Server error

---

### `POST /api/digest/run`
Manually trigger digest execution (test run).

**Authentication**: Required

**Request Body**: None

**Response**:
```json
{
  "success": true,
  "message": "Digest is being processed"
}
```

**Process**:
1. Fetches tweets from configured usernames
2. Filters by time window
3. Filters noise with AI (GPT-4o-mini)
4. Formats digest with AI (GPT-4o)
5. Sends email via Gmail OAuth
6. Logs execution to database

**Error Responses**:
- `401 Unauthorized` - Not authenticated
- `404 Not Found` - Digest not found
- `400 Bad Request` - No Twitter usernames configured or Google account not connected
- `500 Internal Server Error` - Server error

---

## Cron Routes

### `GET /api/cron/digest`
Automated cron job endpoint (runs every hour).

**Authentication**: Bearer token with `CRON_SECRET`

**Headers**:
```
Authorization: Bearer YOUR_CRON_SECRET
```

**Response**:
```json
{
  "success": true,
  "processedCount": 5,
  "results": [
    {
      "digestId": "clx...",
      "status": "processing"
    }
  ]
}
```

**Process**:
1. Gets current hour (0-23)
2. Finds all active digests scheduled for this hour
3. Processes each digest asynchronously
4. Returns processing status

**Error Responses**:
- `401 Unauthorized` - Invalid or missing `CRON_SECRET`
- `500 Internal Server Error` - Server error

---

## Data Models

### User
```typescript
{
  id: string
  name: string | null
  email: string | null
  emailVerified: Date | null
  image: string | null
  createdAt: Date
  updatedAt: Date
}
```

### Digest
```typescript
{
  id: string
  userId: string
  name: string                    // Default: "My Daily Digest"
  twitterUsernames: string[]      // Array of Twitter usernames
  scheduleHour: number            // 0-23 (24-hour format)
  scheduleTimezone: string        // Default: "Asia/Jakarta"
  timeWindowHours: number         // Default: 24
  recipientEmail: string          // Email to send digest to
  isActive: boolean               // Default: true
  createdAt: Date
  updatedAt: Date
}
```

### DigestLog
```typescript
{
  id: string
  digestId: string
  status: string                  // "success" | "failed" | "processing"
  totalScanned: number            // Total tweets fetched
  totalSelected: number           // Tweets selected by AI
  digestContent: string | null    // HTML content of digest
  errorMessage: string | null     // Error message if failed
  executedAt: Date
}
```

---

## Error Handling

All API routes follow consistent error response format:

```json
{
  "error": "Error type",
  "message": "Detailed error message"
}
```

**Common HTTP Status Codes**:
- `200 OK` - Success
- `400 Bad Request` - Invalid request data
- `401 Unauthorized` - Authentication required or invalid
- `404 Not Found` - Resource not found
- `500 Internal Server Error` - Server error

---

## Rate Limits

### OpenAI API
- Depends on your OpenAI plan
- Monitor usage in OpenAI dashboard

### Twitter API
- Depends on twitterapi.io plan
- Currently using shared API key

### Gmail API
- **500 emails/day** for free Gmail accounts
- **2000 emails/day** for Google Workspace

---

## Webhooks

Currently not implemented. Future enhancement for real-time notifications.

---

## Example Integration

### Trigger digest from external service

```bash
curl -X POST https://your-domain.vercel.app/api/digest/run \
  -H "Cookie: next-auth.session-token=YOUR_SESSION_TOKEN"
```

### Monitor cron execution

```bash
curl https://your-domain.vercel.app/api/cron/digest \
  -H "Authorization: Bearer YOUR_CRON_SECRET"
```

---

## Security Best Practices

1. **Never expose**:
   - `NEXTAUTH_SECRET`
   - `CRON_SECRET`
   - `GOOGLE_CLIENT_SECRET`
   - `OPENAI_API_KEY`
   - `TWITTER_API_KEY`

2. **Always use HTTPS** in production

3. **Rotate secrets** periodically

4. **Monitor logs** for suspicious activity

5. **Use environment-specific secrets** (dev vs prod)


