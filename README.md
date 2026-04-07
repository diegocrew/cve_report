# Daily CVE Reporter

An automated security intelligence tool that monitors the National Vulnerability Database (NVD) and creates GitHub Issues for newly published vulnerabilities, split by severity tier.

## How It Works

A GitHub Actions workflow runs daily at **00:00 UTC** (and can be triggered manually) and:

1. Fetches all CVEs published in the last 24 hours from the NVD API 2.0
2. Enriches each CVE with CVSS scores (v4.0 preferred, fallback to v3.1) and EPSS exploit-probability scores from FIRST.org
3. Cross-references against the CISA Known Exploited Vulnerabilities (KEV) catalog
4. Aggregates security news headlines from three RSS feeds
5. Creates **three separate GitHub Issues** — one per severity tier — assigned to the appropriate owner from `security-owners.json`

---

## Issue Structure

Three issues are opened each day (a tier is skipped if no CVEs land in that range):

| Issue | CVSS Range | Priority | Assignee | Sections |
|-------|-----------|----------|----------|----------|
| `[P0] Daily CVE Report — CRITICAL` | 9.0 – 10.0 | P0 | `diegocrew` | CVEs + CISA KEV status |
| `[P1] Daily CVE Report — HIGH` | 7.0 – 8.9 | P1 | `diegocrew` | CVEs |
| `[P2] Daily CVE Report — MEDIUM` | 0 – 6.9 | P2 | `diegocrew` | CVEs + Security News |

Each CVE line follows the format:

```
- [CVE-XXXX-XXXXX](NVD link) **CVSS Score SEVERITY (version)** | EPSS X.XX%: description (200 chars max)
```

If a daily batch produces a body exceeding GitHub's 65,536-character limit, the CVE list is automatically truncated to the last complete line and a notice is appended. Full output is always available in the workflow run logs.

---

## Routing & Labels

Labels applied automatically per issue:

- `security`, `cve`, `automated` — on every issue
- `severity-critical` / `severity-high` / `severity-medium` — tier label
- `actively-exploited` — added to the Critical issue when any CVE in the batch is in the CISA KEV catalog (also upgrades priority to `P0-CRITICAL`)
- `high-exploit-risk` — added to the Critical issue when the batch-wide maximum EPSS score ≥ 0.7

---

## Security News

The Medium issue includes the latest 5 headlines from each of:

- **Bleeping Computer**
- **The Hacker News**
- **Krebs on Security**

Each feed has a 15-second timeout; unavailable feeds show a fallback message.

---

## Data Sources

| Source | Purpose |
|--------|---------|
| [NVD API 2.0](https://services.nvd.nist.gov/rest/json/cves/2.0/) | CVE data — no API key required |
| [FIRST.org EPSS](https://api.first.org/data/v1/epss) | Exploit probability scores |
| [CISA KEV](https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json) | Actively exploited vulnerabilities |

---

## Configuration

### [`security-owners.json`](.github/security-owners.json)

Controls severity routing, assignees, labels, and thresholds:

```json
{
  "severity_routing": {
    "critical": { "cvss_range": [9.0, 10.0], "assignees": ["diegocrew"], "label": "severity-critical", "priority": "P0" },
    "high":     { "cvss_range": [7.0,  8.9], "assignees": ["diegocrew"], "label": "severity-high",     "priority": "P1" },
    "medium":   { "cvss_range": [0,    6.9], "assignees": ["diegocrew"], "label": "severity-medium",   "priority": "P2" }
  },
  "cisa_kev_override": {
    "enabled": true,
    "assignees": ["diegocrew"],
    "label": "actively-exploited",
    "priority": "P0-CRITICAL"
  },
  "epss_threshold": {
    "high_exploit_risk": { "threshold": 0.7, "addLabel": "high-exploit-risk" }
  }
}
```

To change assignees, update the `assignees` arrays and also update [`.github/CODEOWNERS`](.github/CODEOWNERS).

---

## Workflow Steps

1. **Fetch CVEs** — NVD API 2.0, 24-hour window, no auth required
2. **Fetch EPSS scores** — FIRST.org API, batched CVE IDs
3. **Build tier lists** — `jq` filters CVEs into Critical / High / Medium buckets, sorted by CVSS descending
4. **Fetch CISA KEV** — downloads full KEV catalog, cross-references batch CVEs
5. **Fetch news feeds** — RSS from three sources, 5 headlines each
6. **Compute routing metadata** — max CVSS score, max EPSS score, KEV presence flag
7. **Create GitHub Issues** — one per tier with appropriate labels, assignees, and sections; body truncation applied if needed

## Triggering

- **Scheduled**: daily at `00:00 UTC`
- **Manual**: Actions tab → **Daily CVE Report** → **Run workflow**

## Permissions

```yaml
permissions:
  contents: write
  issues: write
```

## Workflow Reference

Full implementation: [`.github/workflows/cve-daily.yml`](.github/workflows/cve-daily.yml)
