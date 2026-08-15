# hallucinations.cloud rewrite requirements (confirmed 2026-08-15)

## Scope and sequencing

Two step project. Step one is a complete remake of hallucinations.cloud. Step two is noaibs.org. Step one must be completed in full before step two begins.

## Look and feel

Sparse, minimal aesthetic, matching the-campus.luxuryproperty-southdakota.com.

## Site structure

1. Landing page: welcome message and description of the tool.
2. Query page: a space for the user to submit a query.
3. After the user clicks submit, the user must authenticate by cell phone.
4. That authentication step triggers an email notification to Brian.

## Business model

The system is free for users up to a point. Brian wants to be paid an amount equal to the actual LLM API costs he is charged for that user's usage. Once a user's accrued cost reaches $20, the user is required to use the payment system to offset those costs before continuing.

## Core detection engine (the rewrite)

Rebuilt with the best tools available now, replacing the prior approach. Must include:

- 8 responses (multiple model responses gathered per query, as in the prior version)
- Red team / blue team adversarial checking of those responses
- Search of current content beyond the LLM's own knowledge cutoff, to check claims against up to date sources

All three of the above should be supplemented and upgraded using the best current tools and services available, not just reimplemented as before.

## Process rule for this session

No em dashes anywhere, under any circumstance. Do not interrupt Brian and do not run anything until he says so.
