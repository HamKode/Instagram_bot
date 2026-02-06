# Instagram Comment Bot with AI - Setup Guide

## Overview
Automatically responds to Instagram comments using AI to detect intent (price, availability, size, link requests) and sends personalized DMs with product details from Shopify. Increases engagement 3x and converts comments into sales.

## Workflow Architecture

**12 Nodes:**
1. Webhook Trigger - Receives Instagram comment webhooks
2. Parse Comment Data - Extracts comment info
3. AI Intent Detector - AI Agent analyzes comment intent
4. OpenAI Model - Powers AI analysis (GPT-4o-mini)
5. Parse Intent - Structures AI output
6. Fetch Product from Shopify - Gets product details
7. Enhance DM with Product - Creates personalized DM
8. Reply to Comment - Posts public reply
9. Should Send DM - Checks if DM needed
10. Send DM - Sends personalized message
11. Log to Supabase - Tracks all interactions
12. Webhook Response - Returns success

## Setup Steps

### 1. Instagram Business Account Setup

**Requirements:**
- Instagram Business or Creator account
- Facebook Page connected to Instagram
- Meta Developer account

**Steps:**
1. Go to https://developers.facebook.com
2. Create new app → Business type
3. Add Instagram Graph API product
4. Generate access token with permissions:
   - `instagram_basic`
   - `instagram_manage_comments`
   - `instagram_manage_messages`
   - `pages_read_engagement`
   - `pages_manage_metadata`

### 2. Webhook Configuration

**Setup Instagram Webhooks:**
1. In Meta Developer App → Webhooks
2. Subscribe to Instagram → Comments
3. Callback URL: `https://your-n8n.com/webhook/instagram-comment`
4. Verify token: Set any random string
5. Subscribe to fields: `comments`

**Test Webhook:**
```bash
curl -X POST https://your-n8n.com/webhook/instagram-comment \
-H "Content-Type: application/json" \
-d '{
  "comment_id": "17234567890",
  "post_id": "17987654321",
  "from": {
    "id": "123456789",
    "username": "testuser"
  },
  "text": "price?",
  "timestamp": "2024-01-15T10:30:00Z",
  "media": {
    "id": "17987654321",
    "caption": "Premium Wireless Headphones - Best Sound Quality",
    "media_url": "https://instagram.com/p/xyz"
  }
}'
```

### 3. Environment Variables

```
SHOPIFY_STORE=your-store-name
INSTAGRAM_ACCESS_TOKEN=your_instagram_access_token
OPENAI_API_KEY=your_openai_api_key
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_KEY=your_supabase_anon_key
```

### 4. Supabase Database Setup

**Create Table:**
```sql
CREATE TABLE instagram_comments (
  id SERIAL PRIMARY KEY,
  comment_id VARCHAR(255) UNIQUE,
  post_id VARCHAR(255),
  user_id VARCHAR(255),
  username VARCHAR(255),
  comment_text TEXT,
  intent VARCHAR(50),
  confidence INTEGER,
  sentiment VARCHAR(20),
  public_reply TEXT,
  dm_sent BOOLEAN,
  product_url TEXT,
  processed_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_comment_id ON instagram_comments(comment_id);
CREATE INDEX idx_user_id ON instagram_comments(user_id);
CREATE INDEX idx_intent ON instagram_comments(intent);
```

### 5. Credentials Required

**OpenAI API:**
- Get key: https://platform.openai.com/api-keys
- Model: gpt-4o-mini (fast + cheap)

**Shopify API:**
- Admin API access token
- Permissions: `read_products`

**Instagram Access Token:**
- Long-lived token (60 days)
- Refresh before expiry

**Supabase:**
- Project URL and anon key
- From project settings

## How It Works

### Flow:
1. User comments "price?" on Instagram post
2. Webhook triggers n8n workflow
3. AI analyzes comment → detects "price_inquiry" intent
4. Fetches product from Shopify based on post caption
5. Replies to comment: "Shukriya! Details DM me bhej diye 😊"
6. Sends DM with product price, link, discount code
7. Logs interaction to Supabase

### Intent Detection:

**AI detects 7 intents:**
- `price_inquiry` - "price?", "kitne ka?"
- `availability_check` - "available?", "in stock?"
- `size_inquiry` - "size?", "M available?"
- `link_request` - "link?", "send link"
- `general_interest` - "nice", "love it", "😍"
- `question` - "delivery?", "quality?"
- `not_interested` - Negative/spam

### Response Examples:

**Comment: "price?"**
- Public Reply: "Shukriya! 😊 Price aur details DM me send kar diye. Check karein!"
- DM: Full product details with price, link, discount code

**Comment: "available hai?"**
- Public Reply: "Available hai! Link DM me bhej diya. Order kar sakte hain 🛍️"
- DM: Product availability + link + features

**Comment: "😍😍😍"**
- Public Reply: "Thank you! 💕 Product link DM me check karein!"
- DM: Product details with special offer

## Testing

### Test 1: Price Inquiry
```json
{
  "text": "price kitna hai?",
  "from": {"username": "testuser", "id": "123"}
}
```
**Expected:** Public reply + DM with price

### Test 2: Link Request
```json
{
  "text": "link bhejo please",
  "from": {"username": "buyer123", "id": "456"}
}
```
**Expected:** Public reply + DM with product link

### Test 3: General Interest
```json
{
  "text": "😍😍 love this!",
  "from": {"username": "fan789", "id": "789"}
}
```
**Expected:** Public reply + DM with offer

## Customization

### Change Reply Language
In AI prompt, modify:
```
Language: Hinglish (mix of Hindi and English)
Change to: Pure English or Pure Hindi
```

### Add Discount Codes
In "Enhance DM" node:
```javascript
const discountCode = intent.urgency === 'immediate' ? 'INSTA20' : 'INSTA10';
```

### Filter Spam
Add spam detection:
```javascript
const spamKeywords = ['dm for paid promotion', 'follow back', 'check my profile'];
if (spamKeywords.some(k => comment.toLowerCase().includes(k))) {
  return { should_send_dm: false };
}
```

## Performance Metrics

**Expected Results:**
- 90%+ intent detection accuracy
- 3x increase in engagement
- 25-40% comment-to-DM conversion
- 10-15% DM-to-sale conversion
- <30 second response time

**ROI:**
- Manual comment replies: 2-3 min each
- Automated: <5 seconds
- Time saved: 100+ hours/month for active brands
- Sales increase: 20-30% from engaged users

## Troubleshooting

### Issue: Webhook not receiving comments
**Solution:** Check Instagram webhook subscription. Verify callback URL. Test with Meta's webhook tester.

### Issue: DM not sending
**Solution:** Verify Instagram access token. Check if user has messaged page before (required for DM). Use comment reply if DM fails.

### Issue: Wrong product fetched
**Solution:** Improve post caption matching. Add product ID in caption. Use hashtags for product identification.

### Issue: AI intent wrong
**Solution:** Add more examples to prompt. Increase confidence threshold. Review logged data for patterns.

## Best Practices

1. **Respond Fast:** Set up instant notifications for high-intent comments
2. **Personalize DMs:** Use username, reference their comment
3. **Track Conversions:** Link DM clicks to sales in Shopify
4. **A/B Test Replies:** Try different public reply styles
5. **Monitor Sentiment:** Flag negative comments for manual review
6. **Update Product Links:** Ensure Shopify products are active
7. **Refresh Tokens:** Instagram tokens expire, set reminders

## Advanced Features

### Auto-Follow Up
Add delay node:
```
If DM sent but no reply in 24 hours:
  Send follow-up: "Still interested? Let me know if you have questions! 😊"
```

### VIP Detection
Check user followers:
```javascript
if (userFollowers > 10000) {
  discountCode = 'VIP25'; // Higher discount for influencers
}
```

### Multi-Product Posts
Parse caption for multiple products:
```javascript
const products = caption.match(/#product\d+/g);
// Fetch all products and send carousel in DM
```

---

**Created by: Hussain Ahmed Madni**
