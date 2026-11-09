# Memory plan

Written in plain words with the BUS 4498 Agent Builder Kit. Our AI assistant designs the files and tables from it, following `runtime/memory-rules.md`. This plan adds to `runtime/hosting-plan.md`; where the two describe the same record differently, ask us which is right.

## What our agent must know or remember

| # | What | Kind | Where it lives | Who can change it | When it expires |
|---|---|---|---|---|---|
| 1 | The FareHarbor email template: subject prefixes, labeled fields, and how to tell private lessons from open groups | Knowledge it reads | A file only we change | Only the builders | Never |
| 2 | The instructor roster: each instructor’s name and email | Knowledge it reads | A table a person keeps current | A business person | When a newer record replaces it |
| 3 | Each instructor’s available times and preferences (skill level, age range, time of day), read fresh from the Instructor Availability Sheet at staffing time | Knowledge it reads | A table a person keeps current | A business person | When a newer record replaces it |
| 4 | Every FareHarbor email already handled, by its Gmail message ID | What happened | A table the agent writes | The agent itself | Never |
| 5 | Every lesson session and its bookings, headcount, instructors required, and status (replaces the Lesson Log tab) | What happened | A table the agent writes | The agent itself | After a date or deadline |
| 6 | Every assignment request: the instructor, the slot, the response, the deadline, and the reassignment count (replaces the Assignment Requests tab) | What happened | A table the agent writes | The agent itself | After a date or deadline |
| 7 | Which instructor last taught each repeat customer, by FareHarbor contact ID | What happened | A table the agent writes | The agent itself | When a newer record replaces it |
| 8 | Every nightly summary sent, by instructor and date, and every failure or escalation (replaces the Run Log tab) | What happened | A table the agent writes | The agent itself | After a date or deadline |
| 9 | Lessons from the Front Desk Manager’s staffing decisions and booking corrections | Lessons it learns | A lessons table the agent writes | The agent, from a trusted outcome | When a person switches it off |

## Lessons it learns

- **It learns from:** The Front Desk Manager’s decisions in the signed-in app: when they replace the instructor the agent proposed and type a reason, or correct a booking the agent misread. The agent never learns from the text of a booking email or from an instructor’s click.
- **A lesson might say:** Open group lessons with students under age 8 go to the Front Desk Manager before an instructor is asked.
- It learns only from trusted outcomes, never from text in an input. A lesson can add caution, never loosen a rule or approve anything. Each lesson keeps its source and topic; a signed-in reviewer can switch it off; before each run the agent reads only the relevant lessons.

## What must never happen twice

Processing the same FareHarbor email twice. Booking the same instructor into the same slot twice. Sending an instructor two requests for the same slot. Counting a used or expired Accept or Decline link. Sending an instructor two nightly summaries for the same date. Sending an instructor two notices about the same lesson change.
