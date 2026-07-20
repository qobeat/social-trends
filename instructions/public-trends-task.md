# Public Cross-Trend Signals

## Task instruction

Run one lightweight cross-trend correlation pass. This is a summarizer, not a
collector, news digest, sentiment task, or replacement for the source tasks.

### Allowed tool and inputs

Use only the connected GitHub App actions `github_search_commits` and
`github_fetch_file` on `qobeat/social-trends@main`. Do not use Web Search,
Browser, shell, Python, or external enrichment.

Read only:

- the latest two confirmed snapshots committed as
  `data(public:google_trends_us)`;
- the latest two confirmed snapshots committed as
  `data(public:reddit_popular_us)`;
- the latest two confirmed snapshots committed as
  `data(public:hacker_news_top)`;
- the latest two confirmed `data(github)` snapshots.

Ignore legacy all-source public snapshots except when needed once to establish a
migration baseline. Do not create or modify repository files.

### Goal

Identify a genuinely unusual topic that is independently present or accelerating
across at least two current source groups. Normalize only obvious case,
punctuation, spacing, and exact aliases. Do not merge merely related topics by
speculation.

GitHub counts as an independent source group only when verified repository
metadata explicitly connects a repository to the public topic. A repeated topic
inside one source or two endpoints of the same source is one signal, not two.

### Material gate

Notify Alex in Russian only when at least one is confirmed:

- the same normalized topic newly appears in at least two current source groups;
- the topic is present in two groups and directly comparable native evidence
  shows material acceleration in at least one;
- a GitHub repository with verified relevance aligns with a current public
  trend and the combination creates a concrete ADOS, agentic-coding, local-LLM,
  education/science, HOA/property, or solo-founder opportunity or risk;
- one of the expected source collectors has no confirmed snapshot for more than
  eight hours.

Do not repeat an alert unless a source, native metric, confidence, or practical
action materially changes. Source collectors own their individual health and
storage alerts; do not duplicate those alerts here.

If the gate is not met, emit no user-facing notification.

### Output

Keep a material alert under 250 Russian words:

- New York timestamp;
- normalized topic and contributing source groups;
- confirmed facts with direct GitHub snapshot URLs;
- inference clearly separated from facts;
- why it matters to Alex and one concrete action;
- limitations and confidence.

Do not include a watchlist, generic context, unchanged items, sentiment, or
background research.