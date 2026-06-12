# Tech News Research Agent

## Project Overview

An autonomous AI Agent that continuously gathers technology news from APIs and RSS feeds, categorizes articles, generates summaries, and delivers research reports through communication channels.

## Objective

Develop an automated research workflow that reduces manual effort in discovering, analyzing, and distributing technology news updates.

## Key Features

- News Collection from APIs & RSS Feeds
- Automated Categorization
- AI-Powered Summarization
- Daily/Weekly Report Generation
- Discord & Slack Integration
- Workflow Automation
- Scheduled Research Pipelines

## Concepts & Technologies

- n8n Automation
- APIs & Webhooks
- RSS Feed Processing
- AI Summarization
- Workflow Orchestration
- Notifications & Integrations

## Workflows

| Workflow | Description |
|---|---|
| `News Collection Agent` | Runs every 3 hours — fetches articles from TechCrunch, Wired, MIT Technology Review, The Guardian, and GitHub Trending, normalizes them, and stores in the database |
| `AI Processing` | Runs every 6 hours — picks up pending articles, sends each through GPT-4o-mini for categorization, sentiment analysis, and summarization, then saves results |
| `Daily Report Generator` | Runs every morning at 8 AM — queries the top 20 processed articles, generates a structured daily briefing, and delivers it via Discord, Slack, Gmail, and Telegram |
| `Weekly Report Generator` | Runs every Monday at 9 AM — pulls the last 7 days of articles, generates a weekly trend report, and delivers it across all channels |

## Architecture

```
News Sources (RSS / APIs)
        |
        v
News Collection Agent  (every 3h)
        |
        v
PostgreSQL — raw_articles table
        |
        v
AI Processing Workflow  (every 6h)
        |
        v
PostgreSQL — processed_articles table
        |
        +--> Daily Report Generator  (8 AM daily)
        |
        +--> Weekly Report Generator  (9 AM Monday)
                        |
                        v
        Discord / Slack / Gmail / Telegram
```

## Setup Instructions

### Prerequisites

- [n8n](https://n8n.io/) instance (self-hosted or cloud)
- PostgreSQL database
- OpenAI API key
- Accounts/tokens for the channels you want (Discord, Slack, Gmail, Telegram)

### Database Schema

Create the following tables in your PostgreSQL database before importing the workflows:

```sql
CREATE TABLE raw_articles (
  id           SERIAL PRIMARY KEY,
  source       TEXT,
  title        TEXT,
  url          TEXT UNIQUE,
  published_at TIMESTAMP,
  raw_content  TEXT,
  author       TEXT,
  source_name  TEXT,
  status       TEXT DEFAULT 'PENDING',
  created_at   TIMESTAMP DEFAULT NOW()
);

CREATE TABLE processed_articles (
  id               SERIAL PRIMARY KEY,
  raw_article_id   INTEGER REFERENCES raw_articles(id),
  title            TEXT,
  source_name      TEXT,
  summary          TEXT,
  categories       TEXT[],
  primary_category TEXT,
  relevance_score  INTEGER,
  sentiment        TEXT,
  business_impact  TEXT,
  key_companies    TEXT[],
  key_technologies TEXT[],
  article_type     TEXT,
  processed_at     TIMESTAMP DEFAULT NOW()
);

CREATE TABLE reports (
  id           SERIAL PRIMARY KEY,
  report_type  TEXT,
  content      TEXT,
  generated_at TIMESTAMP DEFAULT NOW()
);
```

### Importing Workflows

1. Open your n8n instance
2. Go to **Workflows → Import from file**
3. Import each JSON file from the `workflows/` folder in this order:
   - `Daily Report Generator.json`
   - `AI Processing.json`
   - `News Collection Agent.json`
   - `Weekly Report Generator.json`

### Configuring Credentials

After importing, replace all placeholder values in the workflows with your actual credentials:

| Placeholder | What to replace with |
|---|---|
| `YOUR_POSTGRES_CREDENTIAL_ID` | Your n8n PostgreSQL credential |
| `YOUR_OPENAI_CREDENTIAL_ID` | Your n8n OpenAI credential |
| `YOUR_GITHUB_PAT` | A GitHub personal access token |
| `YOUR_GUARDIAN_API_KEY` | A Guardian API key (free at open-platform.theguardian.com) |
| `YOUR_DISCORD_WEBHOOK_URL` | Your Discord channel webhook URL |
| `YOUR_SLACK_CREDENTIAL_ID` | Your n8n Slack credential |
| `YOUR_SLACK_CHANNEL_ID` | Target Slack channel ID |
| `YOUR_GMAIL_CREDENTIAL_ID` | Your n8n Gmail OAuth2 credential |
| `YOUR_EMAIL` | Recipient email address |
| `YOUR_TELEGRAM_CREDENTIAL_ID` | Your n8n Telegram credential |
| `YOUR_TELEGRAM_CHAT_ID` | Your Telegram chat/user ID |

### Activating

1. Activate `Daily Report Generator` first (it is called as a sub-workflow)
2. Activate `AI Processing` next (it also calls the Daily Report Generator)
3. Activate `News Collection Agent` last (it is the primary entry point)
4. Activate `Weekly Report Generator` independently

## News Sources

| Source | Type |
|---|---|
| TechCrunch | RSS Feed |
| Wired | RSS Feed |
| MIT Technology Review | RSS Feed |
| The Guardian (Technology) | RSS Feed + API |
| GitHub Trending | GitHub REST API |

## Expected Complexity

Data Aggregation + Workflow Automation + AI Processing

## What Defines a Strong Project

- Reliable news collection
- Accurate categorization
- High-quality summaries
- Automated report generation
- Seamless integrations
- Scalable workflow design

## Deliverable

An AI-powered research agent that automatically collects, analyzes, and distributes technology news reports.
