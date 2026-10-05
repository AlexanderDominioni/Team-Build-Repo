# Confirm Assignment on Shared Calendar Task Specification

## Basic Information

- **Task ID:** T8
- **Task name:** Confirm Assignment on Shared Calendar
- **Task type:** Act
- **Task owner:** Lesson Scheduling Agent; the Front Desk Manager is accountable.

## 1. Task Description

T8 makes a confirmed assignment visible to the instructor and staff. When an instructor accepts (or the Front Desk Manager staffs a slot manually), T8 adds the instructor as a guest on the session's calendar event, which sends them a calendar invite, and updates the Lesson Log. The fixed rule is: the event title shows "Unassigned" or "Partially staffed" until every slot is accepted, then shows the instructors' names.

## 2. Inputs

### Input 1

- **Input name:** Accepted assignment
- **Contents and format:** Assignment Requests row with status "accepted" or "accepted — staffed manually": session ID, slot number, instructor name and email.
- **Source:** T7: Send One-Click Assignment Request (recorded response), or H2: Staff Lesson Manually (manual staffing record).

### Input 2

- **Input name:** Session calendar reference
- **Contents and format:** Lesson Log row: session ID, calendar event ID, instructors required, and accepted instructors.
- **Source:** Lesson Log tab, updated by T3: Update Lesson Record and Shared Calendar.

- **If a required input is missing or invalid:** If the session has no calendar event ID, T8 creates the event using the T3 rules before adding the instructor. If the session was cancelled after the instructor accepted, T8 makes no calendar change and sends the instructor to T4: Notify Instructor of Lesson Change.

## 3. Outputs

### Output 1

- **Output name:** Confirmed calendar event
- **Contents and format:** Shared Lesson Calendar event with the instructor added as a guest and the title and staffing status updated.
- **Next task or recipient:** Shared Lesson Calendar; the instructor receives a Google Calendar invite.
- **Complete when:** The instructor appears on the event's guest list.

### Output 2

- **Output name:** Session staffing status
- **Contents and format:** Lesson Log row updated with the accepted instructor and status "partially staffed" or "staffed."
- **Next task or recipient:** Lesson Log tab, read by T9: Compile Nightly Lesson Summaries.
- **Complete when:** The row shows the new status and the instructor's name.

## 4. Planned Tools

### Tool 1

- **Tool name:** `confirm_calendar_assignment`
- **Input:** Accepted assignment; Session calendar reference
- **Output:** Confirmed calendar event; Session staffing status
- **Implementation Route:** Web API calls; Google Calendar API event update, and Google Sheets API write of the Lesson Log tab.
- **Integration approach:** Direct integration.
- **Role in this task:** Adds the instructor to the event, updates the title and staffing status, and records the change in the Lesson Log.
- **Task timeout:** 30 seconds for one task run, including retries.
- **Maximum retries:** 1
- **Retry only when:** Google Calendar or Sheets returns a temporary error. Wait 5 seconds. Before retrying, read the event's guest list; if the instructor is already on it, record success and do not retry. Adding a guest who is already listed causes no duplicate.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record "accepted — calendar not updated" in the Lesson Log and escalate to the Front Desk Manager. The acceptance stands, and T9 still includes the lesson in the instructor's nightly summary.
