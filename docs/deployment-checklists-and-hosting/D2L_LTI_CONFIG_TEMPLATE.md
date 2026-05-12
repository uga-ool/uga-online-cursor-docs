# D2L LTI Configuration Template

Use this template when creating/updating the External Learning Tool launch for the Gamification Widget Agent.

## Tool identity
- Name: `Gamification Widget Agent`
- Description: `Framework-based gamification activity authoring tool`

## Launch URL
- Colocated course files URL:
  - `<course-files-url>/gamification-widget-agent.html`
- Shared assets mode:
  - Same HTML URL, with `app-shared-base` set to shared assets folder in the HTML.

## Security/settings
- Open in iframe: `Enabled`
- Send course context: `Enabled`
- Send user identity claims: `Enabled`
- Third-party cookies: `Allowed` (if required by your org settings)

## Post-save verification
- Tool appears in target org unit/course.
- Launch opens HTML and loads assets.
- Network calls to backend proxy succeed from D2L origin.
