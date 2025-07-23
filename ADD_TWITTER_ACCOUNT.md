# How to Add a Twitter Account to Twitter Manager

This guide explains the simple process of adding a Twitter account to be managed by the Twitter Manager application.

## Prerequisites

1. **Twitter Developer Account**: You need access to the Twitter Developer Portal
2. **Flask Server Running**: Start the server with `python app.py`
3. **API Key**: `2043adb52a7468621a9245c94d702e4bed5866b0ec52772f203286f823a50bbb`

## Step-by-Step Process

### Step 1: Get the Authorization URL

**What it does**: Generates a Twitter OAuth 2.0 login URL with PKCE security.

**How to do it**:
```bash
curl -X GET http://localhost:5555/api/v1/auth/twitter \
  -H "X-API-Key: 2043adb52a7468621a9245c94d702e4bed5866b0ec52772f203286f823a50bbb"
```

**You'll receive**:
```json
{
    "auth_url": "https://twitter.com/i/oauth2/authorize?...",
    "state": "secure_random_state_value"
}
```

### Step 2: Authorize the Account

**What it does**: Logs into Twitter and grants permission to your app.

**How to do it**:
1. Copy the `auth_url` from Step 1
2. Open it in your web browser
3. Log in to the Twitter account you want to add
4. Review the permissions:
   - Read tweets
   - Post tweets
   - Access profile information
   - Offline access
5. Click "Authorize app"

**What happens**: Twitter redirects to:
```
http://localhost:5555/auth/callback?code=AUTH_CODE&state=STATE_VALUE
```

You'll see a **success page** with:
- Confirmation message
- Account ID and username
- Instructions for posting tweets

### Step 3: Verify Account Added

**What it does**: Confirms the account is saved and ready to use.

**How to do it**:
```bash
curl -X GET http://localhost:5555/api/v1/accounts \
  -H "X-API-Key: 2043adb52a7468621a9245c94d702e4bed5866b0ec52772f203286f823a50bbb"
```

**You'll see**:
```json
{
    "accounts": [
        {
            "id": 2,
            "username": "YourTwitterHandle",
            "status": "active",
            "created_at": "2025-07-22T12:31:51.819225"
        }
    ],
    "total": 1
}
```

## That's It! 🎉

Your Twitter account is now added and ready to use. You can now post tweets!

## Visual Flow Diagram

```
1. Request Auth URL          2. User Authorizes          3. Account Saved
┌─────────────────┐         ┌─────────────────┐        ┌─────────────────┐
│   Your App      │         │    Twitter      │        │   Success Page  │
│                 │         │                 │        │                 │
│ GET /auth/twitter ──────> │ OAuth 2.0 Page  │        │ ✅ Account Added │
│                 │         │                 │        │                 │
│ <- Returns URL  │         │ User Logs In    │        │ Account ID: 2   │
└─────────────────┘         │                 │        │ Username: @user │
                            │ Grants Access   │        │                 │
                            │                 │        │ Ready to Post!  │
                            │ Redirects to    │        └─────────────────┘
                            │ /auth/callback  │
                            └─────────────────┘
```

## Using the Account

### Create a Tweet (Pending Status)
```bash
curl -X POST http://localhost:5555/api/v1/tweet \
  -H "X-API-Key: 2043adb52a7468621a9245c94d702e4bed5866b0ec52772f203286f823a50bbb" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Hello from Twitter Manager!",
    "account_id": 2
  }'
```

Response:
```json
{
    "message": "Tweet created successfully",
    "tweet_id": 6
}
```

### Post the Tweet to Twitter
```bash
curl -X POST http://localhost:5555/api/v1/tweet/post/6 \
  -H "X-API-Key: 2043adb52a7468621a9245c94d702e4bed5866b0ec52772f203286f823a50bbb"
```

Response:
```json
{
    "message": "Tweet posted successfully",
    "tweet_id": 6,
    "twitter_id": "1947673926596731295"
}
```

## Common Issues & Solutions

### "Something went wrong" on Twitter
- **Check OAuth 2.0**: Ensure OAuth 2.0 is enabled in Twitter Developer Portal
- **Verify Callback URL**: Must be `http://localhost:5555/auth/callback`
- **Check Permissions**: App needs Read and Write permissions

### "Invalid state" Error
- The authorization state expired or was already used
- Simply start over from Step 1

### 401 Unauthorized When Posting
- The access token has expired
- Re-run the authorization process to refresh tokens
- The system will update the existing account

### OAuth 1.0a Not Supported
- The current implementation only supports OAuth 2.0
- Re-authorize any OAuth 1.0a accounts using the steps above

## Need to Add Multiple Accounts?

Simply repeat the process for each Twitter account you want to manage. Each account will:
- Get its own unique ID
- Be stored with encrypted tokens
- Be available for posting tweets

## Important Security Notes

1. **Tokens are encrypted** using Fernet encryption before storage
2. **Never share your access tokens** or API keys
3. **OAuth 2.0 with PKCE** provides enhanced security
4. **Refresh tokens** are stored for future token renewal

## Twitter App Configuration

Your Twitter app must have:
- **OAuth 2.0 enabled** (not OAuth 1.0a)
- **Callback URL**: `http://localhost:5555/auth/callback`
- **Required Scopes**: tweet.read, tweet.write, users.read, offline.access
- **App Type**: Web App, Automated App, or Bot

For Twitter app setup, visit: https://developer.twitter.com/en/portal/dashboard