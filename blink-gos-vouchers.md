---
name: blink-gos-vouchers
description: Automate Blink/EliteSight GOS voucher report workflows in Chrome. Use when working on elitesight.blinkoms.co.uk/vouchersummaryreport.aspx to process GOS6 patient rows by opening new form tabs, ticking Not known, setting blank Approx Arrival Time to 17:00, saving, closing form/closetab tabs, refreshing with the report Search button, or ticking all Request PVN checkboxes after forms are processed.
---

# Blink GOS Vouchers

Use this skill for the two recurring EliteSight HomeCare voucher tasks on `https://elitesight.blinkoms.co.uk/vouchersummaryreport.aspx`.

Prefer Chrome for this workflow. If the user explicitly asks for the in-app browser, follow that instruction, but Chrome is the default because the form workflow opens new tabs and is more stable there. The user normally preloads the correct date and care home, so do not change search criteria unless explicitly asked.

## Component 1: Process Pending GOS6 Forms

Goal: turn every pending GOS6 row into `Request PVN`.

Workflow:

1. Claim the visible Chrome report tab.
2. Click the report Search button `#Button4` to refresh the list. Do not use the header quick-search button.
3. Read rows from `#ResultsContainer tr`.
4. Treat rows whose final/status cell contains `Request PVN` as done.
5. For the first row not containing `Request PVN`, click its `GOS6` link.
6. If an “already updated” style popup appears, accept/dismiss it, return to the report, click Search, and continue.
7. On the new Chrome form tab:
   - Tick `#gos6_chk_Not_Known`.
   - Inspect the select controls. If the Approx Arrival Time select is blank and has a `17:00` option, select `17:00` before saving.
   - Click `#aSaveGOS`.
   - Accept/dismiss the alert popup.
   - Close the form/closetab tab if it remains open.
8. Return to the report tab and click `#Button4` before reading rows again.
9. Continue in small batches, usually 4-6 patients per tool call in Chrome. Drop to 2-3 if tabs, dialogs, or refreshes become slow.
10. Stop only when the refreshed report shows zero rows without `Request PVN`.

Operational notes:

- The report may briefly show a just-saved patient as pending. If one patient repeats, refresh once; if it still repeats, skip it temporarily and continue down the list, then revisit at the end.
- A repeated sticky row is often caused by blank Approx Arrival Time. Reopen the form and set the blank time to `17:00`.
- If the report tab shows `ERR_CONNECTION_TIMED_OUT`, pause and ask the user to reload or confirm when the page is back.
- If a form tab is left open after a timeout, finish that form first using the same rules, then close it and refresh the report.

## Component 2: Tick Request PVN Checkboxes

Goal: select every checkbox beside rows showing `Request PVN`.

Workflow:

1. Claim the visible Chrome report tab.
2. Find all rows in `#ResultsContainer tr` whose text contains `Request PVN`.
3. For each row, find its `input[type="checkbox"]`.
4. Use normal locator actions such as `setChecked(true)` by checkbox id. Do not rely on page evaluation to set `checked`; this site exposes checkbox properties that can reject direct assignment.
5. Verify that every `Request PVN` row checkbox is checked.
6. Report the total checked count and any unchecked rows, if any.

## Browser Automation Pattern

Prefer small helper functions in the active browser-control session:

- `rows()`: extract `{ href, patient, status }` from `#ResultsContainer tr` rows containing a `GOS6` link.
- `search()`: click `#Button4`, wait briefly, then call `rows()`.
- `dlg(tab)`: poll briefly for a JS dialog and accept/dismiss it.
- `waitForm(href)`: in Chrome, find either a controlled tab or a user-opened tab whose URL contains the GOS form URL, `/printGOSform.aspx`, or `/closetab.aspx`, then claim it if necessary.
- `closeExtras()`: close non-report form/closetab Chrome tabs after handling dialogs. Leave the report tab open and finalize it as a handoff when finished.

Keep user updates concise: mention current done/pending counts, sticky rows, blank-time fixes, and completion.

