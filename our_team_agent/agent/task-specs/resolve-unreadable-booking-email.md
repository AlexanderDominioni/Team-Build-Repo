# Resolve Unreadable Booking Email Task Specification

## Basic Information

- **Task ID:** H1
- **Task name:** Resolve Unreadable Booking Email
- **Task type:** Verify
- **Task owner:** Front Desk Manager.

## 1. Task Description

When T2 cannot read a FareHarbor email or finds required fields missing, the Front Desk Manager opens the booking in FareHarbor, confirms what changed, and enters the booking details by hand. The workflow needs H1 so that a template change or unusual notification does not cause a lesson to be skipped or recorded wrong. The Front Desk Manager uses their own judgment and the FareHarbor dashboard as the source of truth.

## 2. Inputs

### Input 1

- **Input name:** Extraction exception
- **Contents and format:** Structured record: Gmail message ID, subject line, fields that could not be read, and a link to the email.
- **Source:** T2: Extract Booking Details.

### Input 2

- **Input name:** Manual booking entry
- **Contents and format:** Human response through a Booking Correction Google Form: the same fields as the booking details record (notification type, booking number, contact ID, lesson type and category, date, start and end time, headcount, student levels and ages, customer contact), plus the Front Desk Manager's name and time of entry.
- **Source:** Front Desk Manager, using the FareHarbor dashboard.

- **If a required input is missing or invalid:** The form rejects entries with missing required fields or an invalid date, time, or headcount and asks for resubmission. If the email is not a booking notification (for example, a FareHarbor marketing email), the Front Desk Manager marks it "not a booking" and the run ends with no changes.

## 3. Outputs

### Output 1

- **Output name:** Corrected booking details record
- **Contents and format:** A booking details record in the same format T2 produces, with source "entered manually in H1."
- **Next task or recipient:** T3: Update Lesson Record and Shared Calendar.
- **Complete when:** The form entry passes validation and is saved in the Run Log tab against the Gmail message ID.

## 4. Planned Tools

### Tool 1

- **Tool name:** `record_manual_booking`
- **Input:** Extraction exception; Manual booking entry
- **Output:** Corrected booking details record
- **Implementation Route:** Web API calls; a Gmail API notification to the Front Desk Manager linking a prefilled Booking Correction Google Form, and a Google Sheets API write to the Run Log tab.
- **Integration approach:** Direct integration.
- **Role in this task:** Sends the Front Desk Manager the exception, validates their form entry, and passes the corrected record to T3. It does not guess missing values.
- **Task timeout:** Human response deadline: 2 hours after notification during business hours (8:00 AM–6:00 PM); exceptions raised after 6:00 PM are due by 9:00 AM the next day.
- **Maximum retries:** Not applicable — manual task.
- **Retry only when:** Not applicable.
- **On timeout, exhausted retries, or an error that cannot be retried:** If the deadline passes, mark the run "Booking unresolved — overdue" and send one reminder to the Front Desk Manager. The booking is not added to the Lesson Log or calendar until H1 is completed. If saving fails, the Front Desk Manager resubmits; entries are keyed by Gmail message ID so a resubmission replaces the earlier one.
