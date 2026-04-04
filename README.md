# Daily CVE Reporter

An automated security intelligence tool that monitors the National Vulnerability Database (NVD) and automatically creates GitHub Issues with newly published vulnerabilities.

## How It Works

This repository operates as an automated security monitoring system using GitHub Actions.

- **Automation**: A scheduled workflow runs daily at 00:00 UTC
- **Data Collection**: The system queries the NVD API 2.0 to retrieve all vulnerabilities published within the last 24 hours
- **Issue Creation**: For every daily batch with results, the system automatically opens a new GitHub Issue containing a curated list of CVE IDs and their descriptions

## New CVEs (with EPSS Risk Scores)

The workflow enriches each CVE with:

- **EPSS Score**: Exploit Probability Scoring System - a 0-1 probability score indicating the likelihood a CVE is being exploited in the wild (sourced directly from NVD API 2.0)
- **Direct link** to the NVD detail page for each CVE
- **Description** from the official CVE record

## CISA Known Exploited Vulnerabilities

The workflow cross-references all new CVEs against the CISA Known Exploited Vulnerabilities (KEV) catalog, which tracks vulnerabilities actively exploited by attackers. This is updated daily by CISA.

## Recent Security News

Each report includes headlines from major security news sources:

- **Bleeping Computer** - Breaking security news and vulnerability alerts
- **The Hacker News** - Latest cybersecurity stories and threat intelligence

---

## Issue Format

When CVEs are detected, a comprehensive GitHub Issue is created with:

1. **Summary**: Total count of new CVEs
2. **CVE List**: Each CVE with EPSS score and links
3. **CISA KEV Status**: Flags any actively exploited vulnerabilities
4. **Security News**: Recent relevant headlines
5. **Automatic Labels**: `security`, `cve`, `automated`

1. **Scheduled**: Runs automatically every day at midnight UTC (`cron: '0 0 * * *'`)
2. **Manual**: Can be manually triggered from the Actions tab at any time (`workflow_dispatch`)

### API Endpoint

- **URL**: `https://services.nvd.nist.gov/rest/json/cves/2.0/`
- **Method**: GET with date range parameters (`pubStartDate` and `pubEndDate`)
- **Date Range**: Last 24 hours (calculated from current UTC time)

## Data Processing

The workflow executes multiple enrichment steps:

1. **NVD Fetch**: Queries NVD API 2.0 for the last 24 hours of CVEs and extracts EPSS scores
2. **CISA KEV Check**: Fetches the CISA Known Exploited Vulnerabilities database and cross-references
3. **News Aggregation**: Parses RSS feeds from Bleeping Computer and The Hacker News
4. **Data Integration**: Combines all sources into a single comprehensive GitHub Issue

### Pipeline Steps

1. Calculates the date range for the previous 24 hours using `date -u -d "yesterday"`
2. Queries the NVD API to fetch vulnerabilities in that timeframe
3. Extracts EPSS scores for exploitation probability
4. Counts the total results with `jq '.totalResults'`
5. Fetches CISA Known Exploited Vulnerabilities catalog
6. Cross-references new CVEs against known exploits
7. Aggregates security news headlines from RSS feeds
8. Creates a single GitHub Issue with all enriched data

## Getting Started

### Manual Trigger

To run the report immediately without waiting for the daily schedule:

1. Navigate to the **Actions** tab
2. Select **Daily CVE Report** from the sidebar
3. Click **Run workflow**

### Issue Creation

When the workflow runs and finds new CVEs:

- **Title Format**: `Daily CVE Report: YYYY-MM-DD`
- **Body Content**: List of CVEs with links to their NVD detail pages
- **Condition**: Only creates an issue if COUNT > 0

## Technical Details

### Commands and Tools Used

- **curl**: HTTP requests to fetch data from:
  - NVD API 2.0 (`https://services.nvd.nist.gov/rest/json/cves/2.0/`)
  - CISA KEV Database (`https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json`)
  - RSS Feeds (Bleeping Computer, The Hacker News)

- **jq**: JSON parsing and extraction of:
  - CVE IDs and descriptions
  - EPSS scores and metrics
  - CISA vulnerability IDs

- **grep/sed**: Text processing for RSS feed parsing and cross-referencing

- **date**: Dynamic timestamp calculation for 24-hour window

### Permissions Required

The GitHub Action requires these permissions:

- `contents: write` - To access and checkout the repository
- `issues: write` - To create new GitHub Issues

### Important Notes

- **No API Key Required**: All data sources (NVD, CISA, RSS) are publicly accessible
- **EPSS Scores**: Probability (0-1) that a CVE is actively exploited; values >0.5 indicate higher risk
- **CISA KEV**: Tracks vulnerabilities known to be actively exploited - highest priority for patching
- **Rate Limiting**: Unauthenticated requests are subject to rate limits; consider adding API keys for higher quotas
- **Output Format**: Each CVE includes link, description, and exploitation risk metrics
- **Issue Labels**: Automatically tagged with `security`, `cve`, and `automated` for easy filtering
- **Daily Schedule**: Timezone is UTC (00:00 UTC = 8:00 PM ET / 7:00 PM CT, depending on daylight saving)

## Verification Against Workflow

This README has been verified against the workflow file [`.github/workflows/cve-daily.yml`](.github/workflows/cve-daily.yml) and accurately reflects:

- Daily schedule at 00:00 UTC
- Manual workflow dispatch option
- NVD API 2.0 queries with date range
- GitHub Issue creation on findings
- No API key authentication required
- 24-hour lookback window calculation