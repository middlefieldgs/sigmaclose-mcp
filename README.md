# SigmaClose MCP server: QuickBooks Online, every company at once

A remote MCP server that gives Claude, ChatGPT, Claude Code, Codex and other assistants **consolidated statements across all your QuickBooks Online companies, with intercompany amounts taken out**: profit and loss, balance sheet and cash flow, with a column per company, an Eliminations column and the combined total. Single-company QuickBooks connectors read one company per sign-in; this one hands the assistant numbers that are already combined.

- Server: `https://sigmaclose.com/mcp` (streamable HTTP, OAuth sign-in)
- Docs, every tool with a worked example: https://sigmaclose.com/developers
- For assistants: https://sigmaclose.com/llms.txt
- Guide: https://sigmaclose.com/guides/quickbooks-mcp-multiple-companies

It never changes your books. Official reports: an assistant can hand over the same files the website makes, with get_report_file: profit and loss, balance sheet or cash flow as PDF, Excel or CSV, and the report pack as a PDF. It returns a download link that works for 10 minutes, for that one file, with no sign-in, so the person can click it in the chat.

This repository holds no code. The server is hosted at sigmaclose.com; this page is here so it can be found.

## Connect

**Claude Code**

```
claude mcp add --scope user --transport http sigmaclose https://sigmaclose.com/mcp
```

Run it in your terminal. It opens SigmaClose in your browser to sign in and approve.

**Codex**

```
codex mcp add sigmaclose --url https://sigmaclose.com/mcp
```

Then run codex mcp login sigmaclose to sign in and approve.

**Claude, ChatGPT and others**

```
https://sigmaclose.com/mcp
```

Add it as a custom connector (in Settings, under Connectors). It sends you to SigmaClose to sign in and approve.

## Try it without an account

To try it without an account, use the public demo key. It reads made-up sample data (the Northwind group: a holding company, a shop and a property company) and nothing else. The key is `sck_demo_northwind_sample_data_read_only`.

```
curl -H "Authorization: Bearer sck_demo_northwind_sample_data_read_only" "https://sigmaclose.com/api/v1/get_overview"
```

No key and no command line needed: every tool also answers from the sample data at a plain address, which an assistant that can only open web pages can read. For example https://sigmaclose.com/api/v1/demo/get_overview

There is also a read-only REST API: the same tools as GET requests, with a personal key. OpenAPI: https://sigmaclose.com/api/v1/openapi.json

## Tools

- `get_overview`: The group's companies, connection and sync status, months with data, and elimination pairs. Usually the first call.
- `get_statement`: A consolidated statement. by='company': a column per company, Eliminations and Consolidated. by='month': months side by side, consolidated, with a Total column for profit and loss and cash flow. Without 'period' it is one month (by company) or a from/to range (by month); month defaults to the latest. With 'period': month, quarter, ytd (fiscal year to date), year (full fiscal year), t12 (last 12 months) or custom (from/to). 'at' picks which: a month 'YYYY-MM' for month, ytd and t12; '2026-Q3' for quarter; '2026' for year. A period is cut to the months that have data and says so in notes.
- `get_report_file`: A download link for an official report, the same file the website makes: a statement (report pl, bs or cf) as PDF, Excel (xlsx) or CSV, or the report pack (report='pack': cover, summary, then profit and loss, balance sheet and cash flow by company, as one PDF). Use it when the person asks for their statements or a report to share. Takes the same period arguments as get_statement, except that with no month named it ends at the latest complete month (the pack too, which takes period and at, or month), so a file never shows a month still going. The link works for a few minutes with no sign-in, for that one file.
- `explain_line`: Break one line of a statement down by company and eliminations; for a total, also lists the accounts in it.
- `compare_periods`: Consolidated figures for two months side by side with the change and percentage change per line.
- `month_end_checks`: What needs attention before trusting a month: elimination pairs that don't net to zero, balance sheet balance, cash flow vs bank, unchecked new accounts, stale or dropped connections.
- `suggest_elimination_pairs`: Find likely intercompany elimination pairs: an account in one company and an account in another that record the same money from opposite sides (investments, loans, contributions, fees). Uses account names and whether the balances cancel out month by month. Read-only: returns a confirm_url per pair for the person to open and approve.
- `check_elimination_pair`: Test a pair you picked yourself: how the two accounts compare in every month, and a confirm_url for the person to approve it. Read-only. Takes company and account names exactly as get_statement returns them.
- `explain_elimination_pair`: For a pair already set up (pair_id from get_overview): both accounts month by month, the difference each month, and the first month they stopped cancelling out: why a pair doesn't net to zero.
- `refresh_from_quickbooks`: Pull the latest monthly numbers from QuickBooks for every connected company, like the Refresh button. Only reads QuickBooks, never changes the books; it updates the totals SigmaClose holds. Needs an editor or admin. On the Free plan each company's pull counts; a company pulled in the last few minutes is left alone and not counted. Says which companies updated, which are out of free refreshes (with the upgrade link) and which failed.

## Price and limits

$20 a month or $200 a year per group, any number of companies and your whole team. It's free to try, with 5 refreshes per QuickBooks company and no card. Reading, the report files and agent access are free on every plan; only refreshing from QuickBooks is counted on Free.

SigmaClose works with QuickBooks Online only, in US dollars only. It has no budgets, forecasts or custom report layouts, and no accountant view across several client groups yet.

Security: https://sigmaclose.com/security. Made by PerfSpot LLC.
