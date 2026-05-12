# TODO: Course Export Wizard — Reverse Engineering & Completion

## Status

**Last worked on**: March 31, 2026
**Current state**: The agent scaffold, UI, orchestrator, and session keep-alive are implemented and functional. The 3-step wizard submission flow has been coded but needs refinement based on the findings below. The job polling / download mechanism is placeholder — this is the critical remaining work.

**Test environment**: `ugatest2.view.usg.edu`, OU `3550162` (SOWK7225E Course Review Testing)

---

## What Works Today

- OU ID input, validation via Valence API, and course list display
- Export tool/component selection UI with "Select All" toggle
- "Include Course Files in the Export Package" checkbox (threads to wizard Step 2)
- Session keep-alive during long batches
- Sequential batch orchestrator with per-job error handling
- Upload to host course Manage Files via Valence API (untested — export never completes)

## What Does NOT Work Yet

- The wizard form submission may not be sending the correct fields (needs DevTools verification)
- Job completion detection (polling) is entirely placeholder
- ZIP download has never executed against a real URL

---

## The Full Export Wizard Flow

The Brightspace course export is a **3-page wizard** followed by an **async background job** that delivers results through the **notification bell (Alerts)**.

### Page 1: Select Components

**URL**: `GET /d2l/lms/importExport/export/export_select_components.d2l?ou={ouId}`

This page shows checkboxes for all exportable components. **Important**: only components that actually exist in the course are displayed (e.g., if there are no quizzes, the quiz checkbox won't appear).

**What we need to do**:
- [ ] Capture the full HTML of this page (DevTools → Elements panel, or `fetch` from console)
- [ ] Identify all `<input>` elements — names, types, values, which are hidden vs. visible
- [ ] Determine the exact form `action` URL and `method`
- [ ] Identify how the "Next" / submit button triggers the POST (button name? `d2l_action`? JS click handler?)
- [ ] Verify whether unchecked tools simply omit their field or send `false`
- [ ] Test: does our current `buildStep1Form()` in `wizardAutomation.ts` produce a payload that matches a real browser submission? Compare field-by-field.

**Open question**: Since only present-in-course items appear, do we need to dynamically read which checkboxes exist on the page and only include those in the POST? Or does Brightspace ignore fields for non-existent tools?

### Page 2: Confirm & Include Course Files

**URL**: `GET /d2l/lms/importExport/export/export_select_confirm.d2l?ou={ouId}`

Reached by POSTing Page 1's form. This page shows:
- A summary of what will be exported
- A checkbox: **"Include Course Files in the Export Package"**

**What we need to do**:
- [ ] Capture the full HTML of this page
- [ ] Identify the exact field name for the "Include Course Files" checkbox (currently guessing `includeFiles` — may be wrong)
- [ ] Identify all hidden fields (new CSRF tokens for this page)
- [ ] Identify the form action and submit mechanism
- [ ] Verify: does POSTing this page directly trigger the export, or is there another intermediate step?

### Page 3: Export Started

**URL**: `GET /d2l/lms/importExport/export/export_started_information.d2l?ou={ouId}`

Reached by POSTing Page 2's form. This is a confirmation page saying the export is running.

**What we need to do**:
- [ ] Determine if this page contains any useful data (job ID, estimated time, etc.)
- [ ] Determine if we even need to parse this page, or if reaching it (HTTP 200) is sufficient confirmation that the job started
- [ ] Check if any response headers contain a job identifier

---

## Export Completion & Download

This is the **critical unknown** that blocks the entire polling mechanism.

### What We Know

When the export finishes, Brightspace adds a notification to the **Alerts** bell. Clicking the notification takes the user to a **summary page**, which contains a **download link** for the ZIP.

**Observed summary page URL**:
```
/d2l/lms/importExport/export/export_summary.d2l
  ?success=true
  &path=%2fd2l%2fcommon%2fviewFile.d2lfile%2fImmutableTemp%2f639106427767732192%2fD2LExport_3550162_sowk7225e_course_review_testing_20264136.zip%3fou%3d3550162%26fid%3dNDVkNjExOTMtNmNiNC00MGZjLTg0NGUtMzg5NWNjMDZkODA0O0QyTEV4cG9ydF8zNTUwMTYyX3Nvd2s3MjI1ZV9jb3Vyc2VfcmV2aWV3X3Rlc3RpbmdfMjAyNjQxMzYuemlwOzE2NzI3
  &ou=3550162
```

**Observed direct download URL** (from the summary page):
```
/d2l/common/viewFile.d2lfile/ImmutableTemp/639106427767732192/D2LExport_3550162_sowk7225e_course_review_testing_20264136.zip
  ?ou=3550162
  &fid=NDVkNjExOTMtNmNiNC00MGZjLTg0NGUtMzg5NWNjMDZkODA0O0QyTEV4cG9ydF8zNTUwMTYyX3Nvd2s3MjI1ZV9jb3Vyc2VfcmV2aWV3X3Rlc3RpbmdfMjAyNjQxMzYuemlwOzE2NzI3
```

### Download URL Structure Analysis

Decomposing the download URL:

| Component | Value | Notes |
|-----------|-------|-------|
| Base path | `/d2l/common/viewFile.d2lfile/ImmutableTemp/` | Standard D2L temp file serving |
| Numeric ID | `639106427767732192` | Likely a .NET DateTime tick count or internal file ID |
| Filename | `D2LExport_3550162_sowk7225e_course_review_testing_20264136.zip` | Pattern: `D2LExport_{ouId}_{courseCode}_{courseName}_{dateCode}.zip` |
| `ou` param | `3550162` | The source course OU |
| `fid` param | `NDVkNjExOTMt...` | Base64-encoded file identifier (decoding may reveal structure) |

**The `fid` parameter** is base64-encoded. Decoded value:
```
45d61193-6cb4-40fc-844e-3895cc06d804;D2LExport_3550162_sowk7225e_course_review_testing_20264136.zip;16727
```

Structure: `{GUID};{filename};{number}` — the GUID is likely a file/job identifier, the filename matches the URL path, and `16727` may be the file size in KB (~16 MB) or a sequence number. This is a composite key that Brightspace uses for temp file retrieval. We cannot construct this without getting it from the notification/summary page.

### What We Need to Investigate

- [ ] **Decode the `fid` parameter** to understand its structure (run the base64 decode above)
- [ ] **Find the notification/alerts endpoint**. When the export completes, Brightspace's JS fetches notifications. Use DevTools Network tab (XHR filter, Preserve Log) to capture:
  - What URL does the minibar poll for new alerts?
  - What is the response format (JSON? HTML fragment?)
  - Does the alert payload contain the download URL or the summary page URL?
- [ ] **Determine if we can skip the summary page** and go directly to the download URL
- [ ] **Determine if the download URL is stable** — can we fetch it multiple times, or is it single-use?
- [ ] **Determine URL expiration** — how long does the ImmutableTemp download link remain valid?

### Polling Strategy Options (in preference order)

1. **Poll the alerts/notification endpoint** — replicate what the minibar JS does, scan responses for export-related links matching the OU. This is the most reliable approach.

2. **Poll `export_summary.d2l`** — try fetching `/d2l/lms/importExport/export/export_summary.d2l?ou={ouId}` periodically; it may return the summary when ready. Unknown if this works without the full `path` query parameter.

3. **Poll the export status page** — after the export starts, revisit `export_select_components.d2l?ou={ouId}` to see if the page changes to show a "download ready" state.

4. **Construct the download URL** — if the numeric ID and `fid` follow a predictable pattern derivable from the export initiation response, we might be able to construct the URL without polling.

---

## DevTools Capture Procedure

When you return, perform this procedure to gather the remaining information:

### Preparation
1. Open Brightspace in Chrome/Edge
2. Open DevTools (F12)
3. Go to **Network** tab
4. Check **Preserve log**
5. Filter to **Fetch/XHR**
6. Clear the network log

### Step A: Capture the wizard form submissions
1. Navigate to `Course Admin → Import / Export / Copy Components → Export Components`
2. On the component selection page, check a few items and click **Next** (or equivalent)
3. In DevTools, find the POST request that was just made
4. Right-click → **Copy as cURL** (or examine Headers + Payload tabs)
5. Record: URL, method, all form fields and their values, response status, response headers
6. On the confirmation page, check "Include Course Files" and click **Export**
7. Capture that POST request the same way
8. Record the response — does it contain any job identifier?

### Step B: Capture the notification mechanism
1. After clicking Export, **do not navigate away**
2. Watch the Network tab — Brightspace's JS may start polling for notifications
3. Wait for the notification bell to show a new alert (could take 1-10 minutes)
4. Capture all XHR requests that fired during the wait period, especially:
   - Any requests to URLs containing `notification`, `alert`, `minibar`
   - Any requests that return JSON
5. When the notification appears, click it
6. Capture the request that loads the summary/download page

### Step C: Capture the download
1. On the summary page, right-click the download link → **Copy link address**
2. Record the full URL
3. In DevTools, click the download link
4. Capture the request — note response headers (Content-Type, Content-Disposition, Content-Length)
5. Try fetching the same URL again from the console — verify it works a second time:
   ```javascript
   fetch('/d2l/common/viewFile.d2lfile/ImmutableTemp/...', { credentials: 'include' })
     .then(r => console.log(r.status, r.headers.get('content-type')));
   ```

### Step D: Quick experiments from the browser console
```javascript
// 1. Decode the fid to understand its structure
atob('NDVkNjExOTMtNmNiNC00MGZjLTg0NGUtMzg5NWNjMDZkODA0O0QyTEV4cG9ydF8zNTUwMTYyX3Nvd2s3MjI1ZV9jb3Vyc2VfcmV2aWV3X3Rlc3RpbmdfMjAyNjQxMzYuemlwOzE2NzI3');

// 2. Probe notification endpoints (run each and note which return 200)
for (const url of [
  '/d2l/MiniBar/Desktop/GetAlerts',
  '/d2l/minibar/notifications/alerts',
  '/d2l/api/lp/1.54/notifications/instant/',
  '/d2l/lms/notifications/alerts/',
  '/d2l/minibar/notifications',
]) {
  fetch(url, { credentials: 'include' })
    .then(r => console.log(`${r.status} ${url}`))
    .catch(e => console.log(`ERR ${url}`, e.message));
}

// 3. Check if export_summary.d2l works without the path parameter
fetch('/d2l/lms/importExport/export/export_summary.d2l?ou=3550162', { credentials: 'include' })
  .then(r => { console.log('Status:', r.status); return r.text(); })
  .then(html => console.log(html.substring(0, 1000)));
```

---

## Code Files to Update

Once the investigation is complete, these files need updates:

| File | What Needs to Change |
|------|---------------------|
| `services/wizardAutomation.ts` | Correct form field names/values for Step 1 and Step 2 POSTs based on actual DevTools capture. May need to dynamically read which components are available on the page. |
| `services/jobPoller.ts` | Replace placeholder `checkExportStatus()` with the real notification polling endpoint and download URL extraction logic. |
| `types.ts` | May need additional fields in `ExportSubmissionResult` if the wizard returns a job ID or tracking token. |
| `services/exportOrchestrator.ts` | Minimal changes expected — mostly needs the poller and wizard to work correctly. |

---

## File Locations

All agent code is at:
```
frontend/src/agents/course-export/
├── config.ts                    # Agent plugin definition
├── index.ts                     # Public exports
├── entry.tsx                    # React bootstrap
├── entry.html                   # HTML entry point
├── types.ts                     # All TypeScript types
├── .env                         # APP_SHARED_BASE config
├── styles/
│   └── course-export.css        # Agent-specific styles
├── data/
│   └── tools.json               # Empty (no LLM tools)
├── services/
│   ├── wizardAutomation.ts      # 3-step wizard form logic
│   ├── jobPoller.ts             # Completion detection + download
│   ├── exportOrchestrator.ts    # Batch lifecycle manager
│   ├── sessionKeepAlive.ts      # Periodic whoami ping
│   └── courseValidator.ts       # OU ID validation via Valence
└── components/
    ├── MainView.tsx             # Step orchestrator
    ├── CourseInput.tsx           # Step 1: OU ID input + validation
    ├── ExportConfig.tsx         # Step 2: Tool selection + include files
    ├── ExportDashboard.tsx      # Step 3: Progress monitoring
    └── ExportResults.tsx        # Step 4: Summary table
```

---

## Prior Conversation Context

The full design and implementation history is in the agent transcript:
- [Course Export Agent](cfa3be41-85ba-418b-8f7e-caab31a763f8) — contains the original application specification, all implementation decisions, troubleshooting (wizard URL corrections, missing field handling), and the current state of each service module.
