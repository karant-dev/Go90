# Go90 Scale 📊

A crowdsourced rating app for streaming services inspired by The Vergecast. Rate services from 0 (thriving) to 90 (doomed) and see how the community votes!

![Go90 Scale Demo](https://img.shields.io/badge/Status-Live-success)
![No Build](https://img.shields.io/badge/Build-None_Required-blue)
![License](https://img.shields.io/badge/License-MIT-green)

## ✨ Features

- **Slider-based voting** - Rate any streaming service 0-90
- **Community rankings** - See aggregate scores from all users
- **Real-time updates** - Scores refresh from the database
- **Dark/Light mode** - Toggle between themes
- **Rate limiting** - Prevents spam (1 vote per minute)
- **Session tracking** - Your votes are remembered
- **Mobile responsive** - Works on all devices
- **Demo mode** - Works without any backend!

## 🚀 Quick Start (Demo Mode)

The app works immediately without any setup:

1. Download `index.html`
2. Open in your browser
3. Start voting!

Demo mode stores votes locally and simulates community scores.

## 🔧 Full Setup with Supabase

### Step 1: Create Supabase Project

1. Go to [supabase.com](https://supabase.com) and sign up (free)
2. Click "New Project" and fill in details
3. Wait for the project to initialize (~2 minutes)

### Step 2: Set Up Database

Go to **SQL Editor** and run this schema:

```sql
-- Enable UUID extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Services table
CREATE TABLE services (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name TEXT NOT NULL UNIQUE,
    logo_url TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Votes table
CREATE TABLE votes (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    service_id UUID REFERENCES services(id) ON DELETE CASCADE,
    score INTEGER CHECK(score >= 0 AND score <= 90),
    session_id TEXT NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Index for faster queries
CREATE INDEX idx_votes_service ON votes(service_id);
CREATE INDEX idx_votes_session ON votes(session_id);

-- Aggregated scores view
CREATE OR REPLACE VIEW service_scores AS
SELECT 
    s.id,
    s.name,
    s.logo_url,
    COALESCE(ROUND(AVG(v.score)::numeric, 1), 0) as avg_score,
    COUNT(v.id) as vote_count
FROM services s
LEFT JOIN votes v ON s.id = v.service_id
GROUP BY s.id, s.name, s.logo_url;

-- Enable Row Level Security
ALTER TABLE services ENABLE ROW LEVEL SECURITY;
ALTER TABLE votes ENABLE ROW LEVEL SECURITY;

-- Policies: Anyone can read services
CREATE POLICY "Services are viewable by everyone" 
ON services FOR SELECT 
USING (true);

-- Policies: Anyone can insert votes
CREATE POLICY "Anyone can vote" 
ON votes FOR INSERT 
WITH CHECK (true);

-- Policies: Anyone can read votes (for aggregation)
CREATE POLICY "Votes are viewable by everyone" 
ON votes FOR SELECT 
USING (true);
```

### Step 3: Seed Data

Run this to add the streaming services:

```sql
INSERT INTO services (name) VALUES
    ('Netflix'),
    ('Disney+'),
    ('Max'),
    ('Paramount+'),
    ('Peacock'),
    ('Apple TV+'),
    ('Prime Video'),
    ('Hulu'),
    ('YouTube TV'),
    ('Tubi'),
    ('Pluto TV'),
    ('Freevee'),
    ('Crunchyroll'),
    ('Discovery+'),
    ('ESPN+');
```

### Step 4: Get Your API Keys

1. Go to **Settings** → **API**
2. Copy your:
   - **Project URL** (e.g., `https://xxxxx.supabase.co`)
   - **anon/public key** (the long string)

### Step 5: Configure the App

Edit `index.html` and update these lines (around line 280):

```javascript
const SUPABASE_URL = 'https://your-project.supabase.co';
const SUPABASE_ANON_KEY = 'your-anon-key-here';
const DEMO_MODE = false;  // Change to false!
```

## 🌐 Deployment

### Cloudflare Pages

```bash
# Install Wrangler CLI
npm install -g wrangler

# Login to Cloudflare
wrangler login

# Deploy
wrangler pages deploy . --project-name=go90-scale
```

Or use the Cloudflare dashboard:
1. Go to [pages.cloudflare.com](https://pages.cloudflare.com)
2. Connect your GitHub repo
3. Deploy!

### GitHub Pages

1. Push to a GitHub repository
2. Go to **Settings** → **Pages**
3. Select branch and folder
4. Your site is live at `username.github.io/repo-name`

### Netlify

1. Drag and drop folder to [netlify.com/drop](https://app.netlify.com/drop)
2. Done!

### Vercel

```bash
npx vercel
```

## 📁 Project Structure

```
/
├── index.html      # Complete app (HTML + CSS + JS)
├── README.md       # This file
└── assets/         # Optional: service logos
    └── logos/
```

## 🎨 Customization

### Adding Services

Edit the `DEFAULT_SERVICES` array in `index.html`:

```javascript
const DEFAULT_SERVICES = [
    { id: '1', name: 'Netflix', logo_url: null },
    { id: '2', name: 'Your Service', logo_url: '/assets/logos/service.png' },
    // ...
];
```

### Custom Logos

1. Create `/assets/logos/` folder
2. Add logo images (recommended: 96x96 PNG)
3. Update `logo_url` in services array or database

### Theme Colors

Modify CSS custom properties in the `<style>` section:

```css
:root {
    --accent: #6366f1;        /* Primary accent color */
    --gradient-safe: #10b981; /* "Thriving" color */
    --gradient-doom: #ef4444; /* "Doomed" color */
}
```

## 🔒 Security Notes

- The anon key is safe to expose (it's designed for client-side use)
- Row Level Security (RLS) protects your data
- Rate limiting prevents vote spam
- Session IDs are anonymous (no personal data stored)

## 📊 Analytics Queries

Run these in Supabase SQL Editor:

```sql
-- Most voted services
SELECT name, vote_count 
FROM service_scores 
ORDER BY vote_count DESC;

-- Votes over time
SELECT DATE(created_at) as date, COUNT(*) as votes
FROM votes 
GROUP BY DATE(created_at)
ORDER BY date DESC;

-- Score distribution
SELECT 
    CASE 
        WHEN score < 30 THEN 'Thriving (0-29)'
        WHEN score < 60 THEN 'Struggling (30-59)'
        ELSE 'Doomed (60-90)'
    END as category,
    COUNT(*) as votes
FROM votes
GROUP BY category;
```

## 🤝 Contributing

1. Fork the repo
2. Make your changes
3. Submit a PR

## 📜 License

MIT License - do whatever you want with it!

---

Built with ❤️ for The Vergecast community
