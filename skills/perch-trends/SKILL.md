# PerchHQ Trends Generator Skill

**Purpose:** Generate trend files in the exact canonical format for PerchHQ Trends repository.

## Trigger

Use this skill whenever generating, updating, or validating trend JSON files for the PerchHQ-Trends repository.

## Canonical Format (STRICT)

### File Structure
```json
{
  "batch_id": "trends-YYYY-MM-DD",
  "scraped_at": "YYYY-MM-DDTHH:MM:SSZ",
  "expires_at": "YYYY-MM-DDTHH:MM:SSZ",
  "trends": [ ... ]
}
```

### Trend Object
```json
{
  "headline": "Short descriptive title",
  "body": "1-2 sentence description",
  "categories": ["Beauty & Skincare"],
  "source": "TikTok",
  "source_url": "https://...",
  "trend_type": "product",
  "confidence_score": 0.85,
  "relevance_score": 9,
  "metadata": {
    "hashtags": ["#trending"],
    "engagement": { "mentions": "high", "editor_approved": true },
    "sentiment": "positive",
    "related_brands": ["Brand"],
    "context": "What kind of trend",
    "price_point": "mid-range"
  }
}
```

## Validation Rules

Before saving ANY trend file, validate:

1. ✅ Root has `batch_id` (format: `trends-YYYY-MM-DD`)
2. ✅ Root has `scraped_at` (ISO 8601 with T and Z)
3. ✅ Root has `expires_at` (3 days after scraped_at)
4. ✅ Root has flat `trends` array (NOT nested under categories)
5. ✅ Each trend has all required fields
6. ✅ `engagement` is object with `mentions` + `editor_approved` (NOT views/likes)
7. ✅ No null values anywhere
8. ✅ `metadata` has `context` and `price_point`

## Generation Steps

1. **Extract date** from the trend date (e.g., "2026-02-19")
2. **Set batch_id**: `trends-{date}`
3. **Set scraped_at**: `{date}T10:00:00Z`
4. **Set expires_at**: date + 3 days, format `{date}T10:00:00Z`
5. **Build trends array** with each trend object
6. **Validate** using rules above before saving
7. **Test JSON validity**: `python3 -c "import json; json.load(open('file.json'))"`

## Common Pitfalls

| ❌ Wrong | ✅ Correct |
|----------|------------|
| `engagement: { "views": 1000 }` | `engagement: { "mentions": "high", "editor_approved": true }` |
| Missing `context` | `context: "viral recipe"` |
| Missing `price_point` | `price_point: "mid-range"` |
| `date: "2026-02-19"` | `batch_id: "trends-2026-02-19"` |
| `trends` nested under `categories` | Flat `trends: [...]` array |
| `null` values | Always provide default value |

## Quick Reference

- **10 Categories:** Beauty & Skincare, Fashion & Apparel, Food & Beverage, Health & Fitness, Home & Lifestyle, Tech & Gadgets, Baby & Kids, Pets, Travel & Hospitality, Jewelry & Accessories
- **Trend Types:** product, content_format, viral_moment, hashtag
- **Engagement mentions:** "high", "medium", "low"
- **Price points:** budget, mid-range, luxury, various
- **Sentiments:** positive, negative, mixed, neutral
