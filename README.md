# No-LLM Inbox Assistant (Gmail + Jev + Composio)

**LLMs generate, Jev decides.** A beginner-friendly Colab notebook that sorts Gmail into **Urgent**, **Reply Later**, **Newsletter** and **Spam**, sends unsure emails to **Review**, and archives newsletters and scams, without a large language model.

Companion code for the Agentic AI Hub video *Stop Paying GPT to Sort Your Email: Let's Build a No-LLM Inbox Assistant (Gmail + Jev + Composio)*.

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/davolu/no-llm-inbox-assistant/blob/main/inbox_assistant.ipynb)

## Who does what

| Part | Job | Writes text? |
|---|---|---|
| **Python** | Reads each email: sender domain, subject, snippet, unsubscribe header, contact, deadline words | No |
| **[Jev](https://docs.typesafe.ai/sdk/python)** (TypeSafe System One) | **The part that replaces the LLM.** Answers three typed questions per email in one `system_one` call: category, action, and "clear enough to sort?", each with a confidence | **Never** |
| **Python gate** | Low confidence or unclear → `Jev/Review`; urgent and reply-later are never archived | No |
| **Composio** (optional) | Easy Gmail access: list/create labels, add a label, archive | No |

### Don't want Composio?

Composio is here only for easy Gmail access (one OAuth link, no Google Cloud project). You can call the Gmail API directly instead:

| What | Composio tool | Gmail API |
|---|---|---|
| list labels | `GMAIL_LIST_LABELS` | `users.labels.list` |
| create a label | `GMAIL_CREATE_LABEL` | `users.labels.create` |
| label / archive | `GMAIL_ADD_LABEL_TO_EMAIL` | `users.messages.modify(addLabelIds=[...], removeLabelIds=["INBOX"])` |

Replace the `apply()` function in the notebook with `service.users().messages().modify(...)` from `google-api-python-client`. Jev and everything else stay the same.

## Safety

- `DRY_RUN=True` by default: nothing in Gmail changes until you set it to `False`.
- Runs on an **18-email fictional demo inbox** by default (all senders use `.example` domains). Live Gmail is opt-in (`INBOX_LIVE=true`), read-only until `DRY_RUN=False`, and masks senders and subjects in the output.
- **Nothing is ever deleted.** "Archive" only removes the `INBOX` label; the email stays in All Mail. There is no delete or trash call in the code.
- Keys come from Colab Secrets or environment variables and are never printed.

## Run it

1. [Open in Colab](https://colab.research.google.com/github/davolu/no-llm-inbox-assistant/blob/main/inbox_assistant.ipynb) (`inbox_assistant.ipynb`).
2. Add `TYPESAFE_API_KEY` in Colab Secrets (key icon) and enable notebook access. Add `COMPOSIO_API_KEY` only if you want live Gmail.
3. Run all cells. The demo inbox needs no Gmail account.

Locally: `pip install -r requirements.txt`, copy `.env.example` to `.env`, fill in your keys, and export them before starting Jupyter.

## Results from the saved run

The notebook in this repo is saved with the outputs of a real run on the demo inbox (`DRY_RUN=True`):

- Jev's final routes matched the hand labels on **18 of 18** emails; the 2 deliberately vague emails went to **Review**.
- 8 emails (5 newsletters, 3 scams) were planned for archiving.
- Median Jev call: **50 ms** (max 111 ms). Tokens for the whole inbox: **12,231 in / 1,794 out**, and no generated text.

It's a small, hand-made demo set. Test on your own inbox with dry run on before you trust it. Results can vary slightly between runs.

## License

MIT
