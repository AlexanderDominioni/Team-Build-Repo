# Update Lesson Record and Shared Calendar Task Specification

## Basic Information

- **Task ID:** T3
- **Task name:** Update Lesson Record and Shared Calendar
- **Task type:** Act
- **Task owner:** Lesson Scheduling Agent; the Front Desk Manager is accountable.

## 1. Task Description

T3 keeps the Lesson Log and Shared Lesson Calendar in step with FareHarbor so instructors and staff can see lessons as soon as they are booked. It applies fixed rules by notification type:

- **New:** A private lesson creates its own session. An open group booking joins the session with the same lesson type, date, and start time, or creates one if none exists. Session headcount and instructors required (session headcount divided by 5, rounded up) are recalculated.
- **Cancelled:** The booking is removed from its session. If the session is now empty, the session and calendar event are cancelled.
- **Rescheduled:** The booking is removed from its old session and added to the session for the new date and time, using the rules above.

T3 then compares instructors required with instructors already assigned. If assigned instructors exceed the new requirement, T3 releases the extra instructors (those not yet accepted first, then the most recently accepted). If more are needed, it records the number of open slots.

## 2. Inputs

### Input 1

- **Input name:** Booking details record
- **Contents and format:** Structured record: notification type, booking number, contact ID, lesson type and category, date, start and end time, headcount, student levels and ages, customer contact, and instructors required.
- **Source:** T2: Extract Booking Details, or H1: Resolve Unreadable Booking Email (corrected booking details record).

### Input 2

- **Input name:** Existing session records
- **Contents and format:** Lesson Log rows: session ID, lesson type and category, date, start and end time, booking numbers, session headcount, instructors required, assigned instructors with status (requested, accepted, declined, expired), reassignment count, and calendar event ID.
- **Source:** Lesson Log tab of the Lesson Scheduling workbook.

- **If a required input is missing or invalid:** If a cancellation or reschedule refers to a booking number not found in the Lesson Log, T3 makes no changes and escalates to the Front Desk Manager through the Run Log tab with the booking number.

## 3. Outputs

### Output 1

- **Output name:** Updated session record
- **Contents and format:** Lesson Log row for the affected session or sessions: session ID, booking numbers, session headcount, instructors required, open slots, and status (unstaffed, partially staffed, staffed, or cancelled).
- **Next task or recipient:** T5: Retrieve Instructor Availability and Preferences when open slots are greater than 0; otherwise the run ends.
- **Complete when:** The Lesson Log row reflects the booking and the Gmail message is labeled `scheduling/processed`.

### Output 2

- **Output name:** Calendar event
- **Contents and format:** Google Calendar event on the Shared Lesson Calendar: title with lesson type and headcount, start and end time, description with booking numbers and student levels and ages, and staffing status ("Unassigned" until instructors accept).
- **Next task or recipient:** Shared Lesson Calendar (Google Calendar), visible to instructors and the Front Desk Manager.
- **Complete when:** The event exists with matching times and headcount, or is cancelled, and its event ID is saved in the Lesson Log.

### Output 3

- **Output name:** Released instructor list
- **Contents and format:** List of instructor names and emails released from a session, with the reason (cancelled, rescheduled, or fewer instructors needed) and the original lesson date and time.
- **Next task or recipient:** T4: Notify Instructor of Lesson Change.
- **Complete when:** Each released instructor's status in the Lesson Log is set to "released."

## 4. Planned Tools

### Tool 1

- **Tool name:** `update_lesson_record`
- **Input:** Booking details record; Existing session records
- **Output:** Updated session record; Released instructor list
- **Implementation Route:** Web API calls; Google Sheets API read and write of the Lesson Log tab.
- **Integration approach:** Direct integration.
- **Role in this task:** Applies the session rules, recalculates headcount and instructors required, and releases or opens instructor slots.
- **Task timeout:** 60 seconds for one task run, shared with Tool 2, including retries.
- **Maximum retries:** 1
- **Retry only when:** Google Sheets returns a temporary error. Wait 5 seconds. Rows are keyed by booking number and session ID, and T3 reads back the row first, so a retry updates the existing row instead of adding a duplicate.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record "lesson record not updated" in the Run Log tab, leave the Gmail message labeled `scheduling/in-progress`, and escalate to the Front Desk Manager. Do not continue to T4 or T5.

### Tool 2

- **Tool name:** `update_calendar_event`
- **Input:** Updated session record
- **Output:** Calendar event
- **Implementation Route:** Web API calls; Google Calendar API create, update, or cancel on the Shared Lesson Calendar.
- **Integration approach:** Direct integration.
- **Role in this task:** Creates, moves, or cancels the session's calendar event and saves the event ID in the Lesson Log.
- **Task timeout:** 60 seconds for one task run, shared with Tool 1, including retries.
- **Maximum retries:** 1
- **Retry only when:** Google Calendar returns a temporary error. Wait 5 seconds. Before retrying a create, search the calendar for an event tagged with the session ID; if one exists, save its ID and do not create another.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record "calendar not updated" in the Run Log tab and escalate to the Front Desk Manager. The Lesson Log update stands and the run continues, because staffing does not depend on the calendar event existing.
