# X Post Agent: Human-in-the-Loop Automation (n8n)

An n8n workflow that researches a topic, drafts a post for X (Twitter), and **asks for your approval by email before anything is published**. If you want changes, an AI revision agent rewrites the post and sends it back for another review. The loop repeats until you approve, and only then does the workflow post to X.

Built entirely on free tiers: Groq for the LLMs, Tavily for web search, Gmail for approvals, and the X API for publishing.

---

## Workflow Overview

<img width="1143" height="489" alt="Human in the Loop" src="https://github.com/user-attachments/assets/0f6f971c-ebe2-49a5-8479-1fabd40a385c" />


### How it works

1. **Form trigger:** you enter a topic and pick a tone (Professional, Casual, Witty, Educational).
2. **X Post Agent:** an AI agent searches the web with Tavily and writes one post of about 200 characters, with a hard cap below X's 280-character limit.
3. **Set Post:** stores the draft and its character count.
4. **Gmail (Send and Wait):** emails you the draft and pauses the workflow until you respond through a link in the email.
5. **Text Classifier:** reads your reply and decides whether it means **approved** (for example "go", "looks good", "post it") or **declined** (any change request, including "good, but remove the hashtags").
6. **Approved:** the exact approved text is published to X.
7. **Declined:** the Revision Agent applies your feedback, either making a small edit or writing a new post, and the loop returns to step 3.

Ambiguous replies are always treated as **declined**, so nothing is published without a clear approval.

---

## Features

- Human approval before every post, with no accidental publishing
- Unlimited revision rounds in natural language ("make it funnier", "shorter", "use a different angle")
- Web research for factual, current posts, and the agent is told never to invent statistics or quotes
- Tone selection from the form
- Safe classification: unclear replies fall back to revision rather than publishing
- Runs on free tiers

---

## Tech Stack

| Component | Purpose |
|---|---|
| [n8n](https://n8n.io) | Workflow orchestration |
| [Groq](https://console.groq.com) | LLM inference for the agents and classifier |
| [Tavily](https://tavily.com) | Web search tool for the agents |
| Gmail (OAuth2) | Approval emails (Send and Wait) |
| X (Twitter) API | Publishing posts |

---

## Prerequisites

- An n8n instance (cloud or self-hosted)
- A Groq API key
- A Tavily API key
- A Google account with the Gmail API enabled
- An X developer account with an app that has **Read and Write** permissions

---

## Setup

### 1. Import the workflow
In n8n choose **Workflows → Import from File** and select `x_post_hitl_workflow.json`.

The file contains **no credentials or API keys**. You connect your own after import.

### 2. Create credentials

| Node(s) | Credential type | Notes |
|---|---|---|
| Groq Chat Model (x3) | Groq API | API key from the Groq console |
| Tavily Search, Tavily Search1 | Header Auth | Name: `Authorization`, Value: `Bearer YOUR_TAVILY_KEY` |
| Gmail | Gmail OAuth2 | Enable the Gmail API in Google Cloud and add yourself as a test user |
| X Post | X OAuth2 | App permissions must be Read and Write |

### 3. Configure the nodes
- **Gmail:** replace `YOUR_EMAIL@example.com` with the address that should receive approval requests.
- **Groq Chat Model nodes:** open the model dropdown and pick a currently available chat model. Groq's model list changes over time, so check the live list in the node. The workflow was tested with `openai/gpt-oss-120b` for the two agents and `openai/gpt-oss-20b` for the classifier.

### 4. Activate and run
Activate the workflow, open the form's production URL, and submit a topic.

---

## Usage

1. Submit a topic and tone in the form.
2. Open the approval email and click the button to respond.
3. In the form that opens, type either:
   - `go` (or any clear approval) to publish, or
   - your feedback, such as `make it shorter and remove the emoji`, to get a revised draft.
4. Repeat until you approve. The post is then published to X.

---

## Node Reference

| Node | Type | Role |
|---|---|---|
| On form submission | Form Trigger | Collects topic and tone |
| X Post Agent | AI Agent (Tools) | Researches and writes the first draft |
| Set Post | Set | Normalizes the draft (`post`, `charCount`) |
| Gmail | Gmail, Send and Wait | Requests approval with free-text response |
| Text Classifier1 | Text Classifier | Routes to `approved` or `declined` |
| X Post | X (Twitter) | Publishes the approved text from Set Post |
| Revision Agent | AI Agent (Tools) | Edits or rewrites the draft using your feedback |

---

## Customization Ideas

- Change the character target in both agents' system messages
- Add or edit tones in the form dropdown
- Add an IF node to enforce `charCount <= 280` before sending the approval email
- Log every topic, draft, and final post to Google Sheets or Airtable
- Send a confirmation email after posting
- Swap Gmail for Slack or Telegram approvals
- Add a Schedule trigger to propose posts from a topic list

---

## Important Notes

- **Reachable URL:** the approval link in the email works only if your n8n instance is reachable from the device where you open it. For local installs, set `WEBHOOK_URL` or use a tunnel.
- **Wait time:** set a limit on the Gmail node so unanswered approvals do not wait forever.
- **API limits:** Groq, Tavily, and X all enforce free-tier limits. Check their current limits before heavy use.
- **Model changes:** if a model disappears from Groq, select another one from the node's dropdown. No other change is needed.

---

## Security

- No API keys, tokens, or email addresses are stored in the workflow file.
- Credentials live only in your n8n credential store.
- Do not commit exported workflows that contain credential IDs or personal data.

---

## Author

**Ansar Hayat**: AI automation and agentic AI developer
GitHub: [@ansarhayat9](https://github.com/ansarhayat9)
