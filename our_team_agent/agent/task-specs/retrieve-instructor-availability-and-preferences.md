# Retrieve Instructor Availability and Preferences Task Specification

## Basic Information

- **Task ID:** T5
- **Task name:** Retrieve Instructor Availability and Preferences
- **Task type:** Retrieve
- **Task owner:** Lesson Scheduling Agent; the Front Desk Manager is accountable.

## 1. Task Description

T5 reads the current Instructor Availability Sheet at the moment a session needs staffing, because instructors can change their availability at any time. The fixed rule is to read every instructor row and keep only instructors whose available start and end times fully cover the session's start and end time. It returns those instructors with their preferences so T6 can choose among them.

## 2. Inputs

### Input 1

- **Input name:** Updated session record
- **Contents and format:** Structured record: session ID, lesson type and category, date, start and end time, session headcount, open slots.
- **Source:** T3: Update Lesson Record and Shared Calendar.

### Input 2

- **Input name:** Instructor availability rows
- **Contents and format:** One row per instructor: name, email, available start and end times, preferred skill level, preferred age range, and preferred time of day.
- **Source:** Instructor Availability Sheet (Google Sheet maintained by instructors).

- **If a required input is missing or invalid:** Rows with a missing email or unreadable times are left out and listed in the output as "invalid rows" for the Front Desk Manager to fix. If the sheet cannot be read at all, the run stops and escalates to the Front Desk Manager.

## 3. Outputs

### Output 1

- **Output name:** Available instructor list
- **Contents and format:** List of instructors available for the full session: name, email, available start and end times, and preferences; plus a list of invalid rows, and the time the sheet was read.
- **Next task or recipient:** T6: Assign Instructors to Lesson.
- **Complete when:** The list is saved against the session ID in the Run Log tab, even if it is empty.

## 4. Planned Tools

### Tool 1

- **Tool name:** `retrieve_instructor_availability`
- **Input:** Updated session record; Instructor availability rows
- **Output:** Available instructor list
- **Implementation Route:** Web API calls; read-only Google Sheets API access to the Instructor Availability Sheet.
- **Integration approach:** Direct integration.
- **Role in this task:** Reads all instructor rows, filters to those whose availability covers the session, and returns them with their preferences.
- **Task timeout:** 30 seconds for one task run, including retries.
- **Maximum retries:** 2
- **Retry only when:** Google Sheets returns a temporary connection or rate-limit error. Wait 5 seconds. The tool is read-only, so retries cannot create duplicates.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record "availability not retrieved" in the Run Log tab and send the session to H2: Staff Lesson Manually. Do not treat an unreadable sheet as "no instructors available."
