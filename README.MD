# Jobs in Sports — A Web-Scraped Database for Sports-Industry Hiring

> Initated as a capstone project for **DAMG 6210 — Database Design & Data Management** at Northeastern University.
> A solo end-to-end project: web scrapers, normalized MySQL schema, and analytical SQL views built around one question — *what does it actually take to find a job in sports?*

## At a glance

I built a relational database that ingests sports-industry job postings (and adjacent Twitter chatter) from multiple public sources, then wrote analytical SQL views on top so a job seeker can ask questions like *"which NHL teams are hiring analysts?"* or *"which leagues posted the most openings this month?"* — and get an answer in a single query.

| | |
|---|---|
| **Stack** | Python · MySQL · Jupyter · BeautifulSoup · Tweepy · MySQL Workbench |
| **Data sources** | Teamwork Online · LinkedIn · Twitter (X) API |
| **Schema** | 7 tables · 3NF · 5+ analytical views |
| **Deliverables** | Scraping notebooks · Schema (.mwb) · Project report (PDF) · Use-case doc |

## Why this project

Sports-industry job listings are scattered across general boards (LinkedIn, Indeed), specialty boards (Teamwork Online), team career pages, and informal channels like Twitter. There's no clean, structured way to compare them — so I built one.

The questions that drove the design:

- What kinds of jobs are most available across the major leagues?
- How easily can someone find data-related roles in sports specifically?
- Who could a candidate contact to network into a posting?

## Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Teamwork Online │    │    LinkedIn     │    │  Twitter (X)    │
│   BeautifulSoup │    │   BeautifulSoup │    │     Tweepy      │
└────────┬────────┘    └────────┬────────┘    └────────┬────────┘
         │                      │                      │
         └──────────────┬───────┴──────────────┬──────┘
                        │                      │
                ┌───────▼──────┐       ┌───────▼──────┐
                │ Cleaning &   │       │ Cleaning &   │
                │ De-duping    │       │ De-duping    │
                │ (pandas)     │       │ (pandas)     │
                └───────┬──────┘       └───────┬──────┘
                        │                      │
                        └──────────┬───────────┘
                                   ▼
                       ┌───────────────────────┐
                       │   MySQL — 7 tables    │
                       │ (3NF, FK constraints) │
                       └───────────┬───────────┘
                                   ▼
                       ┌───────────────────────┐
                       │  Analytical SQL views │
                       │  (NHL analysts, MA    │
                       │   listings, etc.)     │
                       └───────────────────────┘
```

## Schema

Seven tables capture both the postings side and the social/Twitter side of the data:

| Table | Purpose |
|---|---|
| `Job_Posting` | Core listings (title, company, location, salary, deadline, URL) |
| `Company_Team` | Hiring orgs with league, stadium, headquarters, founding year |
| `Job_Requirements` | Long-form requirement text, normalized away from postings |
| `Tweets` | Scraped tweets relevant to sports hiring keywords |
| `Tweet_User` | Twitter user metadata (handle, bio, follower counts) |
| `Tweet_Mentions` | Many-to-many bridge for `@mentions` in tweet bodies |
| `Sources` | Source identifier lookup for provenance tracking |

Schema designed in MySQL Workbench (`SQL Code/JobsInSports_Schema.mwb`).

## Analytical views

Wrote five+ SQL views on top of the schema so the database answers job-seeker questions directly:

- `NHL_Analysts` — every analyst-titled posting at NHL teams or the league office.
- `league_listings` — count of postings per league, ordered by volume.
- `MA_job_postings` — Massachusetts-only listings with full requirement text.
- `remote_non_league` — remote roles at non-league sports companies.
- `spring_jobs` — internships and seasonal postings tagged "Spring".

## What I learned the hard way

- **Indeed is essentially un-scrapable** without paid APIs or significant anti-bot work. After multiple weeks I cut my losses and pivoted to Teamwork Online.
- **LinkedIn moves fast on scraping defenses** — what worked for me may not work the next day; treat any LinkedIn pipeline as fragile.
- **Twitter's free API is a poor fit for hiring data** — its much too noisy. The Twitter side of the schema is preserved as a learning artifact rather than a live signal.
- **Iterating on schema design pays off** — early `ALTER` statements were rolled back into the original `CREATE` statements once stable, leaving a clean DDL that a reader can run end-to-end.

## Files of interest

```
JobsInSports/
├── Documentation/
│   ├── JobsInSports_ReportDAMG6210.pdf   # full project report
│   ├── README_FINAL.pdf                   # narrative write-up
│   └── REFERENCES.pdf                     # data sources & references
├── SQL Code/
│   ├── Table Creation Statements.sql      # 7-table DDL (3NF)
│   ├── jobsinsports_views.sql             # 5+ analytical views
│   └── JobsInSports_Schema.mwb            # MySQL Workbench ER diagram
├── Teamwork/
│   ├── Teamwork Online Scrape.ipynb       # initial scrape (BeautifulSoup)
│   └── Teamwork Online Scrape Refreshes.ipynb  # idempotent refresh script
├── Linkedin/
│   └── Linkedin Scrape.ipynb              # league-keyword-driven scraper
└── Twitter/
    └── Twitter_Scrape.ipynb               # Tweepy ingestion → tweet tables
```

## How to run

1. Stand up a MySQL 8.x instance and run `SQL Code/Table Creation Statements.sql`.
2. Apply views: `SQL Code/jobsinsports_views.sql`.
3. Open the Teamwork notebook, set your DB connection string, run cells top-to-bottom.
4. Repeat for LinkedIn and Twitter notebooks (Twitter requires Twitter Developer credentials).
5. Query a view, e.g. `SELECT * FROM NHL_Analysts;`.

## What I'd do next (Most of which are in the works)

- Replace the Twitter pipeline with an RSS / team-newsroom ingestion since the free X API has degraded materially.
- Add a `recruiter_info` table once a reliable source exists — original goal, abandoned because recruiter data is locked behind LinkedIn-quality boundaries.
- Wrap the refresh logic so the database stays current without manual notebook runs.
- Push the analytical views into a small Streamlit or Tableau dashboard for non-SQL readers.

---

Built by **Vincenzo Coppola** — aspiring data analyst in the sports industry.
[GitHub](https://github.com/VinCoppola) · [LinkedIn](https://www.linkedin.com/in/vincenzo-coppola12/)
