# Prompt to Create Twitter Manager API

## Overview
Create a Flask-based REST API for managing multiple Twitter accounts and posting tweets. The application should support OAuth 2.0 authentication with PKCE, encrypted credential storage, batch operations, and comprehensive account/tweet management.

## Technical Requirements

### Core Stack
- Python 3.8+ with Flask web framework
- SQLite database with direct sqlite3 queries (no ORM)
- Port 5555 (configurable)
- Environment-based configuration (.env file)
- Fernet encryption for sensitive data

### Dependencies
```
Flask==3.0.0
python-dotenv==1.0.0
requests==2.31.0
cryptography==41.0.7
```

## Database Schema

Create four tables:

1. **api_key**
   - id (INTEGER PRIMARY KEY AUTOINCREMENT)
   - key_hash (TEXT UNIQUE NOT NULL) - SHA256 hash of API key
   - created_at (DATETIME DEFAULT CURRENT_TIMESTAMP)
   - is_active (BOOLEAN DEFAULT 1)

2. **twitter_account**
   - id (INTEGER PRIMARY KEY AUTOINCREMENT)
   - username (TEXT UNIQUE NOT NULL)
   - access_token (TEXT NOT NULL) - Encrypted
   - access_token_secret (TEXT) - For OAuth 1.0a detection
   - refresh_token (TEXT) - Encrypted
   - status (TEXT DEFAULT 'active') - active/suspended/failed
   - created_at (DATETIME DEFAULT CURRENT_TIMESTAMP)
   - updated_at (DATETIME)

3. **tweet**
   - id (INTEGER PRIMARY KEY AUTOINCREMENT)
   - twitter_account_id (INTEGER NOT NULL FOREIGN KEY)
   - content (TEXT NOT NULL)
   - status (TEXT DEFAULT 'pending') - pending/posted/failed
   - twitter_id (TEXT) - ID from Twitter after posting
   - created_at (DATETIME DEFAULT CURRENT_TIMESTAMP)
   - posted_at (DATETIME)

4. **oauth_state**
   - state (TEXT PRIMARY KEY)
   - code_verifier (TEXT NOT NULL)
   - created_at (DATETIME NOT NULL)

## Security Implementation

### API Authentication
- All endpoints except /health require X-API-Key header
- API key stored as SHA256 hash in database
- Support fallback to ?api_key query parameter

### OAuth 2.0 with PKCE
1. Generate cryptographically secure state and code_verifier
2. Create code_challenge using SHA256(code_verifier)
3. Store state and code_verifier temporarily
4. Required scopes: tweet.read, tweet.write, users.read, offline.access
5. Exchange authorization code for tokens
6. Encrypt tokens before storage using Fernet

### Encryption
- Generate Fernet key for encryption
- Encrypt access_token and refresh_token before database storage
- Decrypt tokens when needed for API calls
- Handle decryption failures gracefully

## API Endpoints

### Core Endpoints

1. **GET /api/v1/health** (No auth)
   - Returns: {status, timestamp, version}

2. **GET /api/v1/test** 
   - Validates API key
   - Returns: {status: "success", message}

### OAuth Flow

3. **GET /api/v1/auth/twitter**
   - Generates OAuth URL with PKCE parameters
   - Returns: {auth_url, state}

4. **GET /auth/callback**
   - Handles OAuth callback
   - Exchanges code for tokens
   - Creates/updates account
   - Returns: HTML success page for browser

### Account Management

5. **GET /api/v1/accounts**
   - Lists all accounts
   - Returns: {accounts: [{id, username, status, created_at}], total}

6. **GET /api/v1/accounts/{id}**
   - Get specific account details

7. **DELETE /api/v1/accounts/{id}**
   - Deletes account and all associated tweets
   - Returns: {message, deleted_tweets}

8. **POST /api/v1/accounts/cleanup**
   - Body: {statuses: ["failed", "suspended", "inactive"]}
   - Deletes accounts with specified statuses

### Tweet Management

9. **POST /api/v1/tweet**
   - Body: {text, account_id}
   - Creates tweet with "pending" status
   - Returns: {message, tweet_id}

10. **GET /api/v1/tweets**
    - Query params: status, account_id, page
    - Lists tweets (max 50)
    - Returns: {tweets: [{id, text, status, created_at, username}], total}

11. **POST /api/v1/tweet/post/{id}**
    - Posts specific pending tweet to Twitter
    - Updates status to "posted" or "failed"
    - Returns: {message, tweet_id, twitter_id}

12. **POST /api/v1/tweets/post-pending**
    - Posts all pending tweets
    - Returns: {total, posted, failed, details}

13. **DELETE /api/v1/tweets/{id}**
    - Deletes specific tweet

14. **POST /api/v1/tweets/cleanup**
    - Body: {statuses: [], days_old: N, account_id: N}
    - Deletes tweets by criteria

### Utilities

15. **GET /api/v1/stats**
    - Returns: {accounts: {total, active}, tweets: {total, pending, posted, failed}}

16. **GET /api/v1/mock-mode**
    - Check mock mode status

17. **POST /api/v1/mock-mode**
    - Body: {enabled: boolean}
    - Toggle mock mode for testing

## Implementation Details

### Startup Sequence
1. Create instance/ directory if not exists
2. Initialize SQLite database
3. Create all tables with proper schema
4. Insert API key from environment
5. Start Flask server on port 5555

### Twitter API Integration
- Use Twitter API v2 endpoint: POST https://api.twitter.com/2/tweets
- Headers: Authorization: Bearer {access_token}
- Body: {text: "tweet content"}
- Handle 401 errors (expired tokens)
- Store twitter_id from response

### Error Handling
- Return consistent JSON: {error: "message"}
- Use appropriate HTTP status codes
- Log errors for debugging
- Handle database connection errors gracefully

### Special Considerations
1. Reject OAuth 1.0a attempts (check for access_token_secret)
2. Auto-create instance directory on startup
3. Support both browser and API OAuth callbacks
4. Clean up expired OAuth states periodically
5. Handle Windows path issues in SQLite

## Environment Variables
```
API_KEY=<generated-api-key>
ENCRYPTION_KEY=<fernet-key>
TWITTER_CLIENT_ID=<from-twitter-dev>
TWITTER_CLIENT_SECRET=<from-twitter-dev>
TWITTER_CALLBACK_URL=http://localhost:5555/auth/callback
```

## File Structure
```
twitter-manager/
├── app.py                    # Main application
├── requirements.txt          # Dependencies
├── .env                      # Configuration (not committed)
├── .env.example              # Configuration template
├── .gitignore                # Git ignore rules
├── instance/                 # Auto-created
│   └── twitter_manager.db    # SQLite database
└── README.md                 # Documentation
```

## Additional Features to Implement

1. **Input Validation**
   - Validate tweet length (max 280 chars)
   - Validate account_id exists
   - Sanitize all inputs

2. **Rate Limiting**
   - Track API calls per account
   - Implement backoff strategies

3. **Logging**
   - Log all API requests
   - Log Twitter API interactions
   - Rotate logs daily

4. **Monitoring**
   - Track posting success rates
   - Monitor token expiration
   - Alert on repeated failures

## Testing Considerations
- Create comprehensive Postman collection
- Test OAuth flow end-to-end
- Verify encryption/decryption
- Test error scenarios
- Validate database constraints

## Production Recommendations
1. Use PostgreSQL instead of SQLite
2. Implement Redis for OAuth state storage
3. Add request rate limiting
4. Use production WSGI server (Gunicorn)
5. Implement proper logging and monitoring
6. Add backup and recovery procedures
7. Implement token refresh mechanism
8. Add webhook support for real-time updates

This application should be production-ready with proper security, error handling, and comprehensive API documentation. Focus on clean code, proper separation of concerns, and maintainability.