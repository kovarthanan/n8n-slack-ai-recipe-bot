# 🍛 Slack AI Recipe Bot (n8n + OpenAI)

An n8n workflow that turns a Slack channel into a recipe assistant. Post a dish name in the channel, and the bot replies in the same channel with a South Indian–style ingredient list with approximate quantities.

![Workflow canvas](screenshots/workflow-canvas.png)

---

## 🔄 How It Works

```
Slack Trigger → If (ignore bot messages) → Basic LLM Chain → Slack: Send a message
                                                 ↑
                                         OpenAI Chat Model
```

| Step | Node | What it does |
|------|------|--------------|
| 1 | **Slack Trigger** | Receives every new message posted in `#ai-test` |
| 2 | **If** | Lets only human messages through; the bot's own replies are dropped (`bot_id` and `subtype` must be empty) |
| 3 | **Basic LLM Chain** + **OpenAI Chat Model** (`gpt-5-mini`) | Lists the products and ingredients needed for the dish, South Indian style |
| 4 | **Send a message** | Posts the answer back to the same Slack channel |

### Example

**In Slack:**
```
Paneer Butter Masala
```

**Bot reply:**
```
- Paneer — 250–300 g, cubed
- Butter — 3–4 tbsp
- Onion — 1 large, finely chopped
- Tomatoes — 3 medium or 1 cup puree
- Ginger-garlic paste — 1 tbsp
- Cashews — 10–12, soaked and ground
...
```

---

## 📁 Project Structure

```
├── workflow/
│   └── slack-ai-recipe-bot.json   # n8n workflow export
├── screenshots/
│   ├── workflow-canvas.png
│   └── slack-reply.png
├── README.md
└── .gitignore
```

---

## 🚀 Setup

### 1. Import the workflow
In n8n, go to **Workflows → ⋯ → Import from File** and select `workflow/slack-ai-recipe-bot.json`.

### 2. Create the Slack app
1. Go to [api.slack.com/apps](https://api.slack.com/apps) → **Create New App** → **From scratch**.
2. **OAuth & Permissions → Bot Token Scopes:** add `channels:history`, `channels:read` and `chat:write`.
3. Click **Install to Workspace** and copy the **Bot User OAuth Token** (starts with `xoxb-`).
4. In n8n, create a **Slack API** credential with that token and select it in both Slack nodes.

### 3. Connect Slack events to n8n
1. In the **Slack Trigger** node, copy the webhook URL (**Test URL** while testing, **Production URL** when live).
2. In your Slack app, open **Event Subscriptions**, turn it on and paste the URL into **Request URL**. It must show **Verified ✓**. (For the test URL, click **Listen for test event** in n8n first.)
3. Under **Subscribe to bot events**, add `message.channels` and save. Reinstall the app if Slack asks.

### 4. Add the bot to the channel
In the Slack channel, send `/invite @YourBotName`. Then select that channel in both the **Slack Trigger** and **Send a message** nodes.

### 5. OpenAI
Create an API key at [platform.openai.com/api-keys](https://platform.openai.com/api-keys) (the full key is shown only once) and add it as an **OpenAI** credential in n8n.

### 6. Go live
Switch the Slack app's Request URL to the n8n **Production URL**, save the workflow and turn it **Active**.

---

## 🔁 Preventing the Reply Loop

The bot's reply is also a new message in the channel, so without a guard the Slack Trigger fires again and the bot keeps answering itself forever.

The **If** node only passes messages where:
- `{{ $json.bot_id }}` **is empty**, and
- `{{ $json.subtype }}` **is empty**

Human messages have neither field, so they reach the AI. Bot messages have them, so they stop at the If node.

An alternative check: `{{ $json.client_msg_id }}` **is not empty**. Slack adds `client_msg_id` only to messages a person types in the Slack app.

---

## 🐞 Troubleshooting

| Problem | Fix |
|---------|-----|
| Request URL not verified | Click **Listen for test event** in n8n before pasting the test URL, or use the Production URL with the workflow active |
| Trigger never fires | Subscribe to `message.channels`, reinstall the app, and invite the bot to the channel |
| Replies posted as your user, not the bot | Use the **Bot** token (`xoxb-`), not the User token (`xoxp-`) |
| Bot keeps replying to itself | Add the **If** node above; make sure only its **true** output connects to the LLM |
| `not_in_channel` / `missing_scope` | Add `chat:write`, reinstall the app, and `/invite` the bot |
| Bot asks follow-up questions | Each message is handled on its own (no memory), so tell the prompt to answer directly |

---

## 🔒 Security

- Tokens and API keys live only in **n8n Credentials**; the exported JSON contains just credential names and IDs, which are not secrets.
- Never commit `.env` files or paste keys into node fields.

---

## 🛠️ Tech Stack

- [n8n](https://n8n.io) – workflow automation
- Slack Events API – input and output channel
- OpenAI (`gpt-5-mini`) – recipe generation

---

## 🙏 Acknowledgements

Built while learning AI automation with **Social Eagle**.
