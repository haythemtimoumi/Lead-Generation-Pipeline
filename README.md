# Lead Generation & Qualification System

This folder contains two n8n workflows that work together to automate lead generation and qualification using AI.
## 🎥 Demo

<video src="## 🎥 Demo

<video src="workflow-demo.mp4" controls width="700"></video>" controls width="700"></video>
## Workflows

### 1. Lead Generation Pipeline
**File:** `Lead Generation Pipeline.json`

Automated system that discovers executive contacts from LinkedIn and enriches them with email data.

**Workflow:**
1. **Triggers:** Runs daily at 9 AM or manually
2. **LinkedIn Search:** Uses SerpAPI to find CEOs in tech companies in San Francisco
3. **Company Extraction:** Parses LinkedIn profiles to extract company names and domains
4. **Domain Validation:** Validates company domains before email lookup
5. **Email Discovery:** Uses Hunter.io to find executive email addresses
6. **Executive Filter:** Filters for executive-level contacts only
7. **Data Processing:** Formats contact data with company information
8. **Lead Submission:** Sends qualified leads to the AI Lead Qualification webhook

**Key Features:**
- Smart company domain extraction from LinkedIn profiles
- Domain validation to avoid false positives
- Executive-level filtering (Directors, VPs, C-suite)
- Automatic email verification
- Integration with Hunter.io for email discovery

**APIs Used:**
- SerpAPI (Google Search)
- Hunter.io (Email Discovery)

---

### 2. AI Lead Qualification
**File:** `AI Lead Qualification.json`

Intelligent lead scoring system that uses AI to qualify incoming leads and sends notifications via Telegram.

**Workflow:**
1. **Webhook Trigger:** Receives lead data via POST request at `/lead-capture`
2. **Data Cleaning:** Normalizes incoming lead data
3. **AI Analysis:** Uses HuggingFace AI model for lead scoring
4. **Smart Scoring:** Assigns scores based on company domain (Salesforce, HubSpot, Techstars, etc.)
5. **Lead Classification:** Categorizes leads as hot, warm, or cold
6. **Telegram Notifications:** 
   - Hot leads (score 80+) get priority notifications
   - Other leads get standard notifications

**Scoring Logic:**
- **Hot Leads (80-95):** Salesforce, HubSpot, Techstars executives
- **Warm Leads (60-79):** Lever and other tech company executives
- **Cold Leads (50-59):** Standard leads requiring qualification

**Key Features:**
- Real-time lead qualification
- AI-powered scoring with HuggingFace
- Company-based intelligence (extracts real company from email domain)
- Instant Telegram notifications for hot leads
- Detailed scoring reasons for each lead

**APIs Used:**
- HuggingFace (AI Model: facebook/bart-large-cnn)
- Telegram Bot API

---

## Setup Instructions

### Prerequisites
- n8n instance (cloud or self-hosted)
- API keys for:
  - SerpAPI
  - Hunter.io
  - HuggingFace
  - Telegram Bot

### Installation

1. **Import Workflows:**
   - Open n8n
   - Import `Lead Generation Pipeline.json`
   - Import `AI Lead Qualification.json`

2. **Configure Credentials:**
   - SerpAPI account
   - Hunter.io account
   - HuggingFace API token
   - Telegram Bot token and chat ID

3. **Update Webhook URL:**
   - In `Lead Generation Pipeline.json`, update the HTTP Request node with your AI Lead Qualification webhook URL
   - In `AI Lead Qualification.json`, activate the webhook to get the URL

4. **Activate Workflows:**
   - Enable both workflows
   - Test with manual trigger first

---

## Configuration

### Lead Generation Pipeline
- **Search Query:** Modify the SerpAPI query to target different roles/locations
- **Hunter.io Limit:** Adjust email discovery limit (default: 10)
- **Domain Validation:** Add/remove valid domains in the validation node

### AI Lead Qualification
- **Telegram Chat ID:** Update with your Telegram chat ID
- **Scoring Rules:** Modify company scoring in the "Parse AI" node
- **Hot Lead Threshold:** Adjust the score threshold for hot leads (default: 80)

---

## Data Flow

```
Lead Generation Pipeline → AI Lead Qualification
                              ↓
                         Telegram Notification
```

1. Pipeline searches LinkedIn for executives
2. Extracts company domains and finds emails
3. Sends qualified leads to AI webhook
4. AI scores and classifies leads
5. Sends notifications based on lead quality

---

## Customization

### Adding New Companies
Edit the `Parse AI` node in `AI Lead Qualification.json`:
```javascript
if (email.includes('@newcompany.com')) {
  realCompany = 'New Company';
  score = 85;
  status = 'hot';
  reason = 'New Company executive - high-value lead';
}
```

### Changing Search Criteria
Edit the SerpAPI query in `Lead Generation Pipeline.json`:
```
site:linkedin.com/in "CTO" "fintech" "New York"
```

---

## Monitoring

- Check n8n execution logs for errors
- Monitor Telegram for lead notifications
- Review Hunter.io API usage
- Track lead quality and conversion rates

---

## Notes

- The workflows use pinned test data for development
- Remove pinData before production use
- Respect API rate limits (especially Hunter.io)
- Ensure GDPR compliance for contact data
- Test thoroughly before running at scale
