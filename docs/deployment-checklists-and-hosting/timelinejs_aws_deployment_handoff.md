# TimelineJS + AWS Deployment Handoff

## Goal
Replace Knight Lab TimelineJS's Google Sheet backend with an AWS backend:

- **DynamoDB** stores timeline data
- **Lambda** reads DynamoDB and transforms it into TimelineJS JSON
- **API Gateway** exposes a public JSON endpoint
- **TimelineJS** loads the JSON from the API URL

---

## Current working status

The backend is working.

Confirmed:
- Lambda test returns `statusCode: 200`
- API Gateway endpoint returns valid JSON in the browser
- `events` and `title` are both present in the JSON response
- DynamoDB data is being read correctly

The API response currently looks like this shape:

```json
{
  "events": [
    {
      "start_date": { "year": "1066", "month": "10", "day": "14" },
      "text": {
        "headline": "Battle of Hastings",
        "text": "William the Conqueror defeats Harold II."
      },
      "display_date": "October 14, 1066",
      "group": "Norman Conquest",
      "media": {
        "url": "https://upload.wikimedia.org/...jpg",
        "caption": "Scene from the Bayeux Tapestry",
        "credit": "Public Domain"
      }
    }
  ],
  "title": {
    "text": {
      "headline": "The Norman Conquest",
      "text": "A TimelineJS timeline backed by DynamoDB"
    }
  }
}
```

---

## AWS resources in use

## Lambda
- **Function name:** `timeline-get`
- **Region:** `us-east-1`

### Lambda environment variables
```text
EVENTS_TABLE=timeline_events
META_TABLE=timeline_meta
```

### Important note about regions
Lambda is in `us-east-1`, but DynamoDB tables are in `us-east-2`.

Because of that, the Lambda code explicitly sets the DynamoDB client region to `us-east-2`.

If I later move Lambda into `us-east-2`, I can remove that explicit region override.

---

## DynamoDB

### Table 1: `timeline_events`
Schema:
- **Partition key:** `timeline_id` (String)
- **Sort key:** `event_id` (String)

Example item:

```json
{
  "timeline_id": "medieval-knights",
  "event_id": "1066-battle-of-hastings",
  "sort_order": 10,
  "start_year": 1066,
  "start_month": 10,
  "start_day": 14,
  "headline": "Battle of Hastings",
  "text": "William the Conqueror defeats Harold II.",
  "display_date": "October 14, 1066",
  "group": "Norman Conquest",
  "media_url": "https://upload.wikimedia.org/wikipedia/commons/6/6a/Bayeux_Tapestry_scene57_Harold_death.jpg",
  "media_caption": "Scene from the Bayeux Tapestry",
  "media_credit": "Public Domain",
  "published": true
}
```

Another example:

```json
{
  "timeline_id": "medieval-knights",
  "event_id": "1066-coronation",
  "sort_order": 20,
  "start_year": 1066,
  "start_month": 12,
  "start_day": 25,
  "headline": "Coronation of William I",
  "text": "William is crowned King of England at Westminster Abbey.",
  "display_date": "December 25, 1066",
  "group": "Norman Conquest",
  "published": true
}
```

### Table 2: `timeline_meta`
Schema:
- **Partition key:** `timeline_id` (String)

Example item:

```json
{
  "timeline_id": "medieval-knights",
  "title_headline": "The Norman Conquest",
  "title_text": "A TimelineJS timeline backed by DynamoDB"
}
```

---

## IAM permissions for Lambda role
The Lambda execution role needs permission to:

- `dynamodb:Query` on `timeline_events`
- `dynamodb:GetItem` on `timeline_meta`

Policy shape:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["dynamodb:Query"],
      "Resource": "arn:aws:dynamodb:us-east-2:<ACCOUNT_ID>:table/timeline_events"
    },
    {
      "Effect": "Allow",
      "Action": ["dynamodb:GetItem"],
      "Resource": "arn:aws:dynamodb:us-east-2:<ACCOUNT_ID>:table/timeline_meta"
    }
  ]
}
```

Replace `<ACCOUNT_ID>` with the real AWS account ID if recreating or documenting this elsewhere.

---

## Lambda code currently in use

```javascript
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, QueryCommand, GetCommand } from "@aws-sdk/lib-dynamodb";

const client = new DynamoDBClient({ region: "us-east-2" });
const ddb = DynamoDBDocumentClient.from(client);

const EVENTS_TABLE = process.env.EVENTS_TABLE;
const META_TABLE = process.env.META_TABLE;

function buildDate(year, month, day) {
  if (year == null) return undefined;
  const d = { year: String(year) };
  if (month != null) d.month = String(month);
  if (day != null) d.day = String(day);
  return d;
}

function toTimelineSlide(item) {
  const slide = {
    start_date: buildDate(item.start_year, item.start_month, item.start_day),
    text: {
      headline: item.headline || "",
      text: item.text || ""
    }
  };

  const endDate = buildDate(item.end_year, item.end_month, item.end_day);
  if (endDate) slide.end_date = endDate;
  if (item.display_date) slide.display_date = item.display_date;
  if (item.group) slide.group = item.group;

  if (item.media_url) {
    slide.media = {
      url: item.media_url
    };
    if (item.media_caption) slide.media.caption = item.media_caption;
    if (item.media_credit) slide.media.credit = item.media_credit;
  }

  return slide;
}

export const handler = async (event) => {
  try {
    const timelineId =
      event?.pathParameters?.timelineId ||
      event?.queryStringParameters?.timeline_id ||
      "medieval-knights";

    const [metaResp, eventsResp] = await Promise.all([
      ddb.send(
        new GetCommand({
          TableName: META_TABLE,
          Key: { timeline_id: timelineId }
        })
      ),
      ddb.send(
        new QueryCommand({
          TableName: EVENTS_TABLE,
          KeyConditionExpression: "timeline_id = :tid",
          ExpressionAttributeValues: {
            ":tid": timelineId
          }
        })
      )
    ]);

    const events = (eventsResp.Items || [])
      .filter((x) => x.published !== false)
      .sort((a, b) => (a.sort_order ?? 999999) - (b.sort_order ?? 999999))
      .map(toTimelineSlide);

    const body = { events };

    if (metaResp.Item) {
      body.title = {
        text: {
          headline: metaResp.Item.title_headline || "",
          text: metaResp.Item.title_text || ""
        }
      };

      if (metaResp.Item.title_media_url) {
        body.title.media = {
          url: metaResp.Item.title_media_url
        };
        if (metaResp.Item.title_media_caption) {
          body.title.media.caption = metaResp.Item.title_media_caption;
        }
        if (metaResp.Item.title_media_credit) {
          body.title.media.credit = metaResp.Item.title_media_credit;
        }
      }
    }

    return {
      statusCode: 200,
      headers: {
        "Content-Type": "application/json",
        "Access-Control-Allow-Origin": "*"
      },
      body: JSON.stringify(body)
    };
  } catch (error) {
    console.error(error);

    return {
      statusCode: 500,
      headers: {
        "Content-Type": "application/json",
        "Access-Control-Allow-Origin": "*"
      },
      body: JSON.stringify({
        error: "Failed to load timeline",
        details: error.message
      })
    };
  }
};
```

---

## API Gateway
An existing API for the project was reused.

A new route was added for TimelineJS:

```text
GET /timeline/{timelineId}
```

This route is integrated with Lambda:

```text
timeline-get
```

Working browser endpoint pattern:

```text
https://<API_ID>.execute-api.us-east-1.amazonaws.com/timeline/medieval-knights
```

The endpoint is already returning valid JSON in the browser.

---

## TimelineJS frontend usage
Use the API URL directly in TimelineJS:

```html
<link rel="stylesheet" href="https://cdn.knightlab.com/libs/timeline3/latest/css/timeline.css">
<script src="https://cdn.knightlab.com/libs/timeline3/latest/js/timeline.js"></script>

<div id="timeline-embed" style="width: 100%; height: 600px"></div>

<script>
  new TL.Timeline(
    "timeline-embed",
    "https://<API_ID>.execute-api.us-east-1.amazonaws.com/timeline/medieval-knights"
  );
</script>
```

---

## Known cleanup items before deployment

### 1. Remove debug logs if still present
Temporary logs were added during debugging and should be removed if they remain:

```javascript
console.log("metaResp.Item:", JSON.stringify(metaResp.Item));
console.log("response body:", JSON.stringify(body));
```

### 2. Region alignment
Current setup works, but is not ideal because:
- Lambda is in `us-east-1`
- DynamoDB is in `us-east-2`

For production, consider moving everything into the same region.

### 3. CORS
API Gateway should allow TimelineJS to fetch the JSON.
For testing, permissive CORS is acceptable:
- origin: `*`
- method: `GET`
- header: `Content-Type`

For production, lock CORS down to the actual frontend domain.

### 4. API URL source of truth
Use the real API Gateway invoke URL copied from AWS, not a manually typed URL.

### 5. Data validation
Consider validating required event fields before returning JSON.
At minimum:
- `timeline_id`
- `event_id`
- `start_year`
- `headline`

### 6. Sorting
Events are currently sorted in Lambda using `sort_order`.
That is fine for now.

---

## Suggested production refinements

### Infrastructure
- move Lambda to `us-east-2` or recreate all resources in one region
- manage infrastructure with Terraform, CDK, or SAM
- store API and table names in IaC, not just console config

### Security
- tighten CORS to the deployed frontend domain
- consider API auth if the timeline should not be public
- least-privilege IAM only for the exact tables and actions needed

### Reliability
- add structured logging
- add better error messages
- add unit tests for JSON transformation
- optionally add API Gateway caching or CloudFront

### Content management
- create write endpoints for admin editing:
  - `POST /timeline/{timelineId}/events`
  - `PUT /timeline/{timelineId}/events/{eventId}`
  - `DELETE /timeline/{timelineId}/events/{eventId}`
- alternatively build an admin UI instead of editing DynamoDB manually

---

## Cursor handoff tasks
If refining this in Cursor, likely next tasks are:

1. Extract Lambda into a proper project structure
2. Remove temporary debugging logs
3. Add schema validation for event/meta items
4. Add deployment config (SAM / Terraform / CDK)
5. Add a README with setup instructions
6. Add CORS and stage config notes for API Gateway
7. Add local tests for:
   - empty timeline
   - timeline with title only
   - timeline with events only
   - timeline with media
8. Decide whether to keep cross-region setup or consolidate regions

---

## Final note
The backend is already working end-to-end. The remaining work is mostly deployment hardening, cleanup, and making the setup reproducible outside the AWS console.

