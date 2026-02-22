# PRD: PerchHQ Sparks + Marketing Calendar

**Status:** Draft v0.1  
**Date:** 2026-02-09  
**Author:** JaxBot + Randy Digital

---

## Overview

**Sparks** is an AI-powered content ideation engine for PerchHQ that combines:
- A curated marketing calendar (holidays, trending events, cultural moments)
- Real-time trend research (last30days skill)
- AI-generated content ideas ("Sparks") tailored to each event

**Goal:** Content creators never run out of ideas. PerchHQ surfaces relevant, timely content suggestions before they happen.

---

## User Experience

### The "Spark" Card

```
┌─────────────────────────────────────┐
│ 🍩 National Donut Day               │
│ Friday, Jun 5 • 3 days away         │
├─────────────────────────────────────┤
│                                     │
│ ✨ Spark 1: Behind-the-Scenes       │
│ "Film your morning routine but      │
│  replace everything with donuts"    │
│                                     │
│ 📈 Trending: #morningroutine (↑15%) │
│ 💡 Hook: "POV: You're living in a   │
│          donut shop"                │
│                                     │
│ [Copy Hook] [Save Idea] [Create]    │
│                                     │
├─────────────────────────────────────┤
│ ✨ Spark 2: Educational             │
│ "5 donut facts that will blow your  │
│  mind (I can't believe #3)"         │
│                                     │
│ 📊 Research: Educational content    │
│   up 23% this week                  │
│                                     │
│ [Copy Hook] [Save Idea] [Create]    │
└─────────────────────────────────────┘
```

---

## Database Schema (Supabase)

### `marketing_events`
Core calendar of events/holidays.

```sql
create table marketing_events (
  id uuid default gen_random_uuid() primary key,
  name text not null,
  slug text unique not null,
  
  -- Timing
  event_date date not null,
  is_recurring boolean default false,           -- Annual holiday?
  recurring_pattern text,                       -- "annually", "first-friday-may"
  
  -- Categorization
  category text not null,                       -- holiday | trending | cultural | industry
  tags text[] default '{}',
  
  -- Content
  description text,
  default_hashtags text[] default '{}',
  related_topics text[] default '{}',           -- For last30days research
  
  -- AI Configuration
  spark_templates jsonb default '{}',           -- Template prompts for this event
  
  -- Metadata
  is_active boolean default true,
  created_at timestamp with time zone default now(),
  updated_at timestamp with time zone default now()
);

-- Index for fast upcoming queries
create index idx_marketing_events_date on marketing_events(event_date);
create index idx_marketing_events_active on marketing_events(is_active) where is_active = true;
```

### `sparks`
AI-generated content ideas for each event.

```sql
create table sparks (
  id uuid default gen_random_uuid() primary key,
  marketing_event_id uuid references marketing_events(id) on delete cascade,
  
  -- Content
  title text not null,
  description text,
  content_format text not null,                 -- reel | carousel | story | text | live
  
  -- The Hook (copy-paste openers)
  suggested_hooks text[] default '{}',
  
  -- Research Data (cached last30days results)
  research_summary jsonb default '{}',
  trending_hashtags text[] default '{}',
  trending_sounds text[] default '{}',          -- For Reels/TikTok
  
  -- AI Metadata
  ai_prompt text,                               -- Full prompt used to generate
  engagement_prediction int check (engagement_prediction between 1 and 10),
  
  -- Usage Tracking
  copy_count int default 0,
  save_count int default 0,
  create_count int default 0,
  
  -- Status
  status text default 'active',                 -- active | archived | used
  created_at timestamp with time zone default now(),
  updated_at timestamp with time zone default now()
);

-- Indexes
create index idx_sparks_event on sparks(marketing_event_id);
create index idx_sparks_status on sparks(status);
create index idx_sparks_format on sparks(content_format);
```

### `user_saved_sparks`
Creators can save sparks for later.

```sql
create table user_saved_sparks (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users(id) on delete cascade,
  spark_id uuid references sparks(id) on delete cascade,
  
  -- User customization
  personal_notes text,
  scheduled_for date,
  
  created_at timestamp with time zone default now(),
  
  unique(user_id, spark_id)
);
```

### `spark_generations` (Audit Log)
Track when/what AI generated.

```sql
create table spark_generations (
  id uuid default gen_random_uuid() primary key,
  marketing_event_id uuid references marketing_events(id),
  
  -- Generation metadata
  triggered_at timestamp with time zone default now(),
  triggered_by text default 'cron',             -- cron | manual | api
  
  -- Research data
  last30days_query text,
  research_results jsonb,
  
  -- Generation results
  sparks_generated int default 0,
  generation_time_ms int,
  model_used text,
  
  -- Errors (if any)
  error_message text,
  status text default 'success'                 -- success | partial | failed
);
```

---

## API Design

### Get Upcoming Events with Sparks
```http
GET /api/sparks/upcoming?days=7&limit=10

Response:
{
  "events": [
    {
      "id": "uuid",
      "name": "National Donut Day",
      "event_date": "2026-02-13",
      "days_away": 3,
      "category": "holiday",
      "sparks": [
        {
          "id": "uuid",
          "title": "Behind-the-Scenes Donut Routine",
          "content_format": "reel",
          "suggested_hooks": ["POV: You're living in a donut shop"],
          "trending_hashtags": ["#morningroutine", "#donutday"],
          "engagement_prediction": 8
        }
      ]
    }
  ]
}
```

### Generate Sparks for Event
```http
POST /api/sparks/generate
{
  "marketing_event_id": "uuid",
  "force_refresh": false  -- Skip if sparks exist and are recent
}
```

### Track Spark Usage
```http
POST /api/sparks/{id}/track
{
  "action": "copy" | "save" | "create"
}
```

---

## Automation Flow

### Daily Cron (9 AM EST)
```
1. Find events happening in next 3-7 days
2. For each event without recent sparks:
   a. Run last30days research on event topics
   b. Generate 3-5 sparks using AI
   c. Cache research in sparks.research_summary
3. Push notification to users with relevant saved interests
```

### Last30Days Integration
```javascript
// Pseudo-code for spark generation
async function generateSparks(event) {
  // Research what's trending for this topic
  const research = await last30days.research({
    query: `${event.name} content ideas ${event.related_topics.join(' ')}`,
    sources: ['reddit', 'twitter', 'web']
  });
  
  // Generate sparks based on research
  const sparks = await ai.generate({
    prompt: `Generate 3 content ideas for ${event.name}...
             
             Research context: ${research.summary}
             
             Trending formats: ${research.trendingFormats}
             Popular hooks: ${research.viralHooks}`,
    output: 'json_array'
  });
  
  // Save to database
  await db.sparks.insert(sparks.map(s => ({
    ...s,
    marketing_event_id: event.id,
    research_summary: research
  })));
}
```

---

## JaxBot OS Integration (Future)

### Marketing Calendar App
- View full calendar of upcoming events
- Preview sparks for each event
- Manually trigger spark generation
- Edit marketing events
- Analytics dashboard (which sparks perform best)

### Features
- **Calendar View:** Month/week view of all marketing events
- **Spark Editor:** Refine AI-generated sparks before publishing
- **Research Panel:** View raw last30days research data
- **Analytics:** Track which sparks get used most
- **Quick Add:** Create new marketing events manually

---

## Content Formats Supported

| Format | Description | Best For |
|--------|-------------|----------|
| `reel` | Short-form vertical video | Trends, behind-scenes |
| `carousel` | Multi-image swipe post | Educational, lists |
| `story` | 24hr ephemeral content | Polls, quick updates |
| `text` | Static text post | Quotes, questions |
| `live` | Live streaming | Q&A, launches |
| `thread` | Twitter/X thread | Deep dives, stories |

---

## Seed Data: Marketing Events

```sql
-- Q1 2026 Sample Events
INSERT INTO marketing_events (name, event_date, category, is_recurring, description, default_hashtags, related_topics) VALUES
('New Year''s Day', '2026-01-01', 'holiday', true, 'Fresh start content', '{"#newyear", "#freshstart"}', '{"new years resolutions", "goal setting"}'),
('Valentine''s Day', '2026-02-14', 'holiday', true, 'Love and relationships', '{"#valentines", "#love"}', '{"date ideas", "self love", "gift ideas"}'),
('National Tortellini Day', '2026-02-13', 'holiday', true, 'Italian food content', '{"#tortellini", "#pasta"}', '{"italian food", "recipes", "comfort food"}'),
('Super Bowl Sunday', '2026-02-08', 'cultural', true, 'Sports and entertainment', '{"#superbowl", "#gameday"}', '{"football", "parties", "commercials"}'),
('Spring Launch Season', '2026-03-01', 'industry', false, 'Product launches', '{"#newlaunch", "#springcollection"}', '{"product photography", "launches"}');
```

---

## Open Questions

1. **Event Sources:** Curate manually, pull from external APIs (Google Calendar holidays, etc.), or both?
2. **Personalization:** Should sparks adapt to user's niche/content history?
3. **Timing:** How many days in advance to surface sparks? (3? 7? Vary by event size?)
4. **Research Scope:** How deep should last30days research go? (Top 3 results? 10?)
5. **User Control:** Can users "mute" certain event categories?

---

## Next Steps

- [ ] Review schema with Randy
- [ ] Create Supabase migration
- [ ] Build seed data for Q1-Q2 2026
- [ ] Prototype spark generation pipeline
- [ ] Design PerchHQ UI for spark cards
- [ ] Consider JaxBot OS management app

---

*"Never stare at a blank screen again."* ✨