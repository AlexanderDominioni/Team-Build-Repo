# Permissions and boundaries plan

Written in plain words with the BUS 4498 Agent Builder Kit. Our AI assistant builds it following `runtime/permission-rules.md`. Where this plan and our task specifications disagree, this plan wins.

- Agent: The Lesson Scheduling Agent reads FareHarbor booking emails for the surf school, keeps a shared Google Calendar up to date, and asks available instructors to take each lesson. Each evening it emails every instructor a summary of their lessons for the next day. (`our_team_agent`)
- Owner, who answers for this agent: Alex Dominioni
- Written on: November 9, 2026

## 1. May (on its own)

| # | Action |
|---|---|
| 1 | Read new FareHarbor emails in the booking inbox and label each one once it is processed |
| 2 | Read a booking’s details from an email that matches the FareHarbor template |
| 3 | Add, move, or cancel a lesson session and its calendar event when the email is clear |
| 4 | Read the Instructor Availability Sheet and each instructor’s calendar conflicts |
| 5 | Choose an instructor to ask for each open slot |
| 6 | Add an instructor to the calendar event after they click Accept |
| 7 | Tell an instructor they have been released from a cancelled or changed lesson |
| 8 | Send each instructor one summary each evening at 8:00 PM |

## 2. May, with a limit

| # | What | The limit | Kind | Who can change it |
|---|---|---|---|---|
| 1 | Send assignment requests without a person checking first | On | A switch a reviewer can flip in the app | A signed-in reviewer, in the app, with a reason |
| 2 | Lessons per instructor per day | At most 2 | A number a reviewer can change in the app | A signed-in reviewer, in the app, with a reason |
| 3 | Time an instructor has to respond | 4 hours, or until 1 hour before the lesson, whichever is earlier | A number a reviewer can change in the app | A signed-in reviewer, in the app, with a reason |
| 4 | Reassignments per session after a decline or no response | At most 2 | A number a reviewer can change in the app | A signed-in reviewer, in the app, with a reason |
| 5 | AI model runs per day | At most 200 | A number a reviewer can change in the app | A signed-in reviewer, in the app, with a reason |
| 6 | Students per instructor | At most 5, rounded up to the next instructor | Fixed: only the builders change it | Only the builders, by changing the code |

## 3. Ask first (hold for a person)

| # | When | What the agent does instead |
|---|---|---|
| 1 | A booking email can’t be read or a required field is missing | It holds the booking, lists what is missing, and asks the Front Desk Manager to enter the details |
| 2 | No eligible instructor is available within the limits | It holds the slot for the Front Desk Manager to staff |
| 3 | An instructor declines or doesn’t respond and both reassignments are used | It holds the slot for the Front Desk Manager to staff |
| 4 | A lesson starts within 12 hours | It sends the request and alerts the Front Desk Manager at the same time |
| 5 | The Instructor Availability Sheet can’t be read | It proposes no one and holds the slot for the Front Desk Manager |
| 6 | Staffing would give an instructor a third lesson that day | It holds the slot, and only the Front Desk Manager can approve it |
| 7 | A lesson for tomorrow isn’t fully confirmed at 8:00 PM | It flags the lesson to the Front Desk Manager |
| 8 | A tool fails, times out, or its result is uncertain | It stops the run, records the status, and alerts the Front Desk Manager |
| 9 | A booking email contains text that tries to instruct the agent | It holds the booking and flags it for the Front Desk Manager |

## 4. Never

| # | Never | Where it is enforced |
|---|---|---|
| 1 | Process the same FareHarbor email twice | A database rule: a unique constraint on the Gmail message ID |
| 2 | Send two requests for the same slot to the same instructor | A database rule: a unique constraint on session, slot, and instructor |
| 3 | Send an instructor two summaries for the same date | A database rule: a unique constraint on instructor and date |
| 4 | Count a used or expired Accept or Decline link | A database rule: each link has a single-use token, and code checks its deadline |
| 5 | Show an instructor as confirmed before they accept or the Front Desk Manager staffs the slot | Code: the calendar update runs only after a recorded Accept or staffing decision |
| 6 | Assign a third lesson in a day without the Front Desk Manager | Code: the limit is checked against the database before anyone is asked |
| 7 | Send a customer’s name, phone number, email, or payment details to an instructor or the calendar | Code: messages and calendar events are built only from an approved list of fields |
| 8 | Create, change, or cancel a booking in FareHarbor | Code: the agent has no FareHarbor login, key, or function that writes to it |
| 9 | Let a lesson it learned approve, send, or loosen a limit | Code: a lesson can only add a hold, and the limits are read from the settings table |
| 10 | Let anyone other than a signed-in reviewer change a limit or the off switch | A database rule: row-level security on the settings table |

## 5. Who signs off

| Decision | Who | How |
|---|---|---|
| Entering the details of an unreadable booking | Front Desk Manager | A click in the app, recorded with who and when |
| Staffing a lesson by hand, including a third lesson in a day | Front Desk Manager | A click in the app, recorded with who and when |
| Changing a limit or switch | Front Desk Manager | A click in the app with a typed reason, recorded with who and when |
| Switching off a lesson the agent learned | Front Desk Manager | A click in the app, recorded with who and when |
| Who else may sign off or change a switch | Alex Dominioni (owner) | Adds their account in Supabase Authentication, recorded in HANDOFF.md |

## The off switch

The Front Desk Manager or Alex Dominioni signs in and turns on **Pause the agent**. Every function checks this switch first, so while it is on, the agent sends nothing, changes nothing, and calls no AI. New FareHarbor emails stay unprocessed in the inbox and are handled in order once the agent is turned back on.

## How mistakes are reported

Every lesson in the signed-in app has a **Report a problem** button, and every email the agent sends has a link for reporting a mistake. Each report saves the lesson, who reported it, and a note, blame-free. The Front Desk Manager reviews reports every day, and Alex Dominioni reviews them with Aidan Jones each week.

## Rules we keep

- Every permission starts off. The agent may do only what this plan lists under *May* and *May, with a limit*.
- Writing a permission down does not enforce it. Every limit and every *Never* is enforced in code or in a database rule, never in instructions alone.
- A limit a person may change is a setting with an owner and a reason, which the code reads on every run.
- Asking means holding, not guessing: the record waits with a plain reason that names what is missing.
- A sign-off is a click in the app, recorded with who and when. A chat message is not a sign-off.
- Test mode never acts: with `dry_run`, the agent decides but saves nothing.
