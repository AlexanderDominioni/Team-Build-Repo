# Send One-Click Assignment Request Task Specification

## Basic Information

- **Task ID:** T7
- **Task name:** Send One-Click Assignment Request
- **Task type:** Act
- **Task owner:** Lesson Scheduling Agent; the Front Desk Manager is accountable.

## 1. Task Description

T7 asks each instructor T6 proposed to commit to the lesson, because an instructor is not assigned until they accept. It sends one email per proposed slot with one-click Accept and Decline buttons and records the response. The fixed rules are:

- The response window is 4 hours after sending, or until 1 hour before the lesson starts, whichever is earlier.
- If the lesson starts within 12 hours, T7 also alerts the Front Desk Manager through H2: Staff Lesson Manually at the same time.
- Accept sends the slot to T8. Decline or no response sends the slot back to T6 with that instructor excluded, up to 2 reassignments per session; after that, the slot goes to H2.

## 2. Inputs

### Input 1

- **Input name:** Proposed assignments
- **Contents and format:** T6's outbound deliverable: session ID, slot number, proposed instructor name and email, and evidence summary.
- **Source:** T6: Assign Instructors to Lesson (Assignment Requests tab rows with status "proposed").

### Input 2

- **Input name:** Lesson details for request
- **Contents and format:** Session record fields: lesson type, date, start and end time, headcount, student levels and ages, and reassignment count.
- **Source:** Lesson Log tab, updated by T3: Update Lesson Record and Shared Calendar.

### Input 3

- **Input name:** Instructor response
- **Contents and format:** One-click response recorded by the response web page: session ID, slot number, instructor, response (accept or decline), and time.
- **Source:** Proposed instructor, by clicking Accept or Decline in the request email.

- **If a required input is missing or invalid:** If the proposed instructor's email is missing, T7 records "not sent — no email" and returns the slot to T6 as declined. A click on an expired or already-used link is ignored and the instructor sees a "this request is no longer active" page.

## 3. Outputs

### Output 1

- **Output name:** Assignment request email
- **Contents and format:** One email per proposed slot: lesson type, date and time, headcount, student levels and ages, the response deadline, and Accept and Decline buttons linked to that session and slot.
- **Next task or recipient:** Proposed instructor.
- **Complete when:** Gmail returns a message ID and the Assignment Requests row shows status "requested" with send time and deadline.

### Output 2

- **Output name:** Recorded response
- **Contents and format:** Assignment Requests row updated to "accepted," "declined," or "expired," with time; for declined or expired, the session's reassignment count is increased by 1.
- **Next task or recipient:** T8: Confirm Assignment on Shared Calendar if accepted; T6: Assign Instructors to Lesson if declined or expired and the reassignment count is 2 or less; H2: Staff Lesson Manually if declined or expired after 2 reassignments.
- **Complete when:** The row has a final status and the next task has been started.

### Output 3

- **Output name:** Short-notice staffing alert
- **Contents and format:** Staffing escalation for a lesson starting within 12 hours: session ID, lesson details, the instructor asked, and the response deadline.
- **Next task or recipient:** H2: Staff Lesson Manually (Front Desk Manager).
- **Complete when:** The alert is sent at the same time as the assignment request.

## 4. Planned Tools

### Tool 1

- **Tool name:** `send_assignment_request`
- **Input:** Proposed assignments; Lesson details for request
- **Output:** Assignment request email; Short-notice staffing alert
- **Implementation Route:** Web API calls; Gmail API send, and Google Sheets API write of the Assignment Requests tab.
- **Integration approach:** Direct integration.
- **Role in this task:** Sends one request email per proposed slot with unique Accept and Decline links, sets the response deadline, and sends the short-notice alert when needed.
- **Task timeout:** 60 seconds for sending, including retries.
- **Maximum retries:** 1 per email
- **Retry only when:** Gmail returns a temporary error and a Sent-folder search for the session ID and slot number finds no message. Wait 5 seconds. If the Sent folder shows the message was sent, record it as sent and do not retry. If the Sent folder cannot be checked, do not retry.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record "request not sent" or "unknown" and escalate the slot to H2: Staff Lesson Manually. Do not start the response window for a request that was not confirmed as sent.

### Tool 2

- **Tool name:** `record_instructor_response`
- **Input:** Instructor response
- **Output:** Recorded response
- **Implementation Route:** Functions/scripts; a Google Apps Script web page that the Accept and Decline links open, which writes to the Assignment Requests tab, plus a scheduled check every 15 minutes that marks requests past their deadline as "expired."
- **Integration approach:** Direct integration.
- **Role in this task:** Records the first valid response for each request, marks unanswered requests as expired at the deadline, and starts the correct next task.
- **Task timeout:** Response window of 4 hours after sending, or until 1 hour before the lesson starts, whichever is earlier; each recording attempt may take at most 10 seconds.
- **Maximum retries:** 1 per recording attempt
- **Retry only when:** Google Sheets returns a temporary error while saving. Wait 2 seconds. Only the first response for a session ID and slot number is kept, so a retry or a second click cannot record two responses.
- **On timeout, exhausted retries, or an error that cannot be retried:** If a response cannot be saved, show the instructor a "could not record — contact the front desk" page and escalate the slot to H2: Staff Lesson Manually. Do not treat an unsaved accept as accepted.
