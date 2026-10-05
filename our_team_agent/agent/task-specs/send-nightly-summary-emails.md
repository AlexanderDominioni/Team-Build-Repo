# Send Nightly Summary Emails Task Specification

## Basic Information

- **Task ID:** T10
- **Task name:** Send Nightly Summary Emails
- **Task type:** Act
- **Task owner:** Lesson Scheduling Agent; the Front Desk Manager is accountable.

## 1. Task Description

T10 emails each instructor their summary for the next day so every instructor knows their schedule by the evening before. The fixed rule is exactly one summary email per instructor per date, sent from the surf school's scheduling account with the date in the subject line.

## 2. Inputs

### Input 1

- **Input name:** Instructor summaries
- **Contents and format:** One record per instructor: name, email, date, and the list of accepted lessons or "no lessons tomorrow."
- **Source:** T9: Compile Nightly Lesson Summaries.

- **If a required input is missing or invalid:** If no summaries are received, T10 sends nothing. A summary with an invalid email is skipped, recorded as "not sent — invalid email," and escalated to the Front Desk Manager.

## 3. Outputs

### Output 1

- **Output name:** Nightly summary email
- **Contents and format:** One email per instructor with subject "Your lessons for [date]" listing each lesson's time, lesson type, headcount, and student levels and ages, or stating "No lessons tomorrow."
- **Next task or recipient:** Each instructor on the roster.
- **Complete when:** Gmail returns a message ID for the email.

### Output 2

- **Output name:** Summary send record
- **Contents and format:** One Run Log row per instructor and date: send status (sent, failed, or unknown), send time, and Gmail message ID.
- **Next task or recipient:** Run Log tab; the Front Desk Manager reviews failures.
- **Complete when:** Every instructor summary has a send record.

## 4. Planned Tools

### Tool 1

- **Tool name:** `send_summary_emails`
- **Input:** Instructor summaries
- **Output:** Nightly summary email; Summary send record
- **Implementation Route:** Web API calls; Gmail API send, and Google Sheets API write of the Run Log tab.
- **Integration approach:** Direct integration.
- **Role in this task:** Checks that no summary for this instructor and date was already sent, sends the email, and records the result.
- **Task timeout:** 5 minutes for one task run, including retries.
- **Maximum retries:** 1 per email
- **Retry only when:** Gmail returns a temporary error and a Sent-folder search for the instructor and date finds no message. Wait 10 seconds. If the Sent folder shows the message was sent, record it as sent and do not retry. If the Sent folder cannot be checked, do not retry.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record "failed" or "unknown" for that instructor and email the Front Desk Manager a list of instructors who did not receive a summary so they can be contacted directly. Other instructors' emails still go out.
