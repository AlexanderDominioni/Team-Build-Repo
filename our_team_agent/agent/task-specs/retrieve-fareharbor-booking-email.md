# Retrieve FareHarbor Booking Email Task Specification

## Basic Information

- **Task ID:** T1
- **Task name:** Retrieve FareHarbor Booking Email
- **Task type:** Retrieve
- **Task owner:** Lesson Scheduling Agent; the Front Desk Manager is accountable.

## 1. Task Description

T1 starts each booking run by retrieving one FareHarbor notification email that has not yet been processed. The fixed rule is: every 5 minutes, search the booking inbox for messages from `messages@fareharbor.com` that do not carry the `scheduling/processed` label, and hand each one to T2 as a separate run, oldest first. Labeling a message as processed only after T3 succeeds ensures each email is handled exactly once and none are skipped.

## 2. Inputs

### Input 1

- **Input name:** FareHarbor notification email
- **Contents and format:** Gmail message from `messages@fareharbor.com`: Gmail message ID, subject line (for example, "New online booking: [lesson type] on [date] at [time range]"), received time, and HTML body.
- **Source:** Surf school booking inbox (Gmail), which receives FareHarbor notifications.

- **If a required input is missing or invalid:** If the inbox cannot be reached, no run starts and the next 5-minute check tries again. If a message has an empty or unreadable body, T1 passes it to T2 marked "body unreadable" so the case goes to H1: Resolve Unreadable Booking Email.

## 3. Outputs

### Output 1

- **Output name:** Raw booking email
- **Contents and format:** Structured record: Gmail message ID, subject line, received time, and the HTML body as text.
- **Next task or recipient:** T2: Extract Booking Details.
- **Complete when:** One record exists for the Gmail message ID and the message is labeled `scheduling/in-progress`.

## 4. Planned Tools

### Tool 1

- **Tool name:** `retrieve_booking_email`
- **Input:** FareHarbor notification email
- **Output:** Raw booking email
- **Implementation Route:** Web API calls; Gmail API search and read of the booking inbox, and a label change on the retrieved message.
- **Integration approach:** Direct integration.
- **Role in this task:** Finds unprocessed FareHarbor messages, returns the oldest one as a raw booking email, and labels it `scheduling/in-progress` so a parallel check does not start a second run for it.
- **Task timeout:** 30 seconds for one task run, including retries.
- **Maximum retries:** 2
- **Retry only when:** Gmail returns a temporary connection or rate-limit error. Wait 5 seconds before each retry. Reading and labeling are repeatable, so retries cannot create duplicate runs: a message already labeled `scheduling/in-progress` is skipped.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record "retrieval failed" with the error in the Run Log tab. Do not start a run; the message stays unlabeled and is picked up at the next 5-minute check. If three checks in a row fail, email the Front Desk Manager to check FareHarbor directly.
