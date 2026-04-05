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

Each report includes headlines from three major security news sources:

- **Bleeping Computer** - Breaking security news and vulnerability alerts
- **The Hacker News** - Latest cybersecurity stories and threat intelligence
- **Krebs on Security** - Investigative cybersecurity reporting and threat analysis

---

## Issue Format

When CVEs are detected, a comprehensive GitHub Issue is created with:

1. **Summary**: Total count of new CVEs
2. **CVE List**: Each CVE with EPSS score and links
3. **CISA KEV Status**: Flags any actively exploited vulnerabilities
4. **Security News**: Recent relevant headlines
5. **Automatic Labels**: `security`, `cve`, `automated`

1. **Scheduled**: Runs automatically every day at midnight UTC (`cron: '0 0 * * *'`)
- **Manual**: Can be manually triggered from the Actions tab at any time (`workflow_dispatch`)

### NVD API Configuration

- **URL**: `https://services.nvd.nist.gov/rest/json/cves/2.0/`
- **Method**: GET with date range parameters (`pubStartDate` and `pubEndDate`)
- **Date Range**: Last 24 hours (calculated as previous UTC day to current UTC time)
- **Authentication**: Uses `NVD_API_KEY` from GitHub secrets (recommended for higher rate limits; optional for public access)

## Data Processing

The workflow executes multiple enrichment steps:

1. **NVD Fetch**: Queries NVD API 2.0 for the last 24 hours of CVEs and stores raw JSON response
2. **EPSS Enrichment**: Fetches EPSS (Exploit Probability Scoring System) scores from FIRST.org for all retrieved CVEs
3. **CVE Scoring & Sorting**: Merges CVSS metrics (v4.0 or v3.1) with EPSS scores and sorts by severity
4. **CISA KEV Cross-Reference**: Fetches the CISA Known Exploited Vulnerabilities catalog and flags any CVEs currently known to be exploited
5. **News Aggregation**: Parses the latest 5 headlines from RSS feeds (Bleeping Computer, The Hacker News, Krebs on Security)
6. **GitHub IExecution Steps

1. **Date Range Calculation**: Computes start/end times for the previous 24 hours in UTC format
2. **NVD API Query**: Fetches all vulnerabilities published in that window with `pubStartDate` and `pubEndDate` filters
3. **Count Verification**: Extracts total CVE count using `jq '.totalResults'`; pipeline only proceeds if COUNT > 0
4. **EPSS Score Fetching**: Calls FIRST.org EPSS API with comma-separated CVE IDs (up to ~7000 chars to avoid URL limits)
5. **Data Merging**: Uses `jq` to combine NVD CVSS metrics with EPSS scores into a single enriched dataset
6. **Sorting**: Sorts all CVEs by CVSS score (descending), preferring v4.0 scores over v3.1
7. **Description Truncation**: Shortens CVE descriptions to 200 characters with "..." suffix for readability
8. **CISA KEV Lookup**: Cross-references each CVE against the CISA known exploited vulnerabilities list
9. **News Collection**: Fetches latest headlines (max 5 per source) with 15-second timeout per feed
10. **Issue Assembly & Creation**: Constructs markdown-formatted issue body with all sections and creates it in the repository with labels `security`, `cve`, `automated`
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

The GitHub Action requirecheckout the repository and access workflow files
- `issues: write` - To create new GitHub Issues with vulnerability report
- `contents: write` - To access and checkout the repository
- `issues: write` - To create new GitHub Issues

### API Key Optional**: NVD_API_KEY (from `secrets.NVD_API_KEY`) is optional. Public access works but has higher rate limits with an API key
- **EPSS Scores**: Probability (0-100%, converted from 0-1 decimal) that a CVE is actively exploited; higher percentages indicate greater exploitation likelihood
- **CVSS Versions**: Pipeline prioritizes CVSS v4.0 scores but falls back to v3.1 if v4.0 is unavailable
- **CISA KEV**: Flags vulnerabilities actively exploited in the wild - these should be prioritized for patching
- **News Feed Timeouts**: Each RSS feed has a 15-second timeout to prevent pipeline delays from slow feed servers
- **Description Limits**: CVE descriptions are capped at 200 characters to keep issues readable
- **Output Format**: Each CVE includes NVD link, CVSS + severity + version, EPSS percentage, and truncated description
- **Issue Labels**: Automatically tagged with `security`, `cve`, and `automated` for easy searching and filtering
- **Daily Schedule**: Runs at 00:00 UTC (equivalent to 8:00 PM ET or 7:00 PM CT depending on daylight saving time
- **Issue Labels**: Automatically tagged with `security`, `cve`, and `automated` for easy filtering

## Security Team Assignment & Routing

The repository uses intelligent routing to automatically assign CVE reports to the appropriate security team members based on severity levels.

### Team Structure

Security team members are assigned by CVSS severity score:

| CVSS Score | Severity | Assigned To | Priority |
|-----------|----------|------------|----------|
| 9.0 - 10.0 | Critical | `diegocrew` | P0 |
| 6.0 - 8.9 | High | `jonescrew` | P1 |
| 0 - 5.9 | Medium/Low | `alanitacrew` | P2 |
| **CISA KEV** | **Actively Exploited** | **`diegocrew`** | **P0-CRITICAL** |

### Configuration Files

#### **[`.github/security-owners.json`](.github/security-owners.json)**
Defines severity-based routing rules and team assignments. Each CVE is automatically routed based on:
- **CVSS Score**: Primary routing mechanism (Critical/High/Medium/Low)
- **CISA KEV Status**: Override routing - actively exploited vulnerabilities always route to the incident response team
- **EPSS Threshold**: High exploit probability (>70%) is flagged with additional labels

**Example structure:**
```json
{
  "severity_routing": {
    "critical": {
      "cvss_range": [9.0, 10.0],
      "assignees": ["diegocrew"],
      "label": "severity-critical",
      "priority": "P0"
    },
    "cisa_kev_override": {
      "enabled": true,
      "assignees": ["diegocrew"],
      "label": "actively-exploited",
      "priority": "P0-CRITICAL"
    }
  }
}
```

#### **[`.github/CODEOWNERS`](.github/CODEOWNERS)**
Maintains the security team roster and serves as documentation for the team structure. Specifies code ownership for security-critical files:
- Workflow files and configurations → `diegocrew` (primary owner)
- Security owner configurations → `diegocrew`
- Repository documentation → Security team

The CODEOWNERS file is referenced in GitHub for PR review requirements.

### Automated Assignment Workflow

When a daily CVE report is created:

1. **Parse Security Config**: The workflow reads `security-owners.json`
2. **Extract CVSS Scores**: Retrieves CVSS metrics from each CVE in the NVD response
3. **Route Assignment**: Matches CVSS range to appropriate team member
4. **Check CISA KEV**: Red-flags actively exploited CVEs for incident response
5. **Assign Issue**: Automatically assigns the GitHub Issue to the determined team member
6. **Apply Labels**: Adds severity labels (`severity-critical`, `severity-high`, etc.)
7. **Create PR**: Creates a security remediation PR assigned to the same team member for tracking

### How to Update Team Assignments

To modify team assignments, edit [`.github/security-owners.json`](.github/security-owners.json):

```json
"critical": {
  "cvss_range": [9.0, 10.0],
  "assignees": ["new-team-member"],  // Update username here
  "label": "severity-critical",
  "priority": "P0"
}
```

Then update [`.github/CODEOWNERS`](.github/CODEOWNERS) to reflect the change:
```
.github/ @new-team-member
```

## Workflow File Reference

For complete implementation details, see [`.github/workflows/cve-daily.yml`](.github/workflows/cve-daily.yml)

This workflow includes:
- Daily schedule at 00:00 UTC with manual trigger support
- NVD API 2.0 integration with 24-hour lookback window
- EPSS score fetching from FIRST.org
- CISA Known Exploited Vulnerabilities cross-reference
- RSS feed parsing from three security news sources
- Automated GitHub Issue creation with comprehensive report formatting
- Error handling for missing data and feed unavailability
- No API key authentication required
- 24-hour lookback window calculation