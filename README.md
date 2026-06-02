# 🗺️ Data From Maps — Automated Google Maps Lead Scraper with n8n

> Collect business leads from Google Maps automatically — no coding required. Built with n8n, Apify, and AI.

---

## 📌 What Is This Project?

**Data From Maps** is a no-code automation workflow that scrapes business data directly from **Google Maps** and exports it to a **Google Sheets spreadsheet** — fully automated.

### Who Is This For?

- 🧳 **Salespeople** entering a new territory who need local business contacts fast
- 📍 **Travelers or newcomers** wanting to discover places in an unfamiliar city
- 🏢 **Marketers and agencies** building lead lists at scale
- 🤖 **Automation enthusiasts** looking for a practical n8n workflow example

---

## ✨ Key Features

- **Form-based input** — just fill out a simple web form to start a scrape
- **AI-powered query builder** — uses an AI Agent to format your search parameters perfectly
- **Google Maps data** via the Apify Compass Crawler
- **Exports to Google Sheets** automatically
- **Customizable fields** — contacts, social media, emails, place details, and more
- **No coding required** — everything is done visually inside n8n

---

## 🛠️ Tools & Technologies Used

| Tool | Purpose |
|---|---|
| [n8n](https://n8n.io) | Workflow automation platform (the engine) |
| [Apify](https://apify.com) | Google Maps crawler (data source) |
| Groq / Gemini AI | AI model for formatting query parameters |
| Google Sheets | Output destination for scraped data |

---

## 🧩 Workflow Architecture

The workflow is made up of the following n8n nodes, in order:

```
[On Form Submission]
        ↓
   [AI Agent]  ←  Groq/Gemini Chat Model
        ↓         Structured Output Parser
 [HTTP Request]  ← POST to Apify (start scrape)
        ↓
     [Wait]  ← 60 seconds (let Apify run)
        ↓
 [HTTP Request]  ← GET dataset results from Apify
        ↓
   [Split Out]  ← Extract key fields
        ↓
    [Limit]  ← (optional) cap results
        ↓
 [Loop Over Items]
        ↓
 [Google Sheets]  ← Append rows
```

---

## 🚀 Step-by-Step Setup Guide

### Prerequisites

Before you start, make sure you have:

- ✅ A free [n8n](https://n8n.io) account (cloud or self-hosted)
- ✅ A free [Apify](https://apify.com) account
- ✅ A Google account (for Google Sheets)
- ✅ A free [Groq](https://console.groq.com) or Gemini API key

---

### Step 1 — Create a New Workflow in n8n

Log into your n8n instance and click **"New Workflow"**.

---

### Step 2 — Add the Form Trigger Node

Press **CTRL + V** anywhere on the canvas and paste the following JSON. This will automatically create the form trigger with all the correct fields configured:

<details>
<summary>📋 Click to expand — Form Trigger JSON</summary>

```json
{
  "nodes": [
    {
      "parameters": {
        "formTitle": "Details for scraping",
        "formFields": {
          "values": [
            {
              "fieldLabel": "How many searches do you want",
              "fieldType": "number",
              "requiredField": true
            },
            {
              "fieldLabel": "Do you want contacts?",
              "fieldType": "checkbox",
              "fieldOptions": {
                "values": [{ "option": "Yes" }, { "option": "No" }]
              },
              "limitSelection": "exact",
              "requiredField": true
            },
            {
              "fieldLabel": "Do you want place details",
              "fieldType": "checkbox",
              "fieldOptions": {
                "values": [{ "option": "Yes" }, { "option": "No" }]
              },
              "limitSelection": "exact",
              "requiredField": true
            },
            {
              "fieldLabel": "Do you want social media? If yes, check the boxes of the social media you want.",
              "fieldType": "checkbox",
              "fieldOptions": {
                "values": [
                  { "option": "Facebook" },
                  { "option": "Instagram" },
                  { "option": "Tiktok" },
                  { "option": "X(twitter)" },
                  { "option": "Youtube" }
                ]
              }
            },
            {
              "fieldLabel": "Do you want E-mails?",
              "fieldType": "checkbox",
              "fieldOptions": {
                "values": [{ "option": "Yes" }, { "option": "No" }]
              },
              "limitSelection": "exact",
              "requiredField": true
            },
            { "fieldLabel": "Which City?", "requiredField": true },
            { "fieldLabel": "Which country?", "requiredField": true },
            { "fieldLabel": "Which Category", "requiredField": true }
          ]
        },
        "options": { "appendAttribution": false }
      },
      "type": "n8n-nodes-base.formTrigger",
      "typeVersion": 2.5,
      "position": [-16, 256],
      "id": "(Your own ID)",
      "name": "On form submission",
      "webhookId": "(Your own Webhook ID)"
    }
  ],
  "connections": {
    "On form submission": { "main": [[]] }
  },
  "pinData": {},
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": "20cf75cd0417fe5be3c3d8dcf7163bcd4dd5f0947193527bf3d55f850ac76577"
  }
}
```

</details>

> 💡 **Alternatively:** You can manually add an **"On Form Submission"** node and recreate the fields by hand — but copy-pasting the JSON is faster and error-free.

---

### Step 3 — Add the AI Agent Node

Add an **AI Agent** node to your canvas and connect it after the form trigger.

---

### Step 4 — Choose Your AI Model

Inside the AI Agent node, attach a **Chat Model**. Recommended free options:

- **Groq** (fast, generous free tier) — recommended ⭐
- **Gemini** (Google's free tier)

Connect your API key as a credential inside the node.

---

### Step 5 — Set the Prompt Source

Inside the AI Agent node:

1. Find the **"Source for Prompt (User Message)"** dropdown
2. Select **"Define below"**
3. Paste this into the **Prompt (User Message)** field:

```
{{ $json.data }}
```

---

### Step 6 — Enable Structured Output

Still inside the AI Agent node:

1. Enable **"Require Specific Output Format"**
2. Click **"Add option"** → choose **"System Message"**
3. Paste the following system prompt:

```
Role: You are a specialized Data Extraction Assistant designed to output structured JSON for automated workflows in n8n.

Objective: Extract and format information into a flat JSON object. You must strictly follow the schema provided below.

Constraint Checklist:

No Arrays: You are forbidden from using square brackets []. If multiple items exist (like cities), list them as separate string keys or a single comma-separated string.

String Only: Every value must be a string. Do not use integers, booleans, or nulls.

Format: Output valid, minified, or pretty-printed JSON only. No conversational filler or markdown explanations.

Boolean rule: If any answer is yes, use true in JSON. Otherwise false. If you don't have a response for a specific boolean, use false in that.

Language: Always set the language to English
```

---

### Step 7 — Configure the Structured Output Parser

Open the **Structured Output Parser** node (attached to the AI Agent) and paste this JSON into the **"JSON Example"** field:

```json
{
  "includeWebResults": false,
  "language": "en",
  "locationQuery": "Dhaka, Bangladesh",
  "maxCrawledPlacesPerSearch": 10,
  "maximumLeadsEnrichmentRecords": 0,
  "scrapeContacts": true,
  "scrapeDirectories": false,
  "scrapeImageAuthors": false,
  "scrapeOrderOnline": false,
  "scrapePlaceDetailPage": true,
  "scrapeReviewsPersonalData": true,
  "scrapeSocialMediaProfiles": {
    "facebooks": true,
    "instagrams": true,
    "tiktoks": true,
    "twitters": true,
    "youtubes": true
  },
  "scrapeTableReservationProvider": false,
  "searchStringsArray": ["Restaurants"],
  "skipClosedPlaces": false,
  "verifyLeadsEnrichmentEmails": false
}
```

Then enable **"Auto-Fix Format"**.

> 💡 The model attached to the Structured Output Parser does **not** need to match the AI Agent's model. Any model works here.

---

### Step 8 — Get Your Apify API Endpoints

1. Log into your [Apify Console](https://console.apify.com)
2. Go to **API → API Endpoints**
3. Copy the URLs for:
   - **Run Actor** (you'll use `POST`)
   - **Get Last Run Dataset Items** (you'll use `GET`)

---

### Step 9 — Add the First HTTP Request Node (Start the Scrape)

Add an **HTTP Request** node and configure it:

- **Method:** `POST`
- **URL:** *(paste your "Run Actor" URL from Apify)*
- **Send Body:** ✅ Enabled
- **Body Content Type:** JSON
- **Body:**

```json
{
  "includeWebResults": false,
  "language": "en",
  "locationQuery": "{{ $json.output.locationQuery }}",
  "maxCrawledPlacesPerSearch": {{ $json.output.maxCrawledPlacesPerSearch }},
  "maximumLeadsEnrichmentRecords": 0,
  "scrapeContacts": {{ $json.output.scrapeContacts }},
  "scrapeDirectories": {{ $json.output.scrapeDirectories }},
  "scrapeImageAuthors": {{ $json.output.scrapeImageAuthors }},
  "scrapeOrderOnline": {{ $json.output.scrapeOrderOnline }},
  "scrapePlaceDetailPage": {{ $json.output.scrapePlaceDetailPage }},
  "scrapeReviewsPersonalData": {{ $json.output.scrapeReviewsPersonalData }},
  "scrapeSocialMediaProfiles": {
    "facebooks": {{ $json.output.scrapeSocialMediaProfiles.facebooks }},
    "instagrams": {{ $json.output.scrapeSocialMediaProfiles.instagrams }},
    "tiktoks": {{ $json.output.scrapeSocialMediaProfiles.tiktoks }},
    "twitters": {{ $json.output.scrapeSocialMediaProfiles.twitters }},
    "youtubes": {{ $json.output.scrapeSocialMediaProfiles.youtubes }}
  },
  "scrapeTableReservationProvider": false,
  "searchStringsArray": [
    "{{ $json.output.searchStringsArray[0] }}"
  ],
  "skipClosedPlaces": {{ $json.output.skipClosedPlaces }},
  "verifyLeadsEnrichmentEmails": {{ $json.output.verifyLeadsEnrichmentEmails }}
}
```

---

### Step 10 — Add a Wait Node

Add a **Wait** node and set the duration to **60 seconds**.

> This gives Apify time to finish crawling Google Maps before your workflow tries to fetch the results.

---

### Step 11 — Add the Second HTTP Request Node (Fetch Results)

Add another **HTTP Request** node:

- **Method:** `GET`
- **URL:** *(paste your "Get Last Run Dataset Items" URL from Apify)*

---

### Step 12 — Add a Split Out Node

Add a **Split Out** node and enter the following into **"Fields to Split Out"**:

```
title, address, postalCode, phoneUnformatted
```

---

### Step 13 — Add a Limit Node *(Optional)*

Add a **Limit** node and set the maximum to `20`.

> This is optional — skip it if you want all results returned.

---

### Step 14 — Add the Loop Over Items Node

Add a **Loop Over Items** node to iterate through each scraped business.

---

### Step 15 — Add the Google Sheets Node

Inside the Loop Over Items node, replace the default "Replace Me" node with a **Google Sheets** node:

- **Credential:** Connect your Google account
- **Action:** `Append Row`
- **Document:** Select your target spreadsheet
- **Sheet:** Select the sheet tab

Then map the columns as follows:

| Column | Value |
|---|---|
| Name | `{{ $('Loop Over Items').item.json.title }}` |
| Postal Code | `{{ $json.postalCode }}` |
| Address | `{{ $('Loop Over Items').item.json.address }}` |
| Phone Number | `{{ $json.phoneUnformatted }}` |

---

## ✅ Your Workflow Is Ready!

Once everything is connected, activate your workflow. Submit the form with your search criteria, and the automation will:

1. Parse your input with AI
2. Send the query to Apify
3. Wait for Google Maps data to be collected
4. Extract the results
5. Append each business as a new row in your Google Sheet

---

## 🙋 FAQ

**Q: Is this free to use?**
Both n8n (self-hosted) and Apify offer free tiers. Groq and Gemini also have free API credits. You can run this workflow at zero cost for light usage.

**Q: How many results can I get?**
That depends on your Apify plan. The `maxCrawledPlacesPerSearch` parameter controls how many places are fetched per search.

**Q: Can I add more output columns?**
Yes! The Apify dataset contains many more fields (website, rating, reviews, etc.). You can add more columns to Google Sheets and map the corresponding Apify fields.

**Q: Why the 60-second wait?**
Apify runs the crawler asynchronously. The wait gives the crawler enough time to finish before your workflow tries to retrieve the data.

---

## 📎 Related Resources

- [n8n Documentation](https://docs.n8n.io)
- [Apify Google Maps Scraper](https://apify.com/compass/crawler-google-places)
- [Groq Free API](https://console.groq.com)
- [n8n Community Forum](https://community.n8n.io)

---

## 📄 License

This project is open for personal and commercial use. Attribution appreciated but not required.

---

*Built with ❤️ using n8n, Apify, and AI — no code required.*
