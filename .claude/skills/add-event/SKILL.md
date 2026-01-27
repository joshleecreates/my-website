---
name: add-event
description: "Add a new speaking event to the website"
---

# Add Event Skill

Add a new speaking event to the website. The user will provide a conference name.

**Input:** $ARGUMENTS (conference name, e.g., "KubeCon EU 2026")

## Instructions

### Step 1: Web Research

Search the web for the conference "$ARGUMENTS" to find:
- Official conference website URL
- Conference dates
- Location (city and state/country)
- Speaker schedule or session list

Use WebSearch to find the conference, then WebFetch to get details from the official site.

### Step 2: Find the User's Talk

Search for "Josh Lee" in the conference schedule/speakers page to identify which talk(s) the user is presenting. Look for:
- Session titles
- Talk descriptions
- Speaker listings

If you cannot find "Josh Lee" in the schedule, ask the user for their talk title.

### Step 3: Match Against Existing Talks

Read the existing event files to extract all known talk titles:
```
Grep for "talks:" in content/events/ with context to get the talk names
```

Compare the found talk title(s) against existing talks. Use case-insensitive matching and handle minor variations. Known talks include:
- "Modern Application Debugging: An Intro to OpenTelemetry"
- "DevOps is a Foreign Language"
- "Where's the Auto in Auto-Instrumentation?"
- "The OpenTelemetry Hero's Journey"
- "O11y in One: ClickHouse as a Unified Telemetry Database"
- "Open-Source Observability from Scratch"
- "My NixOS-Powered Homelab"
- And others in the event files

If a match is found, use the existing talk title exactly to maintain consistency.

### Step 4: Confirm Details with User

Use AskUserQuestion to confirm all details before creating the file:

Present:
- Conference name
- Date (first day of conference)
- Location
- Event URL
- Talk title(s) - note if it's an existing talk or a new one

Ask the user to confirm or correct any details.

### Step 5: Create the Event File

Create the event file at: `content/events/YYYY-MM-DD-event-slug.md`

File naming:
- Use the first day of the conference for the date prefix
- Create a slug from the conference name (lowercase, hyphenated) + year
- Example: `2026-04-01-kubecon-eu.md`

File format:
```markdown
---
title: "Conference Name"
date: "YYYY-MM-DDT00:00:00Z"
slug: event-slug-year
params:
  location: "City, State/Country"
  eventDate: YYYY-MM-DDT12:00:00
  eventURL: "https://conference-url.com"
  talks:
  - "Talk Title"
---
```

Notes:
- The `date` field uses midnight UTC for the conference start date
- The `eventDate` uses noon (12:00:00) on the same day
- For US locations, use "City, State" format (e.g., "Salt Lake City, Utah")
- For international locations, use "City, Country" format (e.g., "Amsterdam, Netherlands")
- If multiple talks, list each on its own line under `talks:`

### Step 6: Confirm Creation

After creating the file, let the user know the event has been added and provide the file path.
