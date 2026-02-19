# PerchHQ Trends

Daily trending topics across creator categories. Automatically researched and curated for content creators.

## 📊 What's Tracked

**10 Creator Categories:**
- Beauty & Skincare
- Fashion & Apparel
- Food & Beverage
- Health & Fitness
- Home & Lifestyle
- Tech & Gadgets
- Baby & Kids
- Pets
- Travel & Hospitality
- Jewelry & Accessories

## 📁 Structure

```
trends/
├── 2026-02-13.json    # Daily trends (11 records)
├── 2026-02-14.json    # Next day's research
└── ...
```

## ⚠️ REQUIRED FORMAT (Strict)

**DO NOT deviate from this schema. All trend files must follow this exact format.**

### File Structure

```json
{
  "batch_id": "trends-YYYY-MM-DD",
  "scraped_at": "YYYY-MM-DDTHH:MM:SSZ",
  "expires_at": "YYYY-MM-DDTHH:MM:SSZ",
  "trends": [ ... ]
}
```

### Trend Record Schema

```json
{
  "headline": "What's trending (short, descriptive)",
  "body": "Detailed description of the trend",
  "categories": ["Category1", "Category2"],
  "source": "TikTok | Instagram | YouTube | Reddit | Web",
  "source_url": "https://...",
  "trend_type": "product | content_format | viral_moment | hashtag",
  "confidence_score": 0.85,
  "relevance_score": 9,
  "metadata": {
    "hashtags": ["#trending", "#relevant"],
    "engagement": { "mentions": "high|medium", "editor_approved": true|false },
    "sentiment": "positive | negative | mixed | neutral",
    "related_brands": ["Brand1", "Brand2"],
    "context": "What kind of trend this is (e.g., viral recipe, product launch, fashion week)",
    "price_point": "budget | mid-range | luxury | various"
  }
}
```

### Field Rules

| Field | Required | Notes |
|-------|----------|-------|
| `batch_id` | ✅ | Format: `trends-YYYY-MM-DD` |
| `scraped_at` | ✅ | ISO 8601 format with `T` and `Z` |
| `expires_at` | ✅ | 3 days after `scraped_at` |
| `trends` | ✅ | Array of trend objects |
| `headline` | ✅ | Short, descriptive title |
| `body` | ✅ | 1-2 sentence description |
| `categories` | ✅ | Array from 10 Creator Categories |
| `source` | ✅ | Platform or publication name |
| `source_url` | ✅ | Full URL to source |
| `trend_type` | ✅ | `product`, `content_format`, `viral_moment`, or `hashtag` |
| `confidence_score` | ✅ | Decimal 0.0-1.0 |
| `relevance_score` | ✅ | Integer 1-10 |
| `metadata` | ✅ | Object with all sub-fields |
| `metadata.hashtags` | ✅ | Array of hashtag strings |
| `metadata.engagement` | ✅ | Object: `{ "mentions": "high|medium", "editor_approved": true|false }` |
| `metadata.sentiment` | ✅ | `positive`, `negative`, `mixed`, or `neutral` |
| `metadata.related_brands` | ✅ | Array of brand names |
| `metadata.context` | ✅ | Short string describing trend context |
| `metadata.price_point` | ✅ | `budget`, `mid-range`, `luxury`, or `various` |

### ⚠️ Common Mistakes to Avoid

- ❌ Using `views`, `likes`, `shares` in engagement — use `mentions` + `editor_approved`
- ❌ Missing `context` field
- ❌ Missing `price_point` field
- ❌ Using `date` instead of `batch_id`
- ❌ Nesting trends under `categories` object — use flat `trends` array
- ❌ Using null values — always provide a value

## ⏰ Update Schedule

**Daily at 4:00 AM CST** — Automated research across all 10 categories.

## 🚀 Usage

### Raw JSON
```bash
# Get today's trends
curl https://raw.githubusercontent.com/thewhatmatters/PerchHQ-Trends/main/trends/2026-02-13.json
```

### In Your App
```javascript
const trends = await fetch(
  'https://raw.githubusercontent.com/thewhatmatters/PerchHQ-Trends/main/trends/2026-02-13.json'
).then(r => r.json());
```

## 📈 Stats

- **Records per day:** 10-50+ trends
- **Confidence threshold:** ≥0.70
- **Data expiry:** 3 days (freshness prioritized)
- **Sources:** TikTok, Instagram, X/Twitter, Reddit, Web

---

*Powered by JaxBot 🏴‍☠️*  
*Research agent for PerchHQ*
