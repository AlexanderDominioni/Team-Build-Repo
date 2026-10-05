# Staff Lesson Manually Task Specification

## Basic Information

- **Task ID:** H2
- **Task name:** Staff Lesson Manually
- **Task type:** Decide
- **Task owner:** Front Desk Manager.

## 1. Task Description

H2 is where the Front Desk Manager resolves any lesson the system cannot staff on its own. It is used when T6 finds no eligible instructor, when an instructor declines or does not respond after two reassignments, when a lesson starts within 12 hours (as a parallel alert alongside T7's request), when T5 cannot read the availability sheet, and when T9 finds a lesson for the next day that is not fully confirmed. The Front Desk Manager uses their own judgment, for example calling instructors directly, approving a third lesson for an instructor that day, or contacting the customer to reschedule or cancel in FareHarbor.

## 2. Inputs

### Input 1

- **Input name:** Staffing escalation
- **Contents and format:** Structured record: session ID, lesson type, date and time, headcount, open slots, reason for escalation, and T6's deliverable or the request history (instructors asked, declined, or expired).
- **Source:** T5: Retrieve Instructor Availability and Preferences, T6: Assign Instructors to Lesson, T7: Send One-Click Assignment Request, or T9: Compile Nightly Lesson Summaries.

### Input 2

- **Input name:** Manual staffing decision
- **Contents and format:** Human response through a Staffing Decision Google Form: session ID, outcome ("instructor arranged" with instructor name for each slot, or "rescheduled or cancelled in FareHarbor"), the Front Desk Manager's name, and time.
- **Source:** Front Desk Manager.

- **If a required input is missing or invalid:** The form rejects an "instructor arranged" outcome without an instructor named for each open slot, or a name not on the instructor roster. If an instructor accepts through T7 before the Front Desk Manager responds, the escalation for that slot closes automatically.

## 3. Outputs

### Output 1

- **Output name:** Manual staffing record
- **Contents and format:** Assignment Requests tab rows for the session, each with status "accepted — staffed manually," instructor name and email, the Front Desk Manager's name, and time.
- **Next task or recipient:** T8: Confirm Assignment on Shared Calendar.
- **Complete when:** Every open slot has an instructor recorded, or the outcome is "rescheduled or cancelled in FareHarbor," which ends this run (FareHarbor's new email starts a new run at T1).

## 4. Planned Tools

### Tool 1

- **Tool name:** `record_manual_staffing`
- **Input:** Staffing escalation; Manual staffing decision
- **Output:** Manual staffing record
- **Implementation Route:** Web API calls; Gmail API alert to the Front Desk Manager with a prefilled Staffing Decision Google Form, and Google Sheets API write to the Assignment Requests tab.
- **Integration approach:** Direct integration.
- **Role in this task:** Alerts the Front Desk Manager with the escalation details, validates their decision, and saves it so T8 can update the calendar. It does not choose an instructor.
- **Task timeout:** Human response deadline: 2 hours after the alert, and always before the lesson start time. Alerts raised after 6:00 PM for lessons the next morning are due by 7:00 AM.
- **Maximum retries:** Not applicable — manual task.
- **Retry only when:** Not applicable.
- **On timeout, exhausted retries, or an error that cannot be retried:** If the deadline passes, mark the session "unstaffed — overdue" in the Lesson Log, send one reminder, and keep the calendar event marked "Unassigned." The lesson is never shown as staffed until a decision is recorded. If saving fails, the Front Desk Manager resubmits; rows are keyed by session ID and slot number so a resubmission replaces the earlier entry.
