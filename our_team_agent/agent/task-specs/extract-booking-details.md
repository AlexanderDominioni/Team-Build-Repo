# Extract Booking Details Task Specification

## Basic Information

- **Task ID:** T2
- **Task name:** Extract Booking Details
- **Task type:** Sense
- **Task owner:** Lesson Scheduling Agent; the Front Desk Manager is accountable.

## 1. Task Description

T2 turns the FareHarbor email into a structured booking record that the rest of the workflow can use. FareHarbor emails follow a fixed template, so T2 uses fixed parsing rules: the subject prefix sets the notification type ("New online booking:" is new; prefixes for cancellation or rebooking are cancelled or rescheduled), and labeled fields in the body supply the rest. T2 classifies the lesson as private (lesson type name contains "Private") or open group, and calculates instructors required as the headcount divided by 5, rounded up (5 students need 1 instructor; 6 need 2).

## 2. Inputs

### Input 1

- **Input name:** Raw booking email
- **Contents and format:** Structured record: Gmail message ID, subject line, received time, and HTML body text.
- **Source:** T1: Retrieve FareHarbor Booking Email.

- **If a required input is missing or invalid:** If the subject prefix is not recognized, or booking number, lesson type, date, time, or headcount is missing or cannot be parsed, T2 marks the record "needs review" with the missing fields listed and sends it to H1: Resolve Unreadable Booking Email. No partial record is passed to T3.

## 3. Outputs

### Output 1

- **Output name:** Booking details record
- **Contents and format:** Structured record: Gmail message ID, notification type (new, cancelled, rescheduled), FareHarbor booking number, FareHarbor contact ID (from the "View on FareHarbor" link), lesson type, lesson category (private or open group), lesson date, start time, end time, headcount, each student's experience level and age, customer name, phone, and email, and instructors required.
- **Next task or recipient:** T3: Update Lesson Record and Shared Calendar.
- **Complete when:** All required fields are filled and pass format checks (valid date and time, headcount of at least 1, instructors required equal to headcount divided by 5 rounded up).

### Output 2

- **Output name:** Extraction exception
- **Contents and format:** Structured record: Gmail message ID, subject line, fields that could not be read, and a link to the email.
- **Next task or recipient:** H1: Resolve Unreadable Booking Email (Front Desk Manager).
- **Complete when:** The exception is saved in the Run Log tab and the Front Desk Manager has been notified.

## 4. Planned Tools

### Tool 1

- **Tool name:** `extract_booking_details`
- **Input:** Raw booking email
- **Output:** Booking details record; Extraction exception
- **Implementation Route:** Functions/scripts; a Python HTML-parsing script with fixed rules for the FareHarbor template.
- **Integration approach:** Direct integration.
- **Role in this task:** Classifies the notification type, reads each labeled field, calculates instructors required, and returns either a complete booking details record or an extraction exception.
- **Task timeout:** 15 seconds for one task run.
- **Maximum retries:** 0
- **Retry only when:** Not applicable. Parsing the same email again gives the same result.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record "extraction failed" in the Run Log tab and send the extraction exception to H1: Resolve Unreadable Booking Email. The run does not continue until H1 supplies the details.
