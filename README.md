# How to Scrape Trucking and Logistics Driver Jobs: TDR Jobs (Spain) and CDL Jobs (US) in One Schema

> **Run it now, no code: [trucking-logistics-jobs-scraper on the Apify Store](https://apify.com/bibian/trucking-logistics-jobs-scraper)** &nbsp;|&nbsp; $3.00 per 1,000 postings &nbsp;|&nbsp; no browser, no login

TDR Jobs (Spain) and CDL Jobs (United States) are two of the main sector-specific boards for truck and logistics drivers, each with no public export and no official API. This guide shows how to get both as one clean, deduplicated dataset: CDL driver and trucking job postings, with location, remote policy, salary when disclosed, and the source board, with a free Apify Actor, from the console or from code.

Searches this page answers: trucking jobs scraper, CDL driver jobs API, TDR Jobs scraper, CDL Jobs scraper, logistics driver job postings.

## What you get from the Trucking & Logistics Jobs scraper

- Every CDL driver and logistics job posting from TDR Jobs (Spain) and CDL Jobs (United States), in one run, on one unified schema.
- Location (`city`, `region`, `country`), remote policy, employment type and dates, normalized the same way across both sources.
- Salary (`salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod`) when the board discloses one — common on CDL Jobs, rare on TDR Jobs.
- `source` and `country` on every row, so you can filter or split by board or market without a second scraper.

Export the dataset as JSON, CSV or Excel, or read it through the Apify API. A real sample (five results, no personal data, from TDR Jobs) is in [`sample-output.json`](sample-output.json).

## Output fields

| Field | What it is |
|---|---|
| `id`, `url` | Stable identifier (`<source>:<native id>`) and public link |
| `title` | Job title as published |
| `companyName`, `companyUrl` | Employer name and page, when disclosed; `null` when hidden (`confidentialCompany: true`). TDR Jobs never names the real employer. |
| `locationLabel`, `city`, `region`, `postalCode`, `country` | Where the job is based (`country` is an ISO 3166-1 alpha-2 code: `ES` or `US`) |
| `remote` | `remote`, `hybrid`, `onsite` or `unknown` (driving jobs are physical: expect mostly `onsite`) |
| `employmentType` | `full_time`, `part_time`, `contract`, `freelance`, `internship`, `apprenticeship`, `temporary` or `unknown` |
| `salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod` | Salary range, only when the board discloses one |
| `datePosted`, `validThrough` | Publication and expiry dates, ISO 8601 |
| `description` | Plain-text description, only when `includeDescriptions` is on. No personal data. |
| `experienceLevel`, `industry`, `skills` | As named by the site, when available |
| `source`, `sourceId` | Host name of the source site and its native identifier |
| `scrapedAt` | Extraction timestamp, ISO 8601 |

## Use cases

- **Cover 2 countries with one integration.** Query Spain and the United States trucking job markets without maintaining 2 separate scrapers or 2 schemas.
- **Build a daily job alert.** Schedule a run every morning with `publishedWithinDays: 1` and `onlyNew` on, and send only the new postings to Slack, e-mail or a Google Sheet.
- **Track driver pay.** Group `salaryMin`/`salaryMax` by `source`, `region` or `employmentType` where the board shows a number.
- **Feed a niche job board.** Use the structured fields and the canonical `url` to power a trucking and logistics board or newsletter.

## Input

| Field | What it does |
|---|---|
| `keywords` | Job title or trade, for example `CDL driver` or `conductor`. Empty means every posting. |
| `location` | City, state or region, for example `Texas` or `Madrid`. Empty means all locations. |
| `countries` | Keep only the source covering these countries: `ES`, `US`. Defaults to both. |
| `sources` | Which of the 2 job boards to query: `tdrjobs`, `cdljobs`. Defaults to both. |
| `publishedWithinDays` | Keep only postings published in the last N days. Use `1` for a daily monitoring run. |
| `remote` | Keep only postings with this remote policy: `any`, `remote`, `hybrid` or `onsite`. |
| `maxItems` | The run stops once this many postings are saved, across both sources combined (default 50). |
| `includeDescriptions` | Add the job description as plain text (default on). |
| `onlyNew` | For scheduled runs: skip postings an earlier run of this same search already delivered. |

Example input:

```json
{
  "keywords": "CDL driver",
  "countries": ["US"],
  "publishedWithinDays": 7,
  "maxItems": 100,
  "includeDescriptions": false
}
```

## Four ways to use it

### 1. Apify console, no code

1. Open [the Actor page](https://apify.com/bibian/trucking-logistics-jobs-scraper) and click **Try for free** (a free Apify account is enough).
2. Fill in the input form (keywords, countries, filters and the maximum number of results) and click **Start**.
3. When the run finishes, open the **Output** tab, check the table, then **Export** as CSV, Excel or JSON.
4. For a daily alert, open **Schedules**, create a daily schedule for the Actor and, optionally, add an integration that sends new results to Slack, e-mail or Google Sheets.

### 2. Apify API with JavaScript

```bash
npm install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
node examples/run.js
```

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });

const run = await client.actor('bibian/trucking-logistics-jobs-scraper').call({
    "keywords": "CDL driver",
    "countries": ["US"],
    "publishedWithinDays": 7,
    "maxItems": 100,
    "includeDescriptions": false
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
for (const item of items) {
    console.log(`${item.title} | ${item.companyName ?? 'confidential'} | ${item.locationLabel} | ${item.source}`);
}
```

The complete file is [`examples/run.js`](examples/run.js) (ES module, Node 18 or later).

### 3. Apify API with Python

```bash
pip install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
python examples/run.py
```

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])

run = client.actor("bibian/trucking-logistics-jobs-scraper").call(run_input={
    "keywords": "CDL driver",
    "countries": ["US"],
    "publishedWithinDays": 7,
    "maxItems": 100,
    "includeDescriptions": False,
})

for item in client.dataset(run.default_dataset_id).iterate_items():
    print(f"{item['title']} | {item.get('companyName') or 'confidential'} | {item.get('locationLabel')} | {item.get('source')}")
```

The complete file is [`examples/run.py`](examples/run.py). Never put your token in the code: read it from the `APIFY_TOKEN` environment variable as above.

### 4. Monitor new postings daily and connect your tools

1. **Schedule it.** Save a task with your filters, tick **Only new results** in the input, and schedule it daily with Apify Schedules: each run returns, and charges, only the postings earlier runs did not already deliver.
2. **Push the results where you work.** Add an Apify integration or a webhook to the task: Google Sheets, Slack, Make, Zapier, n8n or your own endpoint.
3. **Call it from your stack.** The API and the JavaScript and Python clients return the dataset as JSON, CSV or Excel.

## Pricing

Pay per result: **$3.00 per 1,000 postings** (that is $0.003 per posting), plus a tiny start fee. You pay only for the results saved to the dataset, deduplicated across both sources before being counted or charged. Apify's free plan includes monthly credits, enough to try it on a few hundred results. Prices are those shown on the [Store page](https://apify.com/bibian/trucking-logistics-jobs-scraper) when this guide was written; the Store page is the reference.

## Limits and honest notes

- TDR Jobs never discloses the real employer: every posting names the platform itself as `hiringOrganization`, so `companyName` is always `null` and `confidentialCompany` is always `true` for that source. This is a limit of the source, not a bug.
- Salary is only as structured as the board makes it: common on CDL Jobs, rare on TDR Jobs.
- The dataset contains job postings only. Recruiter contact details are never exported.
- This is an unofficial tool, not affiliated with TDR Jobs or CDL Jobs. Check the site terms and the data protection rules that apply to your use.

## FAQ

**Does it need a browser or a proxy?**
No. The Actor reads the data directly, so runs are fast and cheap.

**How do I only get new postings every day?**
Set `publishedWithinDays` to `1` and schedule the Actor once a day with `onlyNew` on. Results come newest first, so a date-limited run stops early.

**Can I query just one country?**
Yes, set `countries` to `["ES"]` or `["US"]`, or `sources` to `["tdrjobs"]` or `["cdljobs"]`.

**Can I get the data in Excel or Google Sheets?**
Yes. Export the dataset as CSV, Excel or JSON from the Apify console, or connect the Apify Google Sheets integration.

## Try it

**[Open trucking-logistics-jobs-scraper on the Apify Store](https://apify.com/bibian/trucking-logistics-jobs-scraper)**

The code in `examples/` is released under the MIT license (see [LICENSE](LICENSE)). The data belongs to its original publishers.
