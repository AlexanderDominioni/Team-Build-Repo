# Notify Instructor of Lesson Change Task Specification

## Basic Information

- **Task ID:** T4
- **Task name:** Notify Instructor of Lesson Change
- **Task type:** Act
- **Task owner:** Lesson Scheduling Agent; the Front Desk Manager is accountable.

## 1. Task Description

T4 tells each instructor who was released from a lesson that they are no longer needed, so they do not show up for a cancelled or moved lesson. The fixed rule is one email per released instructor per session change, stating the lesson, the original date and time, and the reason. Instructors whose request was still pending also have their Accept and Decline links deactivated.

## 2. Inputs

### Input 1

- **Input name:** Released instructor list
- **Contents and format:** List of instructor names and emails, reason for release, session ID, lesson type, and original date and time.
- **Source:** T3: Update Lesson Record and Shared Calendar.

- **If a required input is missing or invalid:** If an instructor's email is missing, T4 skips that instructor, records "not notified — no email" in the Run Log tab, and escalates to the Front Desk Manager to contact them directly.

## 3. Outputs

### Output 1

- **Output name:** Lesson change notice
- **Contents and format:** One email per released instructor: lesson type, original date and time, reason (cancelled, rescheduled, or fewer instructors needed), and a note that no action is needed.
- **Next task or recipient:** Released instructor(s).
- **Complete when:** Gmail returns a message ID for each email.

### Output 2

- **Output name:** Notice send record
- **Contents and format:** Assignment Requests tab rows updated to status "released — notified" with send time and Gmail message ID, or "released — not notified" with the reason.
- **Next task or recipient:** Assignment Requests tab; the run continues to T5 if the session still has open slots.
- **Complete when:** Every released instructor has a send record.

## 4. Planned Tools

### Tool 1

- **Tool name:** `send_lesson_change_notice`
- **Input:** Released instructor list
- **Output:** Lesson change notice; Notice send record
- **Implementation Route:** Web API calls; Gmail API send, and Google Sheets API write of the Assignment Requests tab.
- **Integration approach:** Direct integration.
- **Role in this task:** Sends each released instructor one notice, deactivates any pending Accept and Decline links, and records the send status.
- **Task timeout:** 60 seconds for one task run, including retries.
- **Maximum retries:** 1 per instructor
- **Retry only when:** Gmail returns a temporary error and a Sent-folder search for the session ID and instructor finds no message. Wait 5 seconds. If the Sent folder shows the message was sent, record it as sent and do not retry. If the Sent folder cannot be checked, do not retry.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record "released — not notified" or "unknown" and escalate to the Front Desk Manager to contact the instructor. The run still continues to T5 if needed, because the instructor is already released in the Lesson Log.
