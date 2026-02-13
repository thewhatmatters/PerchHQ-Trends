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

## 🔄 Data Format

Each trend record includes:

```json
{
  "id": "uuid",
  "headline": "What's trending",
  "body": "Detailed description",
  "categories": ["Category"],
  "source": "TikTok | Instagram | X | Reddit",
  "trend_type": "product | content_format | viral_moment | hashtag",
  "confidence_score": 0.85,
  "relevance_score": 9,
  "metadata": {
    "hashtags": ["#trending"],
    "engagement": { "views": 1000000 },
    "sentiment": "positive",
    "related_brands": ["Brand Name"]
  }
}
```

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