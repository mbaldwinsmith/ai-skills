---
name: blink-patient-import
description: Import patients into Blink OMS from CSV bedlist spreadsheets using the in-app browser or Chrome. Use when the user asks Codex to add/import bedlist patients, continue patient entry, process locations in batches, or resume a Blink add-patient workflow from a spreadsheet.
---

# Blink Patient Import

## Core Workflow

Use the browser control skill/tooling to work in the live Blink tab. Treat this as a live external data-entry task: avoid duplicate submissions, verify each patient after submit, and pause when the user asks.

1. Read the CSV bedlist and preserve row order.
2. Group by contiguous `Location` blocks. Process batches of six unless the user asks for a different size.
3. Before a new `Location` block, pause for 30 seconds, tell the user the location change and the `Clinic Date` for the next entry.
4. If interrupted, inspect the current Blink patient/page or ask the user for the last successfully added patient before resuming.
5. Stop exactly where the user asks, especially at location boundaries.

## Duplicate Checks

Before adding a patient, search Blink for the full patient name. Treat search results as potential matches only until you verify the exact `First Name`, exact `Surname`, and DOB.

- Do not skip a patient for partial or fuzzy hits. Examples: `John Harrison` is not an exact match for `Malcolm John Harrison`; `Sheila Ward` is not an exact match for `Sheila Goodward` or another Sheila with a different surname.
- If Blink search returns partial matches, note them as false positives and continue adding the exact CSV patient.
- If an exact name appears but DOB differs, pause and inspect rather than assuming it is the same patient.
- If an exact name and DOB already exist, record the existing patient ID and skip creating a duplicate.

## Field Mapping

Use these CSV fields when present:

- `Title` -> `#txtTitle`
- `First Name` -> `#txtFirstname`
- `Surname` -> `#txtSurname`
- `Location` -> venue autocomplete `#txtVenue`
- `Sex/Gender` -> `#txtSex`
- `Date of Birth` -> `#datepicker`
- `GOS Eligibility` -> select with id `19`

Always set GOS Evidence Seen to `No`: radio `input[type="radio"][name="24"][value="810"]`.

For Blink venue autocomplete, type the location/home name, wait for suggestions, select the matching venue, then verify `#hdnVenueId` is not empty, `-1`, or `-2`. Do not submit if the venue did not resolve.

## DOB and GOS Timing

Blink depends on client-side scripts after DOB entry. Use the jQuery datepicker UI rather than directly setting the input where possible.

- If CSV DOB is `01/01/1900`, use `01/01/1906`.
- Use the date picker by opening `#datepicker`, selecting the year from `.ui-datepicker-year`, selecting the month from `.ui-datepicker-month` where January is value `0`, then clicking the exact day cell for that month/year.
- Match date-picker days by exact cell text and `td[data-month][data-year]`. Do not search for a loose day string such as `3`, because that can also match `13`, `23`, or `30`.
- After clicking the day, verify the live input value, not just the HTML `value` attribute.
- After selecting DOB, wait about 1 second before changing GOS.
- For patients under 65, use `Patient on Universal Credit` unless the CSV says otherwise.
- Otherwise use `Patient on Pension Credit` unless the CSV says otherwise.
- For older DOBs, Blink may auto-change GOS to `Over 60`; wait for that, then set the final GOS value.

## Submit and Verify

After filling required fields:

1. Click `#Button4`.
2. Wait for navigation or dialogs, dismiss harmless confirmation dialogs if needed.
3. Verify the resulting patient page contains the expected name, DOB, venue, GOS eligibility, and `GOS Evidence Seen: No` when visible.
4. Record the quick-search line or ID, such as `MR EXAMPLE PATIENT (ID: 1234)`.
5. Pause for 15 seconds after each patient is successfully added and verified. Use a longer final pause if requested.

If verification fails, inspect the current page before retrying. Do not assume failure means no patient was created.

## Browser Notes

For in-app browser control, connect to the existing Blink tab rather than opening a new session when possible. If the tab/session becomes stale, reconnect to the in-app browser and claim the Blink tab again.

Useful page anchors observed in Blink:

- Add patient: `/addpatient.aspx`
- Patient info after submit: `/patientinfo.aspx`
- Diary: `/diarydefault.aspx`
- Find patient: `/findpatient.aspx`

When direct navigation is blocked by the browser wrapper, use Blink menu links such as `New Patient` or the current add-patient page controls.
