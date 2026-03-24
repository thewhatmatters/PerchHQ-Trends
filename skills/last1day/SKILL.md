---
name: last1day
description: Real-time trending topics across pop culture and creator niches. Surfaces what's trending RIGHT NOW across Beauty, Fashion, Food, Health, Home, Tech, Baby, Pets, Travel, and Jewelry. Use when asked about "what's trending today", "viral right now", or current buzz in specific creator categories.
metadata:
  openclaw:
    emoji: "🔥"
    triggers:
      - pattern: "what's trending"
      - pattern: "trending right now"
      - pattern: "viral today"
      - pattern: "last 1 day"
      - pattern: "current trends"
---

# last1: Real-Time Trending Topics for Creators

Surface what's trending **RIGHT NOW** across pop culture and creator categories. Perfect for content ideation, jumping on trends, and staying current.

## Creator Categories

Default categories (unless user specifies others):
- **Beauty & Skincare**
- **Fashion & Apparel**
- **Food & Beverage**
- **Health & Fitness**
- **Home & Lifestyle**
- **Tech & Gadgets**
- **Baby & Kids**
- **Pets**
- **Travel & Hospitality**
- **Jewelry & Accessories**

## Use Cases

- **Content Ideas**: "What's trending in beauty today?" → viral products, techniques, challenges
- **Trend Jumping**: "What's viral in food right now?" → trending recipes, formats, sounds
- **Niche Research**: "Current pet trends" → popular products, viral pet content
- **General Pop Culture**: "What's trending today?" → cross-category viral moments

## Step 1: Parse Intent

Extract from user input:

1. **CATEGORY** (optional): Which creator niche? Use defaults if not specified.
2. **PLATFORM** (optional): TikTok, Instagram, X/Twitter, YouTube? Check all if not specified.
3. **QUERY_TYPE**:
   - **PRODUCTS** — trending products, viral items, "TikTok made me buy it"
   - **CONTENT** — viral formats, trending sounds, popular video styles
   - **TOPICS** — hashtags, conversations, cultural moments
   - **GENERAL** — mix of everything

## Step 2: Research (Parallel)

### Source A: X/Twitter (via bird)

Search for real-time trending topics in each category:

```bash
# Trending products/viral items
bird search "TikTok made me buy it {CATEGORY}" --limit 20
bird search "viral {CATEGORY} product" --limit 20
bird search "trending {CATEGORY}" --limit 20

# Trending hashtags
bird search "#{CATEGORY} trend" --limit 15
```

### Source B: Web Search (Freshness: Past Day)

```bash
# Products trending today
web_search "viral {CATEGORY} products today" freshness="pd"
web_search "TikTok made me buy it {CATEGORY}" freshness="pd"

# Content trends
web_search "trending {CATEGORY} content {PLATFORM}" freshness="pd"
web_search "viral {CATEGORY} video today" freshness="pd"

# Hashtags/topics
web_search "{CATEGORY} trending hashtags today" freshness="pd"
```

**Run 2-3 searches per category** (focus on top 3-5 categories if checking all 10).

### Source C: Reddit (Real-Time)

```bash
# Relevant subreddits for each category:
# Beauty: r/beauty, r/skincareaddiction, r/makeupaddiction
# Fashion: r/fashion, r/streetwear, r/femalefashionadvice
# Food: r/food, r/cooking, r/baking
# Health: r/fitness, r/loseit, r/progresspics
# Home: r/homeimprovement, r/decor, r/organization
# Tech: r/gadgets, r/technology, r/buyitforlife
# Baby: r/parenting, r/beyondthebump, r/mommit
# Pets: r/aww, r/dogs, r/cats, r/pets
# Travel: r/travel, r/solotravel, r/vacation
# Jewelry: r/jewelry, r/fashion

web_search "site:reddit.com {CATEGORY} trending today" freshness="pd"
```

## Step 3: Synthesize

**For each category, extract:**

1. **🔥 HOT RIGHT NOW** (1-2 items with highest engagement)
2. **📈 RISING** (2-3 items gaining momentum)
3. **💡 CONTENT OPPORTUNITIES** (formats/styles trending)
4. **#️⃣ HASHTAGS TO WATCH**

**Weight by freshness:**
- Last 4 hours = ⬆️ Highest priority
- Last 24 hours = ➡️ Standard priority
- Older = ⬇️ Background context only

## Step 4: Deliver

### Format: Trending Report

```
🔥 TRENDING NOW — {Date}

{CATEGORY 1}: {Name}
━━━━━━━━━━━━━━━━━━━
🔥 HOT: [Specific product/content with engagement stats]
📈 Rising: [2-3 items building momentum]
💡 Format: [Video style/format working now]
#️⃣ Tags: [#trending #hashtags]

{CATEGORY 2}: {Name}
━━━━━━━━━━━━━━━━━━━
🔥 HOT: [...]
...

💡 CROSS-CATEGORY MOMENT:
[If something is trending across multiple niches]

📊 Sources: X posts, Reddit threads, web articles (last 24h)
```

### Example Output:

```
🔥 TRENDING NOW — Feb 10, 2026

BEAUTY & SKINCARE
━━━━━━━━━━━━━━━━━━━
🔥 HOT: "Glass skin" technique using snail mucin — 50K+ TikToks today
📈 Rising: Undereye patches before makeup (dermatologist-approved)
💡 Format: "Get Ready With Me" + product reveal mid-video
#️⃣ Tags: #glassskin #snailmucin #skincaretips

FOOD & BEVERAGE
━━━━━━━━━━━━━━━━━━━
🔥 HOT: Baked feta pasta 2.0 (with miso twist) — viral on TikTok
📈 Rising: Protein powder in coffee, "proffee"
💡 Format: POV cooking with satisfying ASMR sounds
#️⃣ Tags: #bakedfeta #miso #proffee #foodtok
...
```

## Step 5: Structured Output (For Database)

When pushing to Supabase or structured storage, format each trend as:

```json
{
  "headline": "What's trending (viral product, content format, or topic)",
  "body": "Detailed description with context, why it's trending, creator angle",
  "categories": ["Category Name"],
  "source": "TikTok | Instagram | X | Reddit | YouTube | Web",
  "source_url": "https://... (direct link if available)",
  "trend_type": "viral_moment | product | content_format | hashtag",
  "confidence_score": 0.85,
  "relevance_score": 8,
  "metadata": {
    "hashtags": ["#hashtag1", "#hashtag2"],
    "engagement": {
      "views": 94000000,
      "likes": 8200000,
      "shares": null,
      "mentions": "high"
    },
    "sentiment": "positive | neutral | negative",
    "related_brands": ["Brand1", "Brand2"],
    "context": "sale | event | season",
    "price_point": "budget | mid-range | luxury"
  }
}
```

### Scoring Guidelines

**Confidence Score (0.0-1.0):**
- 0.9-1.0: Multiple sources, high engagement, clear viral moment
- 0.7-0.89: Strong signal from 1-2 sources, building momentum
- 0.5-0.69: Emerging trend, early signals
- <0.5: Weak signal, monitor only

**Relevance Score (1-10):**
- 9-10: Directly actionable for creators, high commercial potential
- 7-8: Good content opportunity, trending format
- 5-6: Interesting but niche or fading
- <5: Background context, not priority

### Batch Processing

```javascript
// Generate batch ID for daily run
const batchId = crypto.randomUUID();
const expiresAt = new Date(Date.now() + 72 * 60 * 60 * 1000).toISOString(); // 3 days

// Insert to Supabase
const record = {
  scraped_at: new Date().toISOString(),
  headline: trend.headline,
  body: trend.body,
  categories: trend.categories,
  source: trend.source,
  source_url: trend.source_url,
  trend_type: trend.trend_type,
  status: 'raw',
  confidence_score: trend.confidence_score,
  relevance_score: trend.relevance_score,
  expires_at: expiresAt,
  batch_id: batchId,
  metadata: trend.metadata
};
```

## Step 6: Save (Optional)

```bash
write memory/trends/YYYY-MM-DD-trending.md
```

## Notes

- **Speed over depth** — surface 5-10 hot items fast vs. deep research
- **Check 3-5 categories max** if user asks for "everything" (top priority)
- **No API keys needed** — uses bird, web_search, web_fetch
- **Update frequency**: Trends change hourly — timestamp everything
- **Creator angle**: Focus on what's actionable for content creation
- **Structured output**: Use schema format when saving to database
