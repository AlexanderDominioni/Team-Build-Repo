# Hosting plan

Written in plain words with the BUS 4498 Agent Builder Kit. Our AI assistant turns it into a GitHub Pages website and a Supabase backend, following `runtime/hosting-rules.md`. Where this plan and our task specifications disagree, this plan wins.

## Our project

- GitHub repository: https://github.com/AlexanderDominioni/Team-Build-Repo
- Our agent’s folder: `our_team_agent` (our plans are in `our_team_agent/runtime/`)
- Supabase project: https://audjjpucbtbtyqortmlv.supabase.co
- Owner of the repository and the Supabase project: Alex Dominioni
- Teammate: Aidan Jones, a collaborator on GitHub and a Developer in the Supabase organization
- We say “I’m working on it” and “pushed” in: Our group chat. One of us works at a time.

## What our agent does

The Lesson Scheduling Agent reads FareHarbor booking emails for the surf school, keeps a shared Google Calendar up to date, and asks available instructors to take each lesson. Each evening it emails every instructor a summary of their lessons for the next day.

## Our tasks

Use the tasks in our task specifications: `our_team_agent/agent/workflow-of-tasks.md` and `our_team_agent/agent/task-specs/`.

## Who can see or change what

The Front Desk Manager signs in and can see every lesson, assignment request, exception, and lesson learned. Only the Front Desk Manager can resolve an unreadable booking, staff a lesson by hand, approve a third lesson for an instructor that day, change a limit, switch off a lesson learned, or pause the agent. Instructors do not sign in: each one sees only their own requests (through single-use Accept and Decline links), the Shared Lesson Calendar, and their own nightly summary, and each one keeps their own row of the Instructor Availability Sheet current. Customer names, phone numbers, and emails are visible only to the Front Desk Manager and never reach an instructor or the calendar. Customers and website visitors see nothing except the page that confirms an instructor’s click.

## External tools and applications

| # | Tool or application | What the agent uses it for | What it does with it | Most sensitive data it touches | How it connects |
|---|---|---|---|---|---|
| 1 | Gmail (booking inbox) | Finds new FareHarbor booking, cancellation, and reschedule emails and labels each one once it is processed | Reads from it | Confidential | A sign-in (OAuth) |
| 2 | Gmail (scheduling account) | Sends assignment requests, lesson-change notices, nightly summaries, and alerts to the Front Desk Manager | Sends with it | Internal | A sign-in (OAuth) |
| 3 | Google Calendar (Shared Lesson Calendar) | Creates, moves, and cancels lesson events and adds instructors who accept | Writes to it | Internal | A sign-in (OAuth) |
| 4 | Google Sheets (Instructor Availability Sheet) | Reads each instructor’s available times and preferences when a lesson needs staffing | Reads from it | Internal | A sign-in (OAuth) |

## Files people upload

None.
