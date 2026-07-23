# Today's Digest

- Status: enabled
- Timing mode: `exact_schedule`
- Timezone: `America/New_York`

## Schedule

```ical
BEGIN:VEVENT
DTSTART:20260709T110000
RRULE:FREQ=DAILY;BYHOUR=11;BYMINUTE=0;BYSECOND=0
END:VEVENT
```

## Prompt

```text
Create Alex's daily priority digest in Russian. Keep it evidence-first, action-first, and under 700 words.

Start from existing evidence instead of researching from scratch:

1. With the GitHub App, search recent commits and fetch the confirmed snapshots from the last 12 hours in qobeat/social-trends:
   - data(public:google_trends_us)
   - data(public:reddit_popular_us)
   - data(public:hacker_news_top)
   - data(github)
2. Use the previous digest in this task's conversation when available. Follow up only on its unresolved or materially changed items.
3. Use live Web Search only to verify or update the small set of candidate items found above. Do not run a broad generic news sweep.

Prioritize: deterministic/agentic AI and coding tools; local LLM/RTX 3090 workflows; solo-founder and vertical-SaaS opportunities; US non-dilutive funding; concrete kids STEM/research opportunities; Jersey City/NJ/NYC risks and opportunities; frontier science with product, grant, or execution relevance.

Context efficiency:
- When a Google snapshot contains source-native `canonical_topic`, `domain`,
  `subdomain`, `domain_confidence`, and `context_terms`, use them as the
  first-pass interpretation of the trend.
- Do not spend Web Search reclassifying entertainment, sports, celebrity, or
  other low-relevance trends unless the item independently matters to Alex.
- Treat domain classification as context only, never as evidence of materiality
  or as a second independent source.
Selection rules:
- maximum 5 items;
- include only confirmed changes, deadlines, decisions, risks, opportunities, or follow-ups that matter today;
- do not repeat an unchanged item from the previous digest or source alerts;
- do not treat raw attention, rank, or total stars alone as acceleration;
- omit celebrity/sports/political noise unless it creates a direct practical consequence for Alex;
- verify dates, amounts, eligibility, and other numbers from primary sources;
- clearly separate fact from inference;
- if evidence is insufficient, say "Не могу подтвердить" and do not recommend action based on it.

Output:
- New York timestamp and one-sentence executive conclusion;
- up to 5 ranked items, each with Fact, What changed, Why it matters, Action today, Confidence, and direct source URL;
- 3–5 concrete actions for today;
- short Blocked / Cannot confirm section only when it affects a decision.

Do not include weather unless there is an actionable local hazard. Do not add generic background, scoring tables, unchanged watchlists, sentiment analysis, sponsorship, or filler. If no material item exists, send only a short "существенных изменений нет" statement plus at most two useful follow-up actions. Do not create or modify Scheduled Tasks from inside the run.
```
