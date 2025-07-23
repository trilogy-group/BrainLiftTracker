# Security Guidelines for Twitter Manager

## Important: Protecting Sensitive Data

### Before Committing to Git

1. **Check .gitignore**: Ensure `.gitignore` is properly configured (already done)
2. **Never commit .env**: The `.env` file contains sensitive keys and is ignored by git
3. **Use .env.example**: Share configuration structure without exposing secrets

### Sensitive Files Protected by .gitignore

- `.env` - Contains API keys, secrets, and tokens
- `*.db` - Database files with encrypted tokens
- `instance/` - Flask instance folder with database
- `logs/` - May contain sensitive information
- `*.pem`, `*.key` - Certificate and key files

### Environment Variables

The following sensitive values MUST be in `.env` and NEVER in code:

1. **API_KEY** - Your Twitter Manager API key
2. **TWITTER_CLIENT_ID** - From Twitter Developer Portal
3. **TWITTER_CLIENT_SECRET** - From Twitter Developer Portal  
4. **ENCRYPTION_KEY** - For encrypting stored tokens

### How to Set Up Securely

1. Copy the example file:
   ```bash
   cp .env.example .env
   ```

2. Generate an encryption key:
   ```python
   python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
   ```

3. Generate an API key:
   ```python
   import secrets
   print(secrets.token_hex(32))
   ```

4. Fill in your Twitter API credentials from the Developer Portal

### Security Features

1. **Token Encryption**: All OAuth tokens are encrypted before database storage
2. **API Key Authentication**: All endpoints (except health) require API key
3. **PKCE OAuth Flow**: Enhanced security for OAuth 2.0
4. **No Hardcoded Secrets**: All sensitive values from environment

### Git Commands for Safety

Before committing, always check:

```bash
# Check what will be committed
git status

# Check if .env is ignored
git check-ignore .env

# View files that would be added
git add --dry-run .

# If you accidentally staged .env
git rm --cached .env
```

### If You Accidentally Commit Secrets

1. **Immediately revoke** the exposed credentials
2. **Generate new** API keys and tokens
3. **Remove from history**:
   ```bash
   git filter-branch --force --index-filter \
     "git rm --cached --ignore-unmatch .env" \
     --prune-empty --tag-name-filter cat -- --all
   ```
4. **Force push** to remote (coordinate with team)

### Best Practices

1. ✅ Always use environment variables for secrets
2. ✅ Rotate API keys periodically
3. ✅ Use different keys for development/production
4. ✅ Review files before committing
5. ❌ Never hardcode credentials in source code
6. ❌ Never commit .env files
7. ❌ Never log sensitive information

### Checking Your Setup

Run this to verify your security setup:

```bash
# Should return: .env
git check-ignore .env

# Should NOT show .env in output
git ls-files

# Check for exposed secrets in code
grep -r "TWITTER_CLIENT_SECRET\|API_KEY.*=" --include="*.py" .
```

Remember: **When in doubt, don't commit!**