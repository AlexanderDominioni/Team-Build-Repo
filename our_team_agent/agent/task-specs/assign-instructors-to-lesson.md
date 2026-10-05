# Assign Instructors to Lesson Task Specification

```yaml
# BASIC INFORMATION
task_id: "T6"
task_name: "Assign Instructors to Lesson"
task_owner: "Front Desk Manager"
```

## 1. Task Goal

- **Objective:** Choose one instructor for each open slot in a lesson session so the session meets the one-instructor-per-five-students ratio. Every chosen instructor must be available and free of conflicts. The choice should keep repeat customers with their previous instructor, match instructor preferences, and spread lessons evenly across instructors, with no instructor over two lessons per day.

## 2. Inbound Inputs

### Input 1

- **Input name:** Updated session record
- **What it contains:** Session ID, lesson type, lesson category (private or open group), date, start and end time, booking numbers, session headcount, each student's experience level and age, FareHarbor contact ID for each booking, instructors required, instructors already assigned, and open slots.
- **Source:** T3: Update Lesson Record and Shared Calendar (Lesson Log tab).

### Input 2

- **Input name:** Available instructor list
- **What it contains:** Instructors whose availability covers the full session: name, email, available start and end times, preferred skill level, preferred age range, and preferred time of day.
- **Source:** T5: Retrieve Instructor Availability and Preferences.

### Input 3

- **Input name:** Request history for this session
- **What it contains:** Instructors who previously declined or did not respond to a request for this session, and the session's reassignment count (0, 1, or 2).
- **Source:** Assignment Requests tab, updated by T7: Send One-Click Assignment Request.

## 3. Tool Permissions and Boundaries

### Task-Wide Limits

- **Total task timeout:** 3 minutes for one task run, including tool calls, retries, and waiting.
- **Maximum tool calls:** 12 across all tools during one task run; retries count toward this total.

Tools may use only the Lesson Log, Assignment Requests tab, and Shared Lesson Calendar. They may not contact instructors or customers, change FareHarbor bookings, edit the Instructor Availability Sheet, or change calendar events. Tools 1–3 are read-only. Tool 4 may write only proposed assignments for this session.

### Tool 1

- **Tool name:** `check_calendar_conflicts`
- **Tool type:** API request (Google Calendar API free/busy query).
- **Supports these permitted subtasks:** Screen Eligible Instructors; Fill Multiple Instructor Slots.
- **Allowed use:** Read whether candidate instructors already have a lesson on the Shared Lesson Calendar that overlaps the session time.
- **Prohibited use:** Creating, changing, or deleting calendar events; reading calendars other than the Shared Lesson Calendar.
- **Approval required:** None within the allowed use.
- **Timeout per call:** 10 seconds.
- **Maximum retries per call:** 1
- **Retry conditions and failure response:** Retry once after 2 seconds on a temporary API error. The call is read-only, so retries cannot cause duplicates. If it still fails, treat the instructor's conflict status as unknown, do not propose them, and note it in Unresolved issues.

### Tool 2

- **Tool name:** `retrieve_instructor_workload`
- **Tool type:** Database query (Google Sheets API read of the Lesson Log and Assignment Requests tabs).
- **Supports these permitted subtasks:** Screen Eligible Instructors; Balance Workload.
- **Allowed use:** Count each candidate's requested and accepted lessons on the session date and over the past 30 days.
- **Prohibited use:** Writing to any tab; reading customer contact details.
- **Approval required:** None within the allowed use.
- **Timeout per call:** 10 seconds.
- **Maximum retries per call:** 1
- **Retry conditions and failure response:** Retry once after 2 seconds on a temporary error. If it still fails, do not propose any instructor whose daily count is unknown; hand off if no candidate can be confirmed under the two-lesson limit.

### Tool 3

- **Tool name:** `retrieve_customer_history`
- **Tool type:** Database query (Google Sheets API read of the Lesson Log).
- **Supports these permitted subtasks:** Check Repeat Customers.
- **Allowed use:** Look up past sessions by FareHarbor contact ID and return the instructors who taught each repeat customer and when.
- **Prohibited use:** Reading or returning customer phone numbers, emails, or payment details; writing to any tab.
- **Approval required:** None within the allowed use.
- **Timeout per call:** 10 seconds.
- **Maximum retries per call:** 1
- **Retry conditions and failure response:** Retry once after 2 seconds on a temporary error. If it still fails, continue without repeat-customer evidence and note it in Unresolved issues; this preference is not required to complete the task.

### Tool 4

- **Tool name:** `record_proposed_assignment`
- **Tool type:** Database query (Google Sheets API write to the Assignment Requests tab).
- **Supports these permitted subtasks:** Record Proposed Assignment.
- **Allowed use:** Write one row per open slot with session ID, slot number, chosen instructor, status "proposed," and the evidence summary.
- **Prohibited use:** Changing rows for other sessions; marking any assignment accepted; overwriting rows with status "accepted."
- **Approval required:** Assigning an instructor a third lesson on the same day requires Front Desk Manager approval through H2: Staff Lesson Manually; the agent may not propose it.
- **Timeout per call:** 10 seconds.
- **Maximum retries per call:** 1
- **Retry conditions and failure response:** Before retrying, read back the rows for this session ID and slot number. Rows are keyed by session ID and slot number, so a retry replaces the proposal instead of adding a duplicate. If the read-back cannot confirm the write, do not retry; hand off with the chosen instructors listed in the deliverable.

## 4. How the Agent Should Reason

### Permitted Subtask 1

- **Subtask name:** Screen Eligible Instructors
- **Subtask description:** Checks each instructor on the available instructor list against the hard rules: no overlapping lesson on the Shared Lesson Calendar, fewer than two lessons already on the session date, and not previously declined or expired for this session. Produces the list of eligible instructors.
- **Subtask boundary:** May only read data. An instructor who fails any hard rule, or whose status is unknown, may not be proposed.
- **Retry limits:** 1 additional attempt, only after a tool retry supplies previously missing data.

### Permitted Subtask 2

- **Subtask name:** Check Repeat Customers
- **Subtask description:** Looks up each booking's FareHarbor contact ID to find instructors who taught that customer before. For a private lesson, the previous instructor is the strongest preference. For an open group session, prefers the previous instructor shared by the most bookings.
- **Subtask boundary:** May consider only eligible instructors from Screen Eligible Instructors. May not read customer contact or payment details.
- **Retry limits:** 0

### Permitted Subtask 3

- **Subtask name:** Compare Preference Fit
- **Subtask description:** Compares each eligible instructor's preferred skill level, age range, and time of day with the session's student levels, ages, and start time. Produces a fit rating (strong, partial, or none) for each instructor.
- **Subtask boundary:** Preferences are tie-breakers, not hard rules; a poor fit does not make an instructor ineligible.
- **Retry limits:** 0

### Permitted Subtask 4

- **Subtask name:** Balance Workload
- **Subtask description:** Compares eligible instructors' lesson counts on the session date and over the past 30 days. Produces a ranking that favors instructors with fewer lessons.
- **Subtask boundary:** May not override the two-lessons-per-day limit or a strong repeat-customer match without stating why in the evidence summary.
- **Retry limits:** 1 additional attempt, only after a tool retry supplies previously missing workload data.

### Permitted Subtask 5

- **Subtask name:** Fill Multiple Instructor Slots
- **Subtask description:** When a session has two or more open slots, chooses a set of different eligible instructors who are all free for the full session. Rechecks calendar conflicts for the chosen set.
- **Subtask boundary:** Each instructor may fill only one slot per session. If only some slots can be filled, propose those and hand off the rest.
- **Retry limits:** 1 additional attempt with a different set if the first set has a conflict.

### Permitted Subtask 6

- **Subtask name:** Record Proposed Assignment
- **Subtask description:** Saves one proposed instructor per filled slot, with the evidence for each choice, to the Assignment Requests tab.
- **Subtask boundary:** Requires that every proposed instructor passed Screen Eligible Instructors. May not mark any assignment accepted or contact instructors.
- **Retry limits:** 1 additional attempt, following Tool 4's read-back rule.

- **Decision guidance:** After each subtask, use its findings to select the permitted subtask most likely to resolve the most important remaining uncertainty. Do not follow a fixed sequence. For example, if screening leaves only one eligible instructor for one open slot, go straight to Record Proposed Assignment. If a repeat customer's previous instructor is eligible, preference and workload checks may be shortened. If screening leaves no eligible instructors, hand off. If no permitted subtask can make useful progress, stop and hand the case to a person.

## 5. When to Stop or Hand Off to a Human

- **Stop successfully when:** Every open slot has a proposed instructor saved in the Assignment Requests tab, each proposed instructor passed every hard rule, and the evidence summary states the repeat-customer, preference, and workload findings behind each choice.
- **Hand off early when:** No eligible instructor exists for one or more slots; the only eligible instructors would exceed two lessons that day; the available instructor list is empty or could not be read; calendar or workload data is unknown for all candidates; or a task-wide limit is reached before every slot is filled.
- **Hand off to:** Front Desk Manager through H2: Staff Lesson Manually.

Stop at the first applicable budget limit or handoff condition. While awaiting review, take no further autonomous action.

## 6. Outbound Deliverable

- **Status:** Completed or escalated to human.
- **Result or recommendation:** One proposed instructor (name and email) per open slot. If escalated before reaching a supported result, write undetermined for each unfilled slot.
- **Evidence summary:** For each proposal: hard rules passed, repeat-customer match (if any), preference fit, and workload comparison; or why no eligible instructor could be found.
- **Subtasks performed:** Permitted subtasks completed, including repeated attempts.
- **Unresolved issues:** Remaining uncertainties, such as instructors with unknown conflict status; use none only if no unresolved issue remains.
- **Handoff note:** Reason for stopping, which slots are unfilled, instructors ruled out and why, and what the Front Desk Manager needs to decide; write "Not applicable" for a completed task.
- **Next task or recipient:** T7: Send One-Click Assignment Request. Unfilled slots go to the Front Desk Manager through H2: Staff Lesson Manually.
