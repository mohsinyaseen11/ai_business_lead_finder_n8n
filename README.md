# AI Business Lead Finder (n8n + Gemini)

An n8n workflow that finds small e-commerce businesses (USA, UK, Canada), scores them as potential customers for a website chatbot, saves them to Google Sheets, and drafts and sends a short outreach email.

## How it works

1. **Schedule Trigger** runs daily at 21:00.
2. **Tavily search** picks one of 15 niche queries (e.g. small independent clothing, skincare, home decor stores) and returns up to 20 results. Large marketplaces and social sites are excluded.
3. **HTTP Request + HTML extract** fetches each site and pulls headings, meta description, email links, contact/about/product/FAQ links, social links and chat-widget scripts.
4. **Gemini (LLM chain)** scores each lead from 1 to 10 and returns structured JSON (business type, country, size, chatbot need, emails, phone, suggested use case).
5. **Filter:** only leads with score >= 5 and at least one email continue.
6. **Google Sheets** appends or updates the lead (matched on Website URL).
7. **Duplicate check:** only rows where "Message Sent" is empty continue.
8. **Gemini** writes one short, plain outreach email (subject + body) using strict rules about what the chatbot may claim.
9. **Gmail** sends the email, then the sheet row is marked "Sent".

## Tech

n8n, Tavily Search, Google Gemini (LangChain nodes), Google Sheets, Gmail.

## Setup

1. Import `AI_Business_Lead_Finder_International.json` into n8n.
2. Create credentials and select them in the nodes: Tavily API, Google Gemini (PaLM) API, Google Sheets OAuth2, Gmail OAuth2.
3. Create a Google Sheet with these columns: Business Name, Website URL, Country, Email, Message Sent, Business Type, Business Size, Lead Score, Qualification Status, Phone, WhatsApp Link, Chatbot Present, Chatbot Need, Suggested Use Case, Score Reason, Website Evidence.
4. Put your Sheet ID in the three Google Sheets nodes (replace `YOUR_GOOGLE_SHEET_ID`).
5. Edit the sender name and signature in the email prompt and the Gmail node.
6. Test manually before activating. Consider replacing the Gmail "Send" node with "Create draft" until you trust the output.

## Known limitations

- The scoring prompt reads `phone_links`, `whatsapp_links`, `chat_elements` and `chat_iframes`, but the HTML node does not extract them yet (it extracts `chat_widgets`, `chat_scripts`, `chat_links`). Those fields arrive empty.
- Static HTML only: chat widgets or emails loaded by JavaScript are not detected.
- Emails are saved to the sheet as an array string.
- The workflow sends real emails. Review the output, respect each recipient's right to opt out, and follow the anti-spam rules for your region.
- Search results are random per run, so duplicates across days are possible (handled by the "Message Sent" check).

## Disclaimer

Credentials, sheet IDs and personal example data were removed. Do not commit your own API keys or credentials.
