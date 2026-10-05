# Compile Nightly Lesson Summaries Task Specification

## Basic Information

- **Task ID:** T9
- **Task name:** Compile Nightly Lesson Summaries
- **Task type:** Reason
- **Task owner:** Lesson Scheduling Agent; the Front Desk Manager is accountable.

## 1. Task Description

At 8:00 PM each day, T9 builds one summary for every instructor on the roster covering the next day. The fixed rules are: include only lessons the instructor has accepted; list them in start-time order with time, lesson type, headcount, and student levels and ages; and give instructors with no accepted lessons a "no lessons tomorrow" summary. Any next-day session that is not fully staffed is flagged to the Front Desk Manager through H2: Staff Lesson Manually.

## 2. Inputs

### Input 1

- **Input name:** Next-day sessions
- **Contents and format:** Lesson Log rows for the next day: session ID, lesson type, start and end time, headcount, student levels and ages, accepted instructors, and staffing status.
- **Source:** Lesson Log tab, maintained by T3: Update Lesson Record and Shared Calendar and T8: Confirm Assignment on Shared Calendar.

### Input 2

- **Input name:** Instructor roster
- **Contents and format:** Every instructor's name and email.
- **Source:** Instructor Availability Sheet.

- **If a required input is missing or invalid:** If the Lesson Log or roster cannot be read, T9 produces no summaries, records "summaries not compiled," and escalates to the Front Desk Manager so no instructor receives a false "no lessons" email. Instructors with no email are listed in the Run Log tab for the Front Desk Manager.

## 3. Outputs

### Output 1

- **Output name:** Instructor summaries
- **Contents and format:** One record per roster instructor: name, email, date, and either the list of accepted lessons (start and end time, lesson type, headcount, student levels and ages) or "no lessons tomorrow."
- **Next task or recipient:** T10: Send Nightly Summary Emails.
- **Complete when:** Every instructor on the roster with a valid email has exactly one summary for the date.

### Output 2

- **Output name:** Unconfirmed lesson flags
- **Contents and format:** One staffing escalation per next-day session that is not fully staffed: session ID, lesson details, open slots, and pending requests.
- **Next task or recipient:** H2: Staff Lesson Manually (Front Desk Manager).
- **Complete when:** Each flag is sent, or the run log shows none were needed.

## 4. Planned Tools

### Tool 1

- **Tool name:** `compile_lesson_summaries`
- **Input:** Next-day sessions; Instructor roster
- **Output:** Instructor summaries; Unconfirmed lesson flags
- **Implementation Route:** Functions/scripts; a Python script using read-only Google Sheets API access to the Lesson Log and Instructor Availability Sheet, with a Gmail API alert for unconfirmed lesson flags.
- **Integration approach:** Direct integration.
- **Role in this task:** Groups next-day accepted lessons by instructor, adds "no lessons" summaries, and flags sessions that are not fully staffed.
- **Task timeout:** 2 minutes for one task run, including retries.
- **Maximum retries:** 2
- **Retry only when:** A Google Sheets read returns a temporary error. Wait 10 seconds. Reads are repeatable and summaries are keyed by instructor and date, so retries cannot create duplicates.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record "summaries not compiled" in the Run Log tab and escalate to the Front Desk Manager to message instructors directly. T10 does not run.
