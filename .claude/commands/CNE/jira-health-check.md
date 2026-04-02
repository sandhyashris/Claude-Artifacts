# 7EDGE Daily Aged Cards Report

**Purpose:** Every day, check all 7EDGE JIRA workspaces for aged cards, expensive work, and dependency comments — and get a clean summary back.

---

## What to check

### 1. Aged cards
Cards that have been sitting in the same column for **more than 3 days** without movement.

- Look at the `State Change Date` field on each card
- If today's date minus that date is more than 3 days → flag it
- Include: Card ID, Title, Column, Assigned To, Days stuck

### 2. Expensive work
Cards with **GReater than 10 hours** that are still open.

- Look at the `Effort` field
- If value is 10 or above → flag it
- Include: Card ID, Title, State, Assigned To, Effort

### 3. Dependency comments
On any flagged card, check the **comments section** for these words:

- blocked by
- Flagged

If found → pull the comment text and include it in the report.

---

## Where to check

Go through **every project** in the 7EDGE  organisation:

```
https://7edge.atlassian.net/jira/software/projects
```

Skip cards in these states — they are done:
- Done
- Deployed

---

## Daily report format

Use this layout when writing the report each day:

---

### 7EDGE Card Report — [Date]

**Project: [Project Name]**

#### Aged Cards (stuck more than 7 days)

| Card | Title | Column | Assigned To | Days Stuck | Has Dependency? |
|------|-------|--------|-------------|------------|-----------------|
| #101 | Fix login bug | In Progress | Ravi K | 14 days | Yes |
| #88  | Update API docs | Review | Priya S | 9 days | No |

#### Expensive Work (10+ hours, still open)

| Card | Title | State | Assigned To | Time | Has Dependency? |
|------|-------|-------|-------------|--------|-----------------|
| #210 | Migrate database | In Progress | sandhya N | 13 | Yes |


**Summary for the day**

| Item | Count |
|------|-------|
| Aged cards | 2 |
| Expensive work items | 1 |
| Cards with dependency comments | 2 |

---