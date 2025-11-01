# Credentials Setup Guide

This document provides a quick reference for all the credentials and IDs you need to replace in the workflow.

## Required Credentials

### 1. AssemblyAI API Key

**Where to get it:**
- Sign up at https://www.assemblyai.com
- Navigate to https://www.assemblyai.com/app/account
- Copy your API key

**Where to use it:**
Replace `ASSEMBLYAI_API_KEY_PLACEHOLDER` in the following nodes:
- **Submit to AssemblyAI** node → Headers → Authorization
- **Check Transcription Status** node → Headers → Authorization

**Format:** `your_actual_api_key_here`

**Example:**
```
Before: ASSEMBLYAI_API_KEY_PLACEHOLDER
After:  a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
```

---

### 2. Airtable Personal Access Token

**Where to get it:**
- Go to https://airtable.com/create/tokens
- Click "Create new token"
- Give it a name (e.g., "n8n Podcast Transcription")
- Add scopes:
  - `data.records:read`
  - `data.records:write`
- Add access to your specific base
- Click "Create token"
- Copy the token (starts with `pat...`)

**Where to use it:**
Create a credential in n8n:
1. Go to n8n → Credentials
2. Add "Airtable Personal Access Token API"
3. Paste your token
4. Save and note the credential ID

---

### 3. Airtable Credential ID

**Where to get it:**
After creating the Airtable credential in n8n (see above), the ID will be shown in the credential URL or list.

**Where to use it:**
Replace `AIRTABLE_CREDENTIAL_ID` in the workflow JSON at these locations:
- **Get Podcast Record** node → credentials → airtableTokenApi → id
- **Update Airtable with Transcription** node → credentials → airtableTokenApi → id

**Format:** Numeric ID (e.g., `1`, `42`, `123`)

**Example:**
```json
Before:
"credentials": {
  "airtableTokenApi": {
    "id": "AIRTABLE_CREDENTIAL_ID",
    "name": "Airtable API"
  }
}

After:
"credentials": {
  "airtableTokenApi": {
    "id": "1",
    "name": "Airtable API"
  }
}
```

---

### 4. Airtable Base ID

**Where to get it:**
- Open your Airtable base in a browser
- Look at the URL: `https://airtable.com/appXXXXXXXXXXXXXX/tblYYYYYYYYYYYYYY`
- The part starting with `app` is your Base ID

**Where to use it:**
This is passed in the webhook payload when triggering the workflow.

**Format:** Starts with `app` followed by alphanumeric characters

**Example:** `appYourBaseId123`

---

### 5. Airtable Table ID

**Where to get it:**
- Open your Airtable table in a browser
- Look at the URL: `https://airtable.com/appXXXXXXXXXXXXXX/tblYYYYYYYYYYYYYY`
- The part starting with `tbl` is your Table ID

**Where to use it:**
This is passed in the webhook payload when triggering the workflow.

**Format:** Starts with `tbl` followed by alphanumeric characters

**Example:** `tblYourTableId123`

---

## Quick Setup Checklist

- [ ] Get AssemblyAI API key from https://www.assemblyai.com/app/account
- [ ] Replace `ASSEMBLYAI_API_KEY_PLACEHOLDER` in 2 HTTP Request nodes
- [ ] Create Airtable Personal Access Token at https://airtable.com/create/tokens
- [ ] Add Airtable credential in n8n with the token
- [ ] Note the Airtable credential ID
- [ ] Replace `AIRTABLE_CREDENTIAL_ID` in the workflow (or select credential in UI)
- [ ] Get your Airtable Base ID from the URL
- [ ] Get your Airtable Table ID from the URL
- [ ] Create required fields in Airtable: `AudioFileURL`, `Transcription`
- [ ] Activate the workflow in n8n
- [ ] Copy the webhook URL from the Webhook node
- [ ] Test the workflow with a sample record

---

## Webhook Trigger Example

Once everything is set up, trigger the workflow with:

```bash
curl -X POST https://your-n8n-instance.com/webhook/podcast-transcribe \
  -H "Content-Type: application/json" \
  -d '{
    "baseId": "appYourActualBaseId",
    "tableId": "tblYourActualTableId",
    "recordId": "recYourActualRecordId"
  }'
```

Replace:
- `your-n8n-instance.com` with your actual n8n instance URL
- `appYourActualBaseId` with your Airtable base ID
- `tblYourActualTableId` with your Airtable table ID
- `recYourActualRecordId` with the specific record you want to transcribe

---

## Verification

After setup, verify each component:

1. **AssemblyAI API Key**: Test with a simple cURL request
   ```bash
   curl https://api.assemblyai.com/v2/transcript \
     -H "Authorization: YOUR_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{"audio_url": "https://example.com/audio.mp3"}'
   ```

2. **Airtable Token**: Test by viewing a record
   ```bash
   curl "https://api.airtable.com/v0/YOUR_BASE_ID/YOUR_TABLE_ID" \
     -H "Authorization: Bearer YOUR_PERSONAL_ACCESS_TOKEN"
   ```

3. **Webhook**: Send a test request and watch the n8n execution log

---

## Common Issues

**"Unauthorized" error with AssemblyAI:**
- Check that you copied the full API key
- Ensure there are no extra spaces or quotes
- Verify your AssemblyAI account is active

**"Invalid credentials" with Airtable:**
- Ensure the token has the required scopes
- Verify the token has access to the specific base
- Check that the Base ID and Table ID are correct

**Workflow doesn't trigger:**
- Ensure the workflow is activated (toggle ON)
- Check the webhook URL is correct
- Verify the payload format matches the expected structure
