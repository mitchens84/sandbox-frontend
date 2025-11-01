# n8n Podcast Transcription Workflow with AI Analysis

This n8n workflow automatically transcribes podcast audio files using AssemblyAI and processes them with OpenAI for fact-checking and content distillation when triggered by a webhook from Airtable.

## Overview

**Workflow:** `podcast-transcription-workflow.json`

**Trigger:** Webhook (HTTP POST)

**Actions:**
1. Receives webhook trigger from Airtable
2. Retrieves podcast record from Airtable
3. Extracts audio file URL from the record
4. Submits audio to AssemblyAI for transcription
5. Polls AssemblyAI until transcription is complete
6. Writes transcription back to Airtable record
7. **Processes transcription with OpenAI GPT-4o for:**
   - Content distillation and summarization
   - Fact-checking with flagged claims
   - Key topics and insights extraction
   - Actionable items identification
   - Notable quotes extraction
8. Writes AI analysis back to Airtable
9. Responds to webhook with success/error status

## Prerequisites

- n8n instance (self-hosted or cloud)
- AssemblyAI API key
- OpenAI API key
- Airtable API key (Personal Access Token)
- Airtable base with a table containing:
  - A field for the audio file URL (e.g., "AudioFileURL")
  - A field for storing the transcription (e.g., "Transcription")
  - A field for storing the AI analysis (e.g., "AIAnalysis")
  - A field for storing the webhook URL (optional)

## Installation

### Step 1: Import Workflow into n8n

1. Log into your n8n instance
2. Click on **Workflows** in the left sidebar
3. Click **Import from File** or **Import from URL**
4. Select the `podcast-transcription-workflow.json` file
5. The workflow will be imported and opened in the editor

### Step 2: Configure AssemblyAI Credentials

The workflow uses HTTP Request nodes with header authentication for AssemblyAI. You need to replace the placeholder with your actual API key:

1. Obtain your AssemblyAI API key from: https://www.assemblyai.com/app/account
2. In the n8n workflow, find the following nodes:
   - **Submit to AssemblyAI**
   - **Check Transcription Status**
3. For each node, click to edit it
4. Scroll to the **Headers** section
5. Find the "Authorization" header
6. Replace `ASSEMBLYAI_API_KEY_PLACEHOLDER` with your actual AssemblyAI API key
7. Click **Save**

**Format:** Your API key should be entered as-is (e.g., `your_api_key_here`)

### Step 3: Configure Airtable Credentials

1. In n8n, go to **Credentials** in the left sidebar
2. Click **Add Credential**
3. Search for and select **Airtable Personal Access Token API**
4. Enter a name (e.g., "Airtable API")
5. Enter your Airtable Personal Access Token
   - Get your token from: https://airtable.com/create/tokens
   - Required scopes: `data.records:read`, `data.records:write`
6. Click **Save**
7. Copy the credential ID (you'll see it in the URL or credential list)

### Step 4: Configure OpenAI Credentials

1. In n8n, go to **Credentials** in the left sidebar
2. Click **Add Credential**
3. Search for and select **OpenAI API**
4. Enter a name (e.g., "OpenAI API")
5. Enter your OpenAI API key
   - Get your key from: https://platform.openai.com/api-keys
6. Click **Save**
7. Copy the credential ID (you'll see it in the URL or credential list)

In the n8n editor:
1. Click on the **OpenAI Fact-Check & Content Distillation** node
2. In the **Credential to connect with** dropdown, select your OpenAI credential
3. Click **Save**

### Step 5: Update Workflow with Airtable Credential ID

1. Open the workflow JSON file in a text editor
2. Find all instances of `"id": "AIRTABLE_CREDENTIAL_ID"` (there are 3 instances now)
3. Replace `AIRTABLE_CREDENTIAL_ID` with your actual credential ID from Step 3
4. Find the instance of `"id": "OPENAI_CREDENTIAL_ID"`
5. Replace `OPENAI_CREDENTIAL_ID` with your actual OpenAI credential ID from Step 4
6. Save the file
7. Re-import the workflow into n8n (or manually update the credential selection in the nodes)

Alternatively, in the n8n editor:
1. Click on the **Get Podcast Record** node
2. In the **Credential to connect with** dropdown, select your Airtable credential
3. Click **Save**
4. Repeat for the **Update Airtable with Transcription** and **Update Airtable with AI Analysis** nodes

### Step 6: Activate Workflow and Get Webhook URL

1. In the n8n workflow editor, click **Activate** (toggle switch in top right)
2. Click on the **Webhook** node
3. Copy the **Production URL** (it will look like: `https://your-n8n-instance.com/webhook/podcast-transcribe`)
4. This is your webhook URL to use in Airtable

## Airtable Setup

### Required Fields in Your Airtable Table

Your Airtable table should have the following fields:

| Field Name | Field Type | Description |
|------------|------------|-------------|
| AudioFileURL | URL or Text | The URL of the podcast audio file to transcribe |
| Transcription | Long text | Will store the transcription result (auto-populated) |
| AIAnalysis | Long text | Will store the AI analysis with fact-checking and insights (auto-populated) |
| WebhookURL | URL or Text | Optional: Store the webhook URL for triggering transcription |

**Note:** You can use different field names, but you'll need to update the workflow accordingly (see Customization section below).

### Field Mapping in Workflow

The workflow expects the following field names by default:
- **Audio URL field:** `AudioFileURL` (referenced in "Submit to AssemblyAI" node)
- **Transcription field:** `Transcription` (referenced in "Update Airtable with Transcription" node)
- **AI Analysis field:** `AIAnalysis` (referenced in "Update Airtable with AI Analysis" node)

If your field names differ, update them in the workflow nodes.

## Triggering the Workflow

The workflow expects a webhook POST request with the following JSON payload:

```json
{
  "baseId": "appXXXXXXXXXXXXXX",
  "tableId": "tblXXXXXXXXXXXXXX",
  "recordId": "recXXXXXXXXXXXXXX"
}
```

### Webhook Payload Parameters

- **baseId**: Your Airtable base ID (starts with `app`)
- **tableId**: Your Airtable table ID (starts with `tbl`)
- **recordId**: The specific record ID to transcribe (starts with `rec`)

### Example cURL Request

```bash
curl -X POST https://your-n8n-instance.com/webhook/podcast-transcribe \
  -H "Content-Type: application/json" \
  -d '{
    "baseId": "appYourBaseId",
    "tableId": "tblYourTableId",
    "recordId": "recYourRecordId"
  }'
```

### Automating Webhook Triggers from Airtable

To automatically trigger the webhook when a record is created or updated, you can:

1. **Use Airtable Automations:**
   - Go to your Airtable base
   - Click **Automations** in the top toolbar
   - Create a new automation with a trigger (e.g., "When record matches conditions")
   - Add an action: "Run script" or "Send webhook request"
   - Configure it to send a POST request to your webhook URL with the required payload

2. **Use a button field:**
   - Add a formula field to construct the webhook URL with parameters
   - Use Airtable's automation or external tools to trigger on button click

### Example Airtable Automation Script

```javascript
// In Airtable automation "Run script" action
let table = base.getTable("Your Table Name");
let record = input.config(); // The triggering record

// Send webhook request
let webhookUrl = "https://your-n8n-instance.com/webhook/podcast-transcribe";

let response = await fetch(webhookUrl, {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify({
        baseId: "appYourBaseId",
        tableId: table.id,
        recordId: record.recordId
    })
});

console.log("Webhook triggered:", response.status);
```

## Workflow Behavior

### Transcription and AI Processing

1. **Webhook receives request** with record details
2. **Airtable record is fetched** using the provided IDs
3. **Audio URL is extracted** from the `AudioFileURL` field
4. **AssemblyAI transcription is submitted** with the audio URL
5. **Polling loop begins:**
   - Waits 5 seconds
   - Checks transcription status
   - If still processing, waits another 5 seconds and checks again
   - Continues until status is "completed" or "error"
6. **Transcription saved to Airtable:**
   - Transcription text is written to the `Transcription` field
7. **AI analysis with OpenAI GPT-4o:**
   - Processes the transcription for fact-checking and content distillation
   - Generates structured JSON with:
     - Summary (2-3 sentence overview)
     - Key topics identified
     - Main insights extracted
     - Fact-check flags (claims needing verification)
     - Actionable items
     - Notable quotes with context
8. **AI analysis saved to Airtable:**
   - AI analysis JSON is written to the `AIAnalysis` field
9. **Webhook responds** with success/error JSON

### AI Analysis Output Format

The OpenAI node generates a structured JSON response saved to the `AIAnalysis` field:

```json
{
  "summary": "Brief 2-3 sentence summary of the podcast",
  "key_topics": ["topic1", "topic2", "topic3"],
  "main_insights": [
    "Insight 1 from the discussion",
    "Insight 2 about the subject matter"
  ],
  "fact_check_flags": [
    {
      "claim": "Specific claim made in the podcast",
      "reason": "Why this claim needs verification"
    }
  ],
  "actionable_items": [
    "Action item 1",
    "Action item 2"
  ],
  "notable_quotes": [
    {
      "quote": "Exact quote from the podcast",
      "context": "Context or speaker information"
    }
  ]
}
```

This structured format makes it easy to:
- Display summaries in Airtable views
- Create automations based on topics or insights
- Build dashboards with key metrics
- Export data for further analysis

### Response Format

**Success Response:**
```json
{
  "success": true,
  "message": "Transcription and AI analysis completed",
  "recordId": "recXXXXXXXXXXXXXX"
}
```

**Error Response:**
```json
{
  "success": false,
  "message": "Transcription failed",
  "status": "error",
  "error": "Error details from AssemblyAI"
}
```

## Customization

### Change Field Names

If your Airtable fields have different names:

1. **For the audio URL field:**
   - Edit the **Submit to AssemblyAI** node
   - Find the `audio_url` parameter
   - Change `={{ $json.fields.AudioFileURL }}` to `={{ $json.fields.YourFieldName }}`

2. **For the transcription field:**
   - Edit the **Update Airtable with Transcription** node
   - Find the `fieldId` parameter
   - Change `"Transcription"` to `"YourFieldName"`

3. **For the AI analysis field:**
   - Edit the **Update Airtable with AI Analysis** node
   - Find the `fieldId` parameter
   - Change `"AIAnalysis"` to `"YourFieldName"`

### Adjust Polling Interval

By default, the workflow checks transcription status every 5 seconds:

1. Edit the **Wait 5 Seconds** node
2. Change the `amount` value (currently 5)
3. You can also change the `unit` to minutes, hours, etc.

### Customize OpenAI Analysis Prompt

To modify what the AI analyzes, edit the **OpenAI Fact-Check & Content Distillation** node:

1. Click on the node in the workflow editor
2. Find the **System Message** in the messages section
3. Modify the prompt to focus on different aspects:

**Example customizations:**

**For marketing focus:**
```
You are a marketing analyst. Analyze this podcast for:
1. Target audience insights
2. Marketing angles and hooks
3. Key value propositions
4. Memorable soundbites for social media
5. Content repurposing opportunities
```

**For educational content:**
```
You are an educational content curator. Extract:
1. Key learning objectives
2. Concepts explained
3. Examples and case studies mentioned
4. Questions raised for further research
5. Resources or references mentioned
```

**For research/academic:**
```
You are a research analyst. Identify:
1. Research questions discussed
2. Methodologies mentioned
3. Data points and statistics
4. Citations and sources
5. Areas for further investigation
```

You can also adjust:
- **Temperature** (0.0-1.0): Lower = more focused, higher = more creative
- **Max Tokens**: Maximum length of the response (currently 2000)
- **Model**: Change from `gpt-4o` to `gpt-4o-mini` for faster/cheaper processing

### Add Additional AssemblyAI Features

AssemblyAI supports many additional features. To enable them, edit the **Submit to AssemblyAI** node and add parameters:

```json
{
  "audio_url": "...",
  "speaker_labels": true,
  "auto_chapters": true,
  "entity_detection": true,
  "sentiment_analysis": true
}
```

See [AssemblyAI API documentation](https://www.assemblyai.com/docs) for all available options.

## Troubleshooting

### Webhook doesn't trigger
- Verify the workflow is activated (toggle switch is ON)
- Check the webhook URL is correct
- Ensure the payload format matches the expected structure

### Airtable authentication fails
- Verify your Airtable Personal Access Token is valid
- Ensure the token has the required scopes (`data.records:read`, `data.records:write`)
- Check that the baseId and tableId in the webhook payload are correct

### AssemblyAI authentication fails
- Verify your AssemblyAI API key is correct in both HTTP Request nodes
- Ensure the API key is entered without extra quotes or spaces

### Transcription doesn't appear in Airtable
- Check that the `Transcription` field exists in your Airtable table
- Verify the field name matches exactly (case-sensitive)
- Look at the n8n execution log to see the exact error

### Audio file not found
- Ensure the `AudioFileURL` field contains a valid, publicly accessible URL
- AssemblyAI must be able to download the file from the URL
- Supported formats: MP3, MP4, WAV, FLAC, AAC, OGG, and more

### OpenAI authentication fails
- Verify your OpenAI API key is correct and active
- Ensure you have sufficient credits in your OpenAI account
- Check that the API key has permissions to use the GPT-4o model

### AI analysis not appearing in Airtable
- Check that the `AIAnalysis` field exists in your Airtable table
- Verify the field name matches exactly (case-sensitive)
- Ensure the field type is "Long text" to accommodate JSON
- Check the OpenAI node execution log for any errors

## Testing the Workflow

### Test in n8n

1. Open the workflow in n8n editor
2. Click on the **Webhook** node
3. Click **Listen for Test Event**
4. Send a test webhook request using the example cURL command above
5. Watch the workflow execute in real-time
6. Check each node's output to verify data flow

### Test with Real Airtable Data

1. Create a test record in your Airtable table
2. Add a valid audio URL to the `AudioFileURL` field
3. Trigger the webhook with the test record's ID
4. Wait for the transcription to complete
5. Verify the `Transcription` field is populated

## Support

- **n8n Documentation:** https://docs.n8n.io
- **AssemblyAI Documentation:** https://www.assemblyai.com/docs
- **Airtable API Documentation:** https://airtable.com/developers/web/api/introduction

## Notes

- **Transcription time** depends on audio file length (typically 15-30% of audio duration)
- **AssemblyAI pricing** is based on audio duration (check their pricing page)
- **Webhook timeout:** n8n webhooks may timeout for very long transcriptions; consider using asynchronous patterns for files longer than 30 minutes
- **Rate limits:** AssemblyAI has rate limits; for high-volume usage, consider adding error handling and retry logic
