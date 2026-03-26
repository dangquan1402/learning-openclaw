# Channel-Agnostic HITL Design

> **Question:** If a system runs in a terminal it can render rich options and step-by-step prompts, but how should it handle human-in-the-loop interactions on channels like Telegram?

The best answer is not "terminal mode" versus "Telegram mode." It is:

> one structured HITL model in the backend, with different renderers per channel.

## Three practical patterns

## 1. Plain text only

The simplest approach is to send a normal message:

- describe the decision
- list the options
- ask the user to reply with a keyword or number

Example:

- `Choose an action:`
- `1. Approve`
- `2. Review first`
- `3. Cancel`

This works everywhere, but it is the weakest UX and easiest to misparse.

## 2. Channel-native interaction

When the channel supports structured UI, use it:

- terminal: interactive prompts
- Telegram: inline keyboards
- Discord/Slack: buttons, selects, dialogs

This is the best UX for:

- yes/no
- pick-one choices
- short approval flows

But it should not be the only path, because channel support differs.

## 3. Hybrid model

This is the best real-world design:

- send a text explanation
- attach structured controls if the channel supports them
- always keep a text fallback

Example:

- message text: `Choose an action: Approve, Review, or Cancel.`
- buttons: `Approve`, `Review`, `Cancel`
- fallback: `Reply with APPROVE / REVIEW / CANCEL if buttons are unavailable.`

## What the backend should store

The backend should not treat this as raw chat only. It should create a structured HITL request object, for example:

- `request_id`
- `type`: `approval`, `choice`, `clarification`
- `origin_agent_id`
- `origin_task_id`
- `question`
- `options`
- `status`: `pending`, `answered`, `timed_out`, `cancelled`
- `expires_at`

Then each channel adapter decides how to render that request.

## Why this is better

This gives you:

- one orchestration model
- multiple presentation modes
- proper timeout/retry/escalation behavior
- less ambiguity than plain text alone

It also lets the runtime treat user decisions as part of control flow, not just as unstructured chat.

## Suggested rendering policy

- **Terminal**: rich interactive prompts, step-by-step questions
- **Telegram**: inline buttons when possible, text fallback always
- **Discord/Slack**: native components where supported
- **Unsupported channels**: plain text only

## Recommendation for OpenClaw

If OpenClaw adds stronger HITL support, the right approach is:

- backend-level structured HITL requests
- channel-specific rendering adapters
- universal text fallback

That keeps the gateway architecture clean and avoids baking channel behavior into the core agent logic.

## Bottom line

Human-in-the-loop should be modeled as a **structured runtime object**, not just as a string in chat.

The UI can vary by channel, but the control flow should stay the same.
