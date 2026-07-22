# 🧪 AI Test Case Generator

> Turn a plain-language requirement into structured QA test cases — automatically — using n8n + OpenAI.

![n8n](https://img.shields.io/badge/Built%20with-n8n-EA4B71) ![OpenAI](https://img.shields.io/badge/AI-OpenAI%20GPT-412991) ![Google Sheets](https://img.shields.io/badge/Output-Google%20Sheets-0F9D58) ![Status](https://img.shields.io/badge/Status-Working-brightgreen)

Paste a user story like *"As a user, I can reset my password via a link sent to my email"* and this workflow generates thorough test cases — happy path, edge cases, and negative scenarios — then writes each one as a row in a Google Sheet. Work that takes a manual tester 20–30 minutes per requirement happens in seconds, in a consistent format.

---

## ✨ Demo

**Input:** a one-line requirement
**Output:** structured test cases in a spreadsheet

| title | preconditions | steps | expected_result | priority |
|---|---|---|---|---|
| Happy Path - Password Reset | User has a valid account | 1. Click Forgot Password... | Password is reset successfully | High |
| Edge Case - Reset Link Expired | User has a valid account | 1. Click Forgot Password... | Error page: link expired | Medium |
| Negative - Invalid Email | User has a valid account | 1. Enter unregistered email... | Error: email not found | High |

---

## 🔧 How it works

```
Trigger  →  Edit Fields (Set)  →  OpenAI (Message a Model)  →  Code (parse JSON)  →  Google Sheets (Append Row)
```

One requirement goes in at the start; multiple structured test-case rows come out at the end.

| Node | Role |
|---|---|
| **Trigger** | Manual (testing) or Form Trigger (shareable web URL) |
| **Edit Fields (Set)** | Holds the requirement text in a clean `requirement` field |
| **OpenAI — Message a Model** | Sends the requirement to GPT with a QA-engineer prompt |
| **Code** | Parses GPT's JSON output into one item per test case |
| **Google Sheets — Append Row** | Writes each test case as a spreadsheet row |

---

## 🧠 The prompt

The quality of the whole tool lives in this system prompt. Asking for strict JSON is what makes automated parsing possible.

```
You are a senior QA engineer. Given a software requirement or user story,
generate thorough test cases covering happy path, edge cases, and
negative/error scenarios. Return ONLY valid JSON: an array of objects with
keys "title", "preconditions", "steps" (numbered string), "expected_result",
and "priority" (High/Medium/Low). Do not include any text outside the JSON.
```

---

## 💻 The parsing code (Code node)

```javascript
// Grab the model's text output
const raw = $input.first().json.message.content;

// Clean any stray markdown fences GPT sometimes adds
let text = raw.trim()
  .replace(/^```json\s*/i, '')
  .replace(/^```\s*/, '')
  .replace(/```$/, '')
  .trim();

// Parse the JSON array and emit one item per test case
const cases = JSON.parse(text);
return cases.map(tc => ({ json: tc }));
```

---

## 🚀 Setup

1. Create a blank Google Sheet with a header row: `title`, `preconditions`, `steps`, `expected_result`, `priority`.
2. Get an OpenAI API key from [platform.openai.com](https://platform.openai.com) and add a few dollars of API credit. *(The API is separate from a ChatGPT subscription.)*
3. In n8n, add the five nodes in order: Trigger → Edit Fields → OpenAI → Code → Google Sheets.
4. Connect your OpenAI credential (OpenAI node) and Google account (Sheets node).
5. Paste the prompt and the code above.
6. Set the OpenAI model to `gpt-4o-mini` and temperature to `0.2`.
7. Map the five columns in the Sheets node and run each node one at a time.

---

## 🐛 Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| `'Column to Match On' parameter is required` | Sheets operation set to "Append or Update" | Switch operation to **Append Row** |
| Code node errors on `JSON.parse` | GPT wrapped output in markdown fences | The cleanup lines handle fences; reinforce "return only raw JSON" in the prompt |
| Inconsistent output between runs | Temperature too high | Set temperature to `0.2` |
| OpenAI node auth error | Invalid key or no credit | Check the key and confirm API billing credit exists |

---

## 🔮 Roadmap

- Swap the manual trigger for a **Form Trigger** so non-technical teammates can use it
- Add **severity** and **test type** tags (functional, UI, security)
- Push output to **Jira / TestRail / Xray** instead of just Sheets
- **Bulk mode** — generate cases for a list of requirements in one run
- **Slack/email** notification when a new batch is ready

---

## 📂 What's in this repo

- `README.md` — this file
- `workflow.json` — the exported n8n workflow *(export from n8n: ⋯ menu → Download, then add it here)*

---

*Built as a hands-on applied-AI project — using an LLM to automate a real, repetitive QA task rather than doing it by hand.*
