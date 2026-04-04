# Daily CVE Reporter

An automated security intelligence tool that monitors the National Vulnerability Database (NVD) and automatically creates GitHub Issues with newly published vulnerabilities.

## How It Works

This repository operates as an automated security monitoring system using GitHub Actions.

- **Automation**: A scheduled workflow runs daily at 00:00 UTC
- **Data Collection**: The system queries the NVD API 2.0 to retrieve all vulnerabilities published within the last 24 hours
- **Issue Creation**: For every daily batch with results, the system automatically opens a new GitHub Issue containing a curated list of CVE IDs and their descriptions

## Workflow Details

### Trigger Schedule

The workflow is triggered in two ways:

1. **Scheduled**: Runs automatically every day at midnight UTC (`cron: '0 0 * * *'`)
2. **Manual**: Can be manually triggered from the Actions tab at any time (`workflow_dispatch`)

### API Endpoint

- **URL**: `https://services.nvd.nist.gov/rest/json/cves/2.0/`
- **Method**: GET with date range parameters (`pubStartDate` and `pubEndDate`)
- **Date Range**: Last 24 hours (calculated from current UTC time)

### Data Processing

The workflow:

1. Calculates the date range for the previous 24 hours using `date -u -d "yesterday"`
2. Queries the NVD API to fetch vulnerabilities in that timeframe
3. Counts the total results with `jq '.totalResults'`
4. Parses each vulnerability to extract CVE ID and description using `jq`
5. Formats each CVE as a markdown link to the NVD detail page
6. Creates a GitHub Issue if vulnerabilities are found (COUNT > 0)

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

### Commands Used

- **curl**: HTTP requests to fetch CVE data from NVD API 2.0
- **jq**: JSON parsing to extract CVE information and format output
- **date**: Dynamic timestamp calculation for 24-hour window

### Permissions Required

The GitHub Action requires these permissions:

- `contents: write` - To access and checkout the repository
- `issues: write` - To create new GitHub Issues

### Important Notes

- **No API Key Required**: The workflow uses unauthenticated requests to the NVD API
- **Rate Limiting**: Unauthenticated requests are subject to NVD's rate limits
- **Output Format**: Each CVE is formatted as `- [CVE-ID](nvd_link): Description`
- **Daily Schedule**: Timezone is UTC (00:00 UTC = 8:00 PM ET / 7:00 PM CT, depending on daylight saving)

## Verification Against Workflow

This README has been verified against the workflow file [`.github/workflows/cve-daily.yml`](.github/workflows/cve-daily.yml) and accurately reflects:

- Daily schedule at 00:00 UTC
- Manual workflow dispatch option
- NVD API 2.0 queries with date range
- GitHub Issue creation on findings
- No API key authentication required
- 24-hour lookback window calculation