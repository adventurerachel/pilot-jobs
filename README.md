# Pilot Jobs

A simple job tracker and checklist for a freelance general aviation pilot, for use on a phone or laptop. This repository holds the web page. The data lives in the pilot's own Google account.

<p align="center">
  <img src="docs/1-jobs.png" width="23%" alt="Active jobs: confirmed and tentative">
  <img src="docs/2-checklist.png" width="23%" alt="A job's checklist with travel by flight, train or driving">
  <img src="docs/3-todo.png" width="23%" alt="To do tab across all confirmed jobs">
  <img src="docs/4-invoice.png" width="23%" alt="Creating an invoice for a job">
</p>

<sub>Screenshots use invented sample data.</sub>

## What it does

- Tracks **tentative** jobs (orange in Google Calendar) and **confirmed** jobs (green). Cancelled jobs are archived and can be restored.
- Gives each confirmed job a checklist: GAR, flight plan and handling for each leg; outbound and return travel (flight, train or driving); accommodation; and post-flight tasks.
- Calculates GAR and flight plan deadlines and sets calendar reminders.
- Fills in the Home Office GAR spreadsheet from saved people and aircraft.
- Creates invoices as PDFs, emails them to the pilot to check and forward, and prompts for overdue payments.

## How it fits together

```
index.html (this repo, on GitHub Pages)  <-->  Google Apps Script (the pilot's own copy)  -->  Calendar, Gmail, Sheets
```

This page has no data and no secrets in it. On first use each device asks for the pilot's own server address and passcode, which are stored only in that browser.

## Files

| File | Purpose |
|---|---|
| `index.html` | The app (a single file) |
| `docs/` | The screenshots shown above |
| `manifest.webmanifest`, `icon-*.png`, `apple-touch-icon.png`, `favicon.svg` | Home-screen icon and name |
| `GAR_Template.xlsx` | The GAR form that is filled in |
| `pilot-jobs-import-template.xlsx` | Template for importing jobs from a spreadsheet |

The backend code (`Code.gs`) and the full setup guide are kept separately and are not needed to host this page.

## Hosting

1. Keep all the files above in the root of the repository.
2. Go to **Settings → Pages**, set the source to the `main` branch and `/ (root)`, and save.
3. Open the address GitHub gives you, then enter the server address and passcode.

To update, upload the new `index.html` over the old one, wait a minute, then refresh. On a phone, close and reopen the page if it still looks old.

## Privacy

Job, passenger and invoice data is stored in the pilot's own Google account, never in this repository. Do not commit personal data, server addresses or passcodes here.
