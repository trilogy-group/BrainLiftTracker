# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Running the Application
```bash
# Activate virtual environment (Windows)
venv\Scripts\activate

# Run the Flask application on port 5555
python app.py
```

### Development Setup
```bash
# Create virtual environment
python -m venv venv

# Install dependencies
pip install -r requirements.txt

# Copy and configure environment variables
cp .env.example .env
```

### Testing
- No unit test framework is configured
- Use the Postman collection `Twitter_Manager_API.postman_collection.json` for API testing
- Mock mode can be enabled for testing without real Twitter posts via `/api/v1/mock-mode` endpoint

## Architecture Overview

This is a single-file Flask application (`app.py`) that implements a RESTful API for managing multiple Twitter accounts. Key architectural patterns:

### Database Layer
- Direct SQLite3 queries (no ORM) with connection management via `get_db()` function
- Database auto-creates in `instance/twitter_manager.db` on first run
- Tables: `api_key`, `twitter_account`, `tweet`, `oauth_state`, `twitter_list`, `list_membership`
- Account types: 'managed' (default) or 'list_owner' for list management permissions

### Authentication & Security
- API authentication via `X-API-Key` header checked by `check_api_key()` decorator
- Twitter OAuth 2.0 with PKCE flow for account authorization
- Credentials encrypted using Fernet encryption before database storage
- Environment-based configuration for sensitive values

### Core Functionality Flow
1. **Account Addition**: OAuth flow initiated via `/api/v1/auth/twitter`, callback handled at `/auth/callback`
2. **Tweet Creation**: Posts created with "pending" status via `/api/v1/tweet`
3. **Tweet Posting**: Individual or batch posting to Twitter via X API v2
4. **Token Management**: Automatic refresh token handling with encrypted storage
5. **List Management**: Create and manage Twitter lists with designated list_owner accounts
6. **List Membership**: Add/remove managed accounts to/from lists with bulk operations

### API Structure
- Base URL: `http://localhost:5555/api/v1/`
- All endpoints except health check and OAuth callback require API key authentication
- Consistent JSON responses with error handling

### External Dependencies
- Twitter API v2 for posting (using direct HTTP requests, not tweepy)
- Minimal Python dependencies: flask, python-dotenv, cryptography, requests

## Important Notes

- The application runs on port 5555 (not the default Flask port)
- Database and logs directories are auto-created if missing
- OAuth callback URL must be exactly `http://localhost:5555/auth/callback` in Twitter app settings
- All Twitter credentials are encrypted before storage
- Tweet lifecycle: pending → posted/failed
- Batch operations available for posting multiple tweets across accounts