# hallucinations.cloud rewrite: architecture and tech stack (draft for review, 2026-08-15)

This is a proposal only. Nothing has been built yet. Review and confirm or redirect before any code is written.

## 1. Product flow

1. Landing page: sparse, minimal, same visual language as the-campus.luxuryproperty-southdakota.com. Welcome message and short description of the tool.
2. Query page: single input for the user's question or claim to check.
3. User clicks submit.
4. Cell phone authentication gate (SMS code) before the query runs.
5. Successful authentication triggers an email notification to Brian.
6. Query runs through the detection engine (section 3).
7. Result shown to user.
8. Running cost meter per user, tied to actual LLM/API spend. Free until it reaches a $20 accrued cost, then the user must pay through the built in payment system before further queries run.

## 2. Tech stack

- Backend: FastAPI (Python), already scaffolded.
- Frontend: server rendered Jinja2 templates plus minimal CSS, no heavy JS framework, matching the sparse look of the-campus site.
- Database: Supabase (Postgres), one instance for operational data (users, sessions, cost ledger), a separate instance for training data capture, as already scaffolded.
- Phone auth: Twilio Verify for SMS one time codes. Already have a working Twilio integration pattern from the original hallucinations.cloud sign in feature.
- Email notification to Brian: Gmail App Password send, same pattern as the original site's sign in notification feature.
- Payment: Stripe, scoped narrowly to a metered/pre-auth charge once a user crosses the $20 cost threshold.
- Hosting: Render, matching the rest of Brian's stack.
- Testing: Playwright for end to end, pytest for unit and integration, matching Brian's standing project default.

## 3. Detection engine

### 3.1 Multi model fan out

Recommendation: use OpenRouter as the transport layer for calling multiple models (Claude, GPT, Gemini, and others) instead of hand building separate SDK integrations for each provider. Keep direct Anthropic and OpenAI API keys configured as a fallback path only, in case OpenRouter has an outage.

Instead of a flat 8 independent full calls every time, use a tiered panel:
- First pass: 3 to 4 diverse models answer the query.
- If those responses agree closely, stop there and return a result.
- If they disagree meaningfully, escalate to the full wider panel (up to 8) for a fuller cross check.

This keeps quality close to the original 8 call approach but meaningfully lowers typical per query cost, which matters directly because that cost is what gets passed through to the user against the $20 cap.

### 3.2 Red team / blue team cross checking

Recommendation: implement the adjudicator pattern directly rather than adopting a full third party observability platform. One model or pass proposes claims, other models in the panel adjudicate and vote, contradictions and unsupported claims get flagged.

Optionally include Patronus AI's Lynx hallucination detection model as one additional adjudicator vote in the panel, since it is a maintained product built specifically for this task.

Considered and set aside for now: Promptfoo is well suited for building and testing the red team adversarial prompts themselves during development, not for runtime scoring, so it may be useful later as a testing tool rather than a production component. Braintrust and Galileo are full observability platforms with real per query audit trail value but cost more than this stage needs; revisit only if compliance or audit requirements grow.

### 3.3 Live web grounding

Recommendation: Exa as the primary live search/grounding provider, on cost to quality grounds. Perplexity Sonar as a secondary tie breaker call used only when Exa's returned sources are ambiguous, to control cost, since Sonar is priced meaningfully higher.

Tavily was considered and set aside, no clear edge over Exa for this specific fact verification use case.

### 3.4 Primary answering/synthesis model

Claude Haiku 4.5, as already decided, used for the final synthesis step that turns the panel's findings into the answer shown to the user.

## 4. Cost control

Every LLM call, search call, and adjudication call in a query's pipeline gets logged against that user's running cost ledger in Supabase. The tiered panel approach in 3.1 is the main lever for keeping typical per query cost down, so the $20 free allowance lasts longer per user before the payment gate triggers.

## 5. Open questions for Brian

- Confirm OpenRouter as the fan out transport layer, versus continuing with the already scaffolded direct multi provider key setup (OpenAI, Anthropic, Google, Cohere, DeepSeek, OpenRouter, Perplexity, Grok, Tavily).
- Confirm Exa as primary search/grounding provider, replacing Tavily, which was the prior draft's placeholder.
- Confirm the tiered panel approach (3 to 4 models first, escalate to 8 only on disagreement) versus always running the full 8 call panel regardless of cost.
- Confirm Twilio Verify and Gmail App Password reuse from the original site for phone auth and the notification email, versus something new.
