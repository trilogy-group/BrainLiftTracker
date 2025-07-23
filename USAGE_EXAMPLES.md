# Twitter Manager API - Usage Examples

This document provides detailed examples of how to use the Twitter Manager API.

## Initial Setup

### 1. Generate Required Keys

```bash
# Generate encryption key
flask generate-encryption-key
# Output: New encryption key: fKs1H2PnDKb6oqM6J9Xy5cZ3vNmLkQwErTyU8iBnGhA=

# Create API key for authentication
flask create-api-key
# Enter name when prompted
# Output: API Key created successfully!
# Name: MyApp
# Key: qWX2n9Kf5mPr8Tj6Nc3Vb7Zx4Lh1Gs9Yd0Wa5Eu8Ri2
```

### 2. Environment Configuration

Create `.env` file:
```env
SECRET_KEY=dev-secret-key-change-in-production
API_KEY=qWX2n9Kf5mPr8Tj6Nc3Vb7Zx4Lh1Gs9Yd0Wa5Eu8Ri2
ENCRYPTION_KEY=fKs1H2PnDKb6oqM6J9Xy5cZ3vNmLkQwErTyU8iBnGhA=

TWITTER_CLIENT_ID=your-twitter-client-id
TWITTER_CLIENT_SECRET=your-twitter-client-secret
TWITTER_CALLBACK_URL=http://localhost:5000/api/v1/auth/callback

DATABASE_URL=sqlite:///instance/twitter_manager.db
LOG_LEVEL=INFO
```

## Python Client Example

```python
import requests
import json
from datetime import datetime, timedelta

class TwitterManagerClient:
    def __init__(self, base_url, api_key):
        self.base_url = base_url
        self.headers = {
            'X-API-Key': api_key,
            'Content-Type': 'application/json'
        }
    
    def add_account(self, access_token, access_token_secret):
        """Add a new Twitter account"""
        response = requests.post(
            f"{self.base_url}/accounts",
            headers=self.headers,
            json={
                'access_token': access_token,
                'access_token_secret': access_token_secret
            }
        )
        return response.json()
    
    def list_accounts(self):
        """List all accounts"""
        response = requests.get(
            f"{self.base_url}/accounts",
            headers=self.headers
        )
        return response.json()
    
    def post_tweet(self, text, account_ids=None, post_to_all=False, scheduled_at=None):
        """Post a tweet"""
        data = {'text': text}
        
        if post_to_all:
            data['post_to_all'] = True
        elif account_ids:
            data['account_ids'] = account_ids
        
        if scheduled_at:
            data['scheduled_at'] = scheduled_at
        
        response = requests.post(
            f"{self.base_url}/tweet",
            headers=self.headers,
            json=data
        )
        return response.json()
    
    def get_stats(self):
        """Get statistics"""
        response = requests.get(
            f"{self.base_url}/stats",
            headers=self.headers
        )
        return response.json()

# Usage
client = TwitterManagerClient(
    base_url='http://localhost:5000/api/v1',
    api_key='your-api-key-here'
)

# Post to all accounts
result = client.post_tweet(
    text="Hello from Python client!",
    post_to_all=True
)
print(json.dumps(result, indent=2))

# Schedule a tweet for tomorrow
tomorrow = (datetime.now() + timedelta(days=1)).isoformat()
result = client.post_tweet(
    text="This is a scheduled tweet",
    account_ids=[1, 2],
    scheduled_at=tomorrow
)
print(json.dumps(result, indent=2))
```

## Bash/cURL Examples

### Adding an Account via OAuth

```bash
# Step 1: Get OAuth URL
AUTH_RESPONSE=$(curl -s -X GET http://localhost:5000/api/v1/auth/twitter \
  -H "X-API-Key: your-api-key")

echo "Visit this URL to authorize: $(echo $AUTH_RESPONSE | jq -r .auth_url)"
echo "State: $(echo $AUTH_RESPONSE | jq -r .state)"

# Step 2: After user authorizes, handle callback
curl -X POST http://localhost:5000/api/v1/auth/callback \
  -H "X-API-Key: your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "code-from-twitter-redirect",
    "state": "state-from-step-1"
  }'
```

### Batch Operations Script

```bash
#!/bin/bash

API_KEY="your-api-key"
BASE_URL="http://localhost:5000/api/v1"

# Function to post tweet
post_tweet() {
    local text="$1"
    local response=$(curl -s -X POST "$BASE_URL/tweet" \
        -H "X-API-Key: $API_KEY" \
        -H "Content-Type: application/json" \
        -d "{
            \"text\": \"$text\",
            \"post_to_all\": true
        }")
    
    echo "$response" | jq .
}

# Post multiple tweets
tweets=(
    "Good morning! Starting the day with some coding"
    "Check out our latest blog post on API development"
    "Thank you to all our followers!"
)

for tweet in "${tweets[@]}"; do
    echo "Posting: $tweet"
    post_tweet "$tweet"
    sleep 5  # Rate limiting
done
```

## JavaScript/Node.js Example

```javascript
const axios = require('axios');

class TwitterManagerAPI {
    constructor(baseUrl, apiKey) {
        this.client = axios.create({
            baseURL: baseUrl,
            headers: {
                'X-API-Key': apiKey,
                'Content-Type': 'application/json'
            }
        });
    }

    async postTweet(text, options = {}) {
        const { data } = await this.client.post('/tweet', {
            text,
            ...options
        });
        return data;
    }

    async getAccounts() {
        const { data } = await this.client.get('/accounts');
        return data;
    }

    async scheduleTweet(text, accountIds, scheduledAt) {
        return await this.postTweet(text, {
            account_ids: accountIds,
            scheduled_at: scheduledAt
        });
    }
}

// Usage
const api = new TwitterManagerAPI(
    'http://localhost:5000/api/v1',
    'your-api-key'
);

// Post to specific accounts
api.postTweet('Hello from Node.js!', {
    account_ids: [1, 2, 3]
}).then(result => {
    console.log('Tweet posted:', result);
}).catch(error => {
    console.error('Error:', error.response.data);
});

// Schedule tweets for the next 7 days
async function scheduleWeeklyTweets() {
    const tweets = [
        'Monday motivation!',
        'Tuesday tips',
        'Wednesday wisdom',
        'Thursday thoughts',
        'Friday feeling',
        'Saturday vibes',
        'Sunday summary'
    ];

    const accounts = await api.getAccounts();
    const activeAccounts = accounts.items
        .filter(acc => acc.status === 'active')
        .map(acc => acc.id);

    for (let i = 0; i < tweets.length; i++) {
        const scheduledAt = new Date();
        scheduledAt.setDate(scheduledAt.getDate() + i + 1);
        scheduledAt.setHours(9, 0, 0, 0); // 9 AM

        await api.scheduleTweet(
            tweets[i],
            activeAccounts,
            scheduledAt.toISOString()
        );
        
        console.log(`Scheduled: ${tweets[i]} for ${scheduledAt}`);
    }
}
```

## Error Handling Examples

```python
import requests
from requests.exceptions import HTTPError

def safe_post_tweet(client, text):
    try:
        result = client.post_tweet(text, post_to_all=True)
        print(f"Success: {result}")
        return result
    except requests.exceptions.HTTPError as e:
        if e.response.status_code == 400:
            error = e.response.json()
            print(f"Bad request: {error['error']}")
        elif e.response.status_code == 401:
            print("Authentication failed - check your API key")
        elif e.response.status_code == 429:
            print("Rate limit exceeded - try again later")
        else:
            print(f"HTTP error: {e}")
    except Exception as e:
        print(f"Unexpected error: {e}")
```

## Monitoring and Analytics

```python
def generate_report(client):
    """Generate a daily report of Twitter activity"""
    stats = client.get_stats()
    accounts = client.list_accounts()
    
    print("=== Twitter Manager Daily Report ===")
    print(f"\nAccount Summary:")
    print(f"  Total Accounts: {stats['accounts']['total']}")
    print(f"  Active Accounts: {stats['accounts']['active']}")
    print(f"  Inactive Accounts: {stats['accounts']['inactive']}")
    
    print(f"\nTweet Statistics:")
    print(f"  Total Tweets: {stats['tweets']['total']}")
    print(f"  Successful: {stats['tweets']['successful']}")
    print(f"  Failed: {stats['tweets']['failed']}")
    print(f"  Scheduled: {stats['tweets']['scheduled']}")
    print(f"  Success Rate: {stats['tweets']['success_rate']}%")
    
    print(f"\nPer-Account Performance:")
    for account in accounts['items']:
        print(f"  @{account['username']}:")
        print(f"    Status: {account['status']}")
        print(f"    Success Rate: {account['statistics']['success_rate']}%")
        print(f"    Total Tweets: {account['statistics']['total_tweets']}")
```

## Advanced Scheduling Example

```python
from datetime import datetime, timedelta
import random

def schedule_content_calendar(client, content_plan):
    """
    Schedule a content calendar with optimal posting times
    
    content_plan = [
        {'text': 'Morning post', 'time': '09:00'},
        {'text': 'Lunch time update', 'time': '12:30'},
        {'text': 'Evening engagement', 'time': '18:00'}
    ]
    """
    accounts = client.list_accounts()
    active_accounts = [acc['id'] for acc in accounts['items'] 
                      if acc['status'] == 'active']
    
    if not active_accounts:
        print("No active accounts found")
        return
    
    results = []
    for day in range(7):  # Schedule for next 7 days
        date = datetime.now() + timedelta(days=day+1)
        
        for post in content_plan:
            # Parse time
            hour, minute = map(int, post['time'].split(':'))
            scheduled_time = date.replace(hour=hour, minute=minute, second=0)
            
            # Add some randomness (±15 minutes)
            random_minutes = random.randint(-15, 15)
            scheduled_time += timedelta(minutes=random_minutes)
            
            # Schedule the post
            result = client.post_tweet(
                text=post['text'],
                account_ids=active_accounts,
                scheduled_at=scheduled_time.isoformat()
            )
            
            results.append({
                'date': scheduled_time,
                'text': post['text'],
                'result': result
            })
    
    return results
```

## Webhook Integration Example

```python
from flask import Flask, request, jsonify
import hmac
import hashlib

app = Flask(__name__)
WEBHOOK_SECRET = 'your-webhook-secret'

@app.route('/webhook/twitter-posted', methods=['POST'])
def handle_tweet_posted():
    """Handle webhook when tweet is posted"""
    
    # Verify webhook signature
    signature = request.headers.get('X-Webhook-Signature')
    if not verify_signature(request.data, signature):
        return jsonify({'error': 'Invalid signature'}), 401
    
    data = request.json
    
    # Process the webhook
    if data['event'] == 'tweet.posted':
        tweet_id = data['tweet_id']
        account = data['account']
        
        # Update your internal systems
        print(f"Tweet {tweet_id} posted by @{account['username']}")
        
        # Maybe send a notification
        send_notification(f"New tweet posted by @{account['username']}")
    
    return jsonify({'status': 'ok'}), 200

def verify_signature(payload, signature):
    """Verify webhook signature"""
    expected = hmac.new(
        WEBHOOK_SECRET.encode(),
        payload,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature)
```

## Testing Rate Limits

```bash
#!/bin/bash

# Test rate limit handling
API_KEY="your-api-key"
BASE_URL="http://localhost:5000/api/v1"

echo "Testing rate limit handling..."

# Post 100 tweets rapidly to trigger rate limits
for i in {1..100}; do
    echo "Posting tweet $i..."
    
    response=$(curl -s -w "\n%{http_code}" -X POST "$BASE_URL/tweet" \
        -H "X-API-Key: $API_KEY" \
        -H "Content-Type: application/json" \
        -d "{
            \"text\": \"Rate limit test tweet #$i\",
            \"account_ids\": [1]
        }")
    
    http_code=$(echo "$response" | tail -n 1)
    body=$(echo "$response" | head -n -1)
    
    if [ "$http_code" == "429" ]; then
        echo "Rate limit hit at tweet $i"
        echo "Response: $body"
        break
    fi
done
```

These examples demonstrate various ways to interact with the Twitter Manager API, from simple posting to complex scheduling and monitoring scenarios.