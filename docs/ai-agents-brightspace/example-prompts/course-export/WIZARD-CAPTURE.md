# Export Wizard — DevTools Capture

## Status: PENDING

This document must be completed before `wizardAutomation.ts` and `jobPoller.ts` can be finalized.

## Instructions

1. Open a Brightspace course where you have Export permissions
2. Open browser DevTools → Network tab
3. Enable **Preserve Log** and filter to **XHR/Fetch**
4. Navigate to **Course Admin → Import / Export / Copy Components → Export Components**
5. Walk through the wizard normally, selecting "Export All Components"
6. After clicking the final submit, watch the Network tab for polling or status requests
7. When the notification bell shows the export is ready, capture the download URL

## Capture Checklist

### Wizard Steps

- [ ] How many form POSTs does the wizard require? (1, 2, 3?)
- [ ] URL of each wizard step page
- [ ] For each step: list all hidden form fields and their values

### Step 1 — Select Components Page

```
URL: /d2l/lms/importexport/export_export.d2l?ou={ouId}
Method: GET → returns HTML

Hidden fields found:
  d2l_hitCode: ???
  d2l_referrer: ???
  d2l_controlMap: ???
  d2l_controlMapPrev: ???
  d2l_state: ???
  d2l_stateScopes: ???
  d2l_statePageId: ???
  d2l_action: ???
  d2l_actionparam: ???
  (list all)
```

### Step 2 — Confirmation? (if exists)

```
URL: ???
Method: POST → returns ???

Additional hidden fields or changes:
  (document here)
```

### Final Submission Response

```
Response status: ???
Response body type: HTML / JSON / redirect?
Contains job ID? ???
Contains filename? ???
Redirect URL (if any): ???
```

### Completion Detection

- [ ] Does the wizard's own JavaScript poll an endpoint? If so:
  - Endpoint URL: ???
  - Poll interval: ???
  - Response format: ???
  - How does it signal "done"?

- [ ] Does a notification appear? If so:
  - What endpoint delivers the notification?
  - What is the notification HTML structure?
  - Does it contain a direct download link?

### Download URL

```
Full download URL: ???
Pattern: ???
Is it session-bound? ???
Expires? ???
```

### Additional Observations

(Any other network requests, cookies, headers, or behaviors observed)
