# n8n Workflow: Author Royalty Notification & AI Summary

This workflow handles the automated generation of personalized royalty summary emails for BookLeaf authors.

---

## Importing the Workflow

1. Open your n8n workspace (Cloud or self-hosted).
2. Go to Workflows, click the + button or use the menu to select Import from File.
3. Choose `n8n_royalty_notification_workflow.json` from this folder.
4. All 8 nodes will appear connected and ready to configure.

---

## Configuration

Set these variables in n8n (Settings > Variables or your .env file):

| Variable | What It Does | Example |
| :--- | :--- | :--- |
| `OPENAI_API_KEY` | API key for GPT-4o-mini | `sk-proj-...` |
| `BOOKLEAF_WEBHOOK_SECRET` | Shared secret for authenticating requests | `bookleaf_secure_token_2026` |
| `BUBBLE_API_URL` | Bubble endpoint for logging notifications | `https://<app>.bubbleapps.io/api/1.1/obj/notification_log` |

If the OpenAI key is missing or runs out of credits, the workflow automatically uses a structured fallback template with the exact numbers instead of failing.

---

## How the Workflow Runs

```
[1. Webhook Trigger]
        │
        ▼
[2. Security & Dedup Check]  ← validates API key, blocks duplicates within 60s
        │
        ▼
[3. Data Aggregation]  ← computes totals, flags overdue and in-production titles
        │
        ▼
[4. Prompt Builder]  ← injects BookLeaf policies (80/20 split, 45-day payout cycle)
        │
        ▼
[5. OpenAI GPT-4o-mini]  ← generates personalized email
        │
        ▼
[6. Parse & Fallback]  ← validates response, uses template if AI fails
        │
        ▼
[7. Log to Bubble]  ← writes status, timestamp, and email content back
        │
        ▼
[8. Return Success]  ← sends 200 OK with preview to the Bubble UI
```

---

## Sample Webhook Payload

This is what Bubble sends to n8n when an admin clicks "Generate & Send":

```json
{
  "author_id": "AUTH001",
  "name": "Priya Sharma",
  "email": "priya.sharma@email.com",
  "city": "Mumbai",
  "books": [
    {
      "title": "Whispers of the Ganges",
      "status": "Published & Live",
      "mrp": 399,
      "total_copies_sold": 342,
      "total_royalty_earned": 11970,
      "royalty_paid": 8400,
      "royalty_pending": 3570,
      "last_royalty_payout_date": "2025-10-15"
    }
  ]
}
```
