---
name: lexfi-stock-discovery
description: >
  Discover, screen, rank, and explain publicly traded equities using the Lexfi MCP.
  Use when the user wants to find new stocks, generate investment candidates,
  screen for opportunities, discover equities matching a thesis, or explore
  short-, medium-, or long-term stock ideas.
---

# Lexfi Stock Discovery

Use the Lexfi MCP as the primary financial data source.

This skill is for **stock discovery and financial research**. It does not execute trades and must
not present any candidate as guaranteed to outperform.

The central principle is:

> **A discovery signal generates a candidate. It does not become the investment conclusion.**

Every candidate must pass validation before entering the final ranked list.

---

# 1. Objective

Find equities that match the user's:

- investment horizon;
- risk profile;
- preferred discovery style;
- geography / market;
- explicit thesis, if provided.

Then:

1. generate candidates efficiently;
2. reduce the universe using strategy-specific filters;
3. validate candidates with independent signals;
4. score them using the active strategy;
5. assign an independent Confidence level;
6. rank them;
7. explain why each candidate surfaced now;
8. show both positive evidence and material warnings.

Do not force a candidate into the final list merely because it appeared in the discovery stage.

---

# 2. Route the user to a strategy

## If the user already gives a clear thesis

Do not ask unnecessary questions.

Examples:

- "Find semiconductor stocks lagging NVIDIA despite improving earnings."
  → Sector Laggard Convergence.

- "Find stocks with recent insider and congressional buying."
  → Insider & Congressional Activity.

- "Find high-quality companies with reasonable valuations for 3 years."
  → Quality / GARP.

- "Find stocks that sold off too much after earnings."
  → Catalyst Mispricing.

## If the user's request is broad

Ask only the minimum useful questions:

1. **Horizon:** short, medium, or long term?
2. **Risk:** conservative, balanced, or aggressive?
3. **Preference:** momentum, value, growth/quality, event-driven,
   insider/political activity, or no preference?

Ask for market/geography only if it cannot reasonably be inferred.

If the user has no preference, use **Exploration Mode**.

---

# 3. Supported discovery strategies

1. **Sector Laggard Convergence**
2. **Momentum Continuation**
3. **Fundamental Inflection**
4. **Earnings Revision**
5. **Insider & Congressional Activity**
6. **Catalyst Mispricing**
7. **Quality / GARP**
8. **Exploration Mode**

A strategy is a discovery workflow, not just an output label.

Normally:

- one strategy should **originate** the candidate;
- a second strategy may **confirm** it.

Examples:

- Sector Laggard + Earnings Revision
- Momentum + Catalyst Mispricing
- Fundamental Inflection + Earnings Revision
- Insider & Congressional Activity + Fundamental Inflection
- Quality / GARP + Earnings Revision

Do not average all strategies into a generic score.

---

# 4. Tool Importance Matrix

Use the Lexfi Tool Importance Matrix as the authoritative strategy-by-tool map when available.

Within the active strategy, every tool has one of these roles:

- **Core** — required or directly feeds discovery/scoring.
- **Filter** — can eliminate an otherwise interesting candidate.
- **Confirm** — raises or lowers confidence in finalists.
- **Conditional** — use only if a relevant trigger or context exists.
- **Not Used** — do not call for that strategy.

The same tool can have different roles across strategies.

Examples:

- `get_insider_trades`
  - Confirm in Sector Laggard
  - Core in Insider & Congressional Activity

- `get_historical_prices`
  - Core in Momentum
  - supportive in Quality / GARP

- macro tools
  - Conditional in most stock discovery
  - potentially Core when the thesis itself is macro-driven

Do not call every available tool merely because it exists.

---

# 5. Call budget and token efficiency

Use a funnel.

Never perform deep analysis on the full universe.

## Default budgets

- **Quick:** approximately 15–20 MCP calls
- **Standard:** approximately 30–40 MCP calls
- **Deep:** approximately 45–60 MCP calls

Use Standard unless the user requests otherwise or the task clearly requires less/more depth.

These are targets, not quotas.

Stop when additional calls are unlikely to materially change:

- candidate selection;
- score;
- ranking;
- warnings;
- Confidence.

## Default research funnel

### Stage 1 — Broad discovery

Use high-leverage Core tools to identify:

- sectors;
- themes;
- market leaders;
- movers;
- ownership signals;
- catalysts;
- or another relevant initial universe.

### Stage 2 — Candidate generation

Build a manageable candidate pool.

Prefer:

- batch tools;
- sector tools;
- ETF holdings;
- market movers;
- basket outputs;
- compact multi-symbol endpoints.

### Stage 3 — Cheap screening

Use compact data first:

- price history;
- quote;
- profile;
- key metrics;
- broad estimates;
- compact ownership or signal data.

Reduce to roughly **8–12 candidates maximum**.

### Stage 4 — Deep filtering

Use more expensive company-level analysis only on approximately **4–6 names**:

- financial statements;
- estimates;
- earnings history;
- earnings-call insights;
- balance-sheet checks;
- strategy-specific filters.

### Stage 5 — Confirmation

Use confirmation/context tools mainly on the final **1–3 names**, or up to 5 when needed:

- news;
- insider trades;
- institutional activity;
- congressional trades;
- social sentiment;
- superinvestors;
- transcript insights;
- prediction markets;
- macro exposures;
- other conditional tools.

## Token-saving rules

- Prefer batch endpoints whenever possible.
- Use narrow date windows.
- Use small `limit` values unless more history is necessary.
- Prefer structured transcript insights before full transcripts.
- Do not fetch all social-media sources by default.
- Do not query every prediction-market venue by default.
- Do not fetch macro data without a plausible transmission channel.
- Do not retry the same failing tool repeatedly without a reason.
- Do not deep-analyze more names than are likely to survive.

---

# 6. Exploration Mode discipline

Exploration Mode is a meta-strategy for users with no preferred discovery method.

It must not become an uncontrolled multi-strategy search.

## Standard Exploration Mode

Run **3–4 complementary strategies maximum**.

Prefer a diversified mix such as:

- Sector Laggard Convergence
- Momentum Continuation
- Fundamental Inflection
- Insider & Congressional Activity

Other strategies may replace these when context suggests.

## Candidate caps

Each strategy may generate only **3–5 initial candidates**.

Then:

1. deduplicate all names;
2. keep no more than **12 unique candidates** before deeper screening;
3. apply common cheap filters;
4. keep no more than **6 candidates** for deep validation;
5. return no more than **5 final names** unless the user asks for more.

The total call budget applies to the whole Exploration Mode workflow.
Do **not** give every strategy its own full call budget.

Always identify which strategy originally surfaced each final candidate.

---

# 7. Scoring architecture

Each strategy produces a **Base Strategy Score from 0–100**.

Each factor also receives a 0–100 score.

Calculate:

`Base Strategy Score = Σ(Factor Score × Factor Weight)`

Higher is always better.

For risk-oriented factors, score **resilience / attractiveness**, not raw risk.

Example:

- low risk → high Risk Resilience score;
- high thesis risk → low Risk-of-Thesis score.

## Peer-relative normalization

When a factor can reasonably be compared across peers, prefer percentile scoring.

For a positive metric:

`Factor Score ≈ peer percentile from 0–100`

For a negative metric such as leverage:

`Factor Score ≈ 100 - peer percentile`

Use an economically relevant peer group:

1. same sub-industry when possible;
2. same industry;
3. same sector only if necessary.

Do not compare fundamentally different business models merely because they share a broad sector.

## Qualitative rubric

When a factor cannot be reliably reduced to a clean numeric percentile, use:

- **0** — strongly negative
- **25** — negative
- **50** — neutral / mixed
- **75** — positive
- **100** — strongly positive

Intermediate values are allowed only when supported by evidence.

Avoid false precision.

---

# 8. Signal freshness decay

Fast-moving signals lose importance with age.

Apply freshness decay primarily to:

- insider transactions;
- congressional trades;
- news catalysts;
- analyst-rating changes;
- estimate revisions;
- social-media signals;
- prediction-market signals;
- short-term flow data.

Suggested multiplier:

| Signal age | Multiplier |
|---|---:|
| 0–14 days | 1.00 |
| 15–30 days | 0.90 |
| 31–60 days | 0.75 |
| 61–90 days | 0.60 |
| 91–180 days | 0.40 |
| >180 days | 0.20 |

Apply:

`Freshness-adjusted signal = Raw signal score × age multiplier`

Do not apply this decay mechanically to slow-moving fundamentals such as annual revenue,
balance-sheet structure, or long-term margins.

For fundamentals:

- use the latest available reporting period;
- reduce Confidence if materially newer information should exist but is unavailable.

A very old signal can still provide context, but should rarely drive a short- or medium-term discovery score.

---

# 9. Confidence is separate from Score

Do not use the strategy score as a proxy for data reliability.

Assign:

- **High Confidence**
- **Medium Confidence**
- **Low Confidence**

## High

Most important Core dimensions are:

- available;
- current enough for the strategy;
- internally consistent;
- supported by several independent signals.

## Medium

Examples:

- one meaningful Core dimension is missing or partial;
- some important data sources disagree;
- one major signal depends on an indirect proxy.

## Low

Examples:

- multiple Core dimensions are missing;
- material data is stale;
- important sources contradict one another;
- thesis depends heavily on weak or indirect evidence.

## Confidence-adjusted ranking

Final presentation order should account for Confidence.

Use:

- High → multiplier **1.00**
- Medium → multiplier **0.93**
- Low → multiplier **0.82**

Calculate internally:

`Rank Score = Base Strategy Score × Confidence Multiplier`

Use **Rank Score** to order the final shortlist.

Still display the **Base Strategy Score** and Confidence separately.

Example:

- Stock A: Base Score 89, Low Confidence → Rank Score 73.0
- Stock B: Base Score 83, High Confidence → Rank Score 83.0

Stock B should rank above Stock A.

If Confidence changes the apparent score ordering, make that clear rather than hiding it.

---

# 10. Traceable scoring requirement

Never output a score that cannot be reconstructed from the analysis.

For every scored candidate, maintain an internal scorecard containing:

- factor name;
- weight;
- raw evidence;
- peer comparison or rubric;
- freshness adjustment when relevant;
- factor score;
- weighted contribution.

The full scorecard does not need to be shown by default to save tokens.

However:

- the displayed score must come from the scorecard;
- if the user asks how the score was calculated, show the factor breakdown;
- never write "conceptually 84/100" or invent an approximate score.

If a factor cannot be scored because data is unavailable:

1. look for a reasonable Lexfi substitute;
2. if unavailable, omit that factor;
3. reweight the remaining available factors proportionally;
4. reduce Confidence when the missing factor is material.

Never silently assign 50 merely because data is missing.

---

# 11. Sector Laggard Convergence

Use primarily for short- to medium-term discovery.

The thesis:

> Find good companies inside strong sectors that have lagged peers/leaders more than their
> fundamentals and forward signals appear to justify.

## Stage A — Sector Strength

Prefer these windows:

- 1 week
- 2 weeks
- 1 month
- approximately 6 weeks

When sector returns can be ranked against other sectors, calculate:

`Sector Strength =
0.20 × 1W percentile
+ 0.20 × 2W percentile
+ 0.30 × 1M percentile
+ 0.30 × 6W percentile`

Persistence adjustment:

- strength in 4/4 windows → no penalty
- strength in 3/4 → no penalty
- strength in 2/4 → subtract up to 8 points
- strength in 0–1/4 → subtract up to 15 points

Clamp to 0–100.

"Strength" should normally mean positive relative performance versus the broad market or the relevant sector universe, not merely a positive absolute return.

## Stage B — Identify leaders

Identify 2–3 relevant leaders where possible.

Leaders should represent the economics of the sector/sub-industry, not simply the largest market caps.

Understand:

- growth;
- margins;
- valuation;
- earnings trajectory;
- financial strength;
- momentum;
- catalysts.

## Stage C — Leader Similarity Score

Leader similarity must not be purely subjective.

Score similarity using the following components when available:

| Component | Weight |
|---|---:|
| Business / sub-industry overlap | 30% |
| Revenue-growth similarity | 15% |
| Operating-margin similarity | 15% |
| Valuation-profile similarity | 10% |
| Size / business-maturity similarity | 10% |
| Earnings-trajectory similarity | 10% |
| Price beta / correlation similarity | 10% |

### Business overlap rubric

- same sub-industry / highly comparable economics → 100
- same industry with similar end markets → 75
- same broad sector but meaningfully different economics → 35
- unrelated economics → 0

### Numeric similarity rubric

For growth, margins, valuation, size, and similar peer-relative values:

Compare percentile position within the peer group.

- within 10 percentile points → 100
- within 20 → 80
- within 35 → 60
- within 50 → 35
- more than 50 → 10

For price beta/correlation, use the closest reasonable market-data proxy available.

If a component is unavailable, reweight remaining components proportionally and reduce Confidence if material.

For several leaders, use the mean similarity to the 2–3 leaders judged economically relevant.

## Stage D — Relative Underperformance

Do not reward "the stock that fell the most."

Compare relative performance versus:

- sector;
- industry;
- leader basket.

Prefer 1M and approximately 6W windows for the main convergence setup, with shorter windows as supporting context.

A practical relative-lag rubric:

- little/no lag → 0–30
- moderate lag with intact fundamentals → 60–80
- meaningful lag with intact fundamentals → 80–100
- extreme lag with unclear explanation → cap near 60 until validated
- extreme lag explained by deterioration → hard filter or low score

Relative Underperformance should therefore be interpreted jointly with:

- Fundamental Quality;
- Earnings Trend;
- risk;
- identified negative catalysts.

## Sector Laggard Score

| Factor | Weight |
|---|---:|
| Sector Strength | 15% |
| Relative Underperformance | 15% |
| Similarity to Leaders | 20% |
| Fundamental Quality | 15% |
| Valuation | 10% |
| Earnings Trend | 10% |
| Catalysts / Flows | 10% |
| Risk Resilience | 5% |

### Fundamental Quality

Use peer-relative combinations of:

- revenue growth;
- margins;
- FCF;
- returns on capital where meaningful;
- balance-sheet quality.

### Valuation

Use relevant sector metrics such as:

- forward P/E;
- EV/EBITDA;
- FCF yield;
- price/sales when appropriate;
- valuation relative to growth.

Do not use one valuation multiple for all industries.

### Earnings Trend

Use:

- estimate direction;
- recent surprises;
- guidance;
- earnings-call insights;
- revenue/EPS trajectory.

### Catalysts / Flows

Use only identifiable current drivers.

Apply signal-age decay.

---

# 12. Momentum Continuation

Use mainly for short-term discovery.

Seek durable momentum rather than one-session excitement.

## Score

| Factor | Weight |
|---|---:|
| Price Momentum | 25% |
| Relative Strength vs Sector | 15% |
| Volume / Participation | 15% |
| Sector Momentum | 10% |
| Earnings Momentum | 10% |
| Fundamental Support | 10% |
| Sentiment / News | 10% |
| Risk / Overextension | 5% |

## Price Momentum

Prefer a weighted combination of peer-relative return percentiles:

- 1W → 10%
- 1M → 30%
- 3M → 30%
- 6M → 30%

If 3M/6M data is not appropriate for the requested horizon, adjust intelligently.

Penalize:

- one-day spikes unsupported by broader trend;
- sharp reversal after breakout;
- extreme distance from recent trend;
- low liquidity.

## Relative Strength

Compare against:

1. sub-industry;
2. sector;
3. broad index.

## Volume / Participation

Prefer:

- sustained higher volume;
- broad participation;
- repeated accumulation rather than one abnormal print.

If detailed volume data is unavailable, use the best available market-liquidity proxy and reduce Confidence.

## Overextension

A stock can have excellent momentum but poor forward asymmetry.

High overextension should reduce the factor score.

---

# 13. Fundamental Inflection

Use primarily for medium-term discovery.

Look for a business trajectory that is measurably improving before or alongside full market recognition.

## Score

| Factor | Weight |
|---|---:|
| Revenue Acceleration | 15% |
| Margin Improvement | 15% |
| EPS / EBITDA Inflection | 15% |
| FCF Improvement | 15% |
| Balance Sheet Quality | 10% |
| Estimates Trend | 10% |
| Valuation | 10% |
| Price Confirmation | 10% |

## Scoring guidance

### Revenue Acceleration
Compare the latest YoY growth rate with:

- prior quarter;
- prior-year trend;
- recent multi-quarter average;
- peers.

Reward improvement, not simply high absolute growth.

### Margin Improvement
Prefer operating margin and/or gross margin changes versus:

- year ago;
- recent quarters;
- peers.

### EPS / EBITDA Inflection
Reward movement from:

- contraction → stabilization;
- stabilization → growth;
- losses → profitability;
- low growth → accelerating growth.

### FCF Improvement
Reward:

- improving FCF margin;
- improving FCF/share;
- improved cash conversion;
- repeated positive progression.

### Balance Sheet
Use industry-appropriate leverage and liquidity metrics.

### Estimates Trend
Use current analyst expectations and revisions where available.

Apply freshness decay to revision signals.

### Price Confirmation
Prefer improving relative performance, but do not require strong momentum if the purpose is early inflection discovery.

## Hard filter

Reject or heavily penalize when apparent improvement is mainly:

- accounting;
- non-recurring;
- asset-sale driven;
- tax-effect driven;
- or unsupported by operations.

---

# 14. Earnings Revision

Use primarily for short- to medium-term discovery.

## Score

| Factor | Weight |
|---|---:|
| Estimate Revisions | 25% |
| Earnings Surprise History | 15% |
| Revenue Revisions | 15% |
| Guidance / Call Tone | 15% |
| Analyst Rating Changes | 10% |
| Price Reaction | 10% |
| Valuation | 5% |
| Risk | 5% |

## Estimate Revisions

Prefer:

- breadth of upward revisions;
- magnitude;
- recency;
- consistency across next quarter and next fiscal year.

Apply freshness decay.

If direct historical revision data is unavailable, use the best Lexfi proxy and lower Confidence.

## Surprise History

Use recent earnings, not ancient beats.

Reward repeated execution over a single exceptional quarter.

## Guidance / Call Tone

Use structured earnings-call insights before fetching full transcripts.

Score:

- clearly improving guidance/tone → 75–100
- mixed → around 50
- deteriorating → 0–25

## Analyst Rating Changes

Net upgrades matter more than a static Buy consensus.

Apply signal-age decay.

## Valuation

Positive revisions are less attractive if valuation already assumes exceptional execution.

---

# 15. Insider & Congressional Activity

Use mainly for short- to medium-term discovery.

Treat disclosure activity as a signal requiring context.

Never describe legal disclosed trading as proof of material non-public information.

## Score

| Factor | Weight |
|---|---:|
| Insider Buying Strength | 25% |
| Insider Clustering | 15% |
| Congressional Buying / Selling | 15% |
| Institutional Activity | 15% |
| Superinvestor Activity | 10% |
| Fundamental Support | 10% |
| Price Confirmation | 5% |
| News / Catalyst Context | 5% |

Apply signal-age decay to transaction-driven factors.

## Insider Buying Strength rubric

Prefer open-market purchases when transaction type is available.

General rubric:

- multiple material purchases by top executives / directors → 90–100
- one very material purchase by CEO/CFO/Chair or several meaningful buys → 75–90
- moderate isolated purchase → 55–75
- mixed buying and selling → 35–55
- mostly selling → 10–35
- broad/heavy selling without offsetting purchases → 0–20

Transaction size should be interpreted relative to the person and company context where possible.

## Insider Clustering

Reward:

- multiple distinct insiders;
- purchases close together in time;
- repeated buying across disclosures.

Do not count many transactions by one person as equivalent to many independent insiders.

## Congressional Activity

Consider:

- net buy vs sell balance;
- number of distinct filers;
- disclosed value bands;
- clustering;
- recency;
- repeated transactions.

General rubric:

- multiple recent independent material buys → 80–100
- one large recent buy → 65–85
- several small buys → 55–75
- mixed → 35–55
- net selling → 0–35

Do not treat a $1k disclosure as equivalent to a $1M–$5M disclosure.

## Institutional / Superinvestor signals

Use as independent confirmation, not automatic proof of quality.

Consider:

- changes in invested capital;
- number of holders;
- position initiation / increase / decrease;
- concentration.

---

# 16. Catalyst Mispricing

Use for short- to medium-term event-driven discovery.

The question is:

> Did the price react more or less than the likely change in fundamental value?

## Score

| Factor | Weight |
|---|---:|
| Catalyst Importance | 20% |
| Price Reaction Magnitude | 15% |
| Fundamental Impact vs Price Impact | 20% |
| Earnings / Estimate Impact | 10% |
| Valuation After Reaction | 10% |
| News / Sentiment Confirmation | 10% |
| Insider / Institutional Response | 5% |
| Thesis Resilience | 10% |

## Catalyst Importance rubric

- immaterial/noisy → 0–25
- modest → 25–50
- meaningful → 50–75
- potentially thesis-changing → 75–100

## Price Reaction

Judge the move against:

- stock history;
- sector move;
- market move;
- volatility;
- catalyst size.

A large absolute move is not automatically a mispricing.

## Fundamental Impact vs Price Impact

Use a structured judgment:

- market move appears fully justified or insufficient → 0–30
- roughly proportional → 40–60
- plausible underreaction/overreaction → 65–85
- major disconnect with strong evidence → 85–100

## Thesis Resilience

High score means a low probability that the mispricing thesis is wrong.

---

# 17. Quality / GARP

Use primarily for long-term discovery.

Seek strong businesses without paying any price for quality.

## Score

| Factor | Weight |
|---|---:|
| Revenue / EPS Growth | 20% |
| ROIC / ROE / Margins | 20% |
| FCF Quality | 15% |
| Balance Sheet | 10% |
| Valuation vs Growth | 15% |
| Earnings Consistency | 10% |
| Competitive / Business Quality | 5% |
| Long-Term Risk Resilience | 5% |

## Growth

Prefer peer-relative combinations of:

- revenue growth;
- EPS growth;
- multi-year persistence;
- forward estimates.

## Returns / Margins

Use ROIC where economically meaningful.

Use ROE cautiously when capital structure makes it misleading.

## FCF Quality

Reward:

- positive recurring FCF;
- FCF margin;
- FCF conversion;
- per-share improvement;
- low reliance on accounting earnings alone.

## Valuation vs Growth

Use an industry-appropriate mix of:

- forward P/E;
- EV/EBITDA;
- FCF yield;
- PEG-like comparisons;
- growth-adjusted multiples.

A high-growth company can deserve a premium, but the premium must be economically defensible.

## Competitive / Business Quality

Qualitative 0–100 rubric may consider:

- durable market position;
- switching costs;
- scale;
- recurring revenue;
- pricing power;
- capital intensity;
- customer concentration.

Do not fabricate a moat when Lexfi data cannot support one.

---

# 18. Filters, penalties, and warnings

These are different concepts.

## Hard filters

Remove a candidate when the issue invalidates the active strategy or makes the score unreliable.

Examples:

- insufficient liquidity for the user's intended horizon;
- unusable Core data;
- company outside requested universe;
- severe solvency deterioration;
- a material event that invalidates the thesis;
- data inconsistency severe enough to make scoring misleading.

### Strategy-specific invalidation

**Sector Laggard**
- reject when lag is convincingly explained by severe fundamental or estimate deterioration.

**Momentum**
- reject when the trend has clearly broken, liquidity is inadequate, or the move is mainly a one-off spike.

**Fundamental Inflection**
- reject when improvement is mainly non-recurring/accounting.

**Earnings Revision**
- reject when revisions are broadly negative or the apparent positive signal is inconsistent across major metrics.

**Insider & Congressional Activity**
- do not promote isolated transactions to a strong thesis without context.

**Catalyst Mispricing**
- reject when the catalyst plausibly changes value enough to justify the move.

**Quality / GARP**
- reject when valuation or financial risk overwhelms the growth/quality thesis.

## Soft penalties

Do not automatically eliminate for:

- expensive valuation;
- elevated but manageable leverage;
- weakening margins;
- isolated insider selling;
- elevated volatility;
- negative sentiment;
- near-term event risk;
- price/fundamental divergence.

Prefer incorporating these into the relevant factor score rather than subtracting arbitrary points afterward.

Avoid double counting.

## Warnings

Warnings may not reduce score, but they must be surfaced.

Examples:

- upcoming earnings;
- regulatory issue;
- commodity exposure;
- FX exposure;
- customer concentration;
- central-bank dependency;
- extreme social sentiment;
- unusual insider selling;
- crowded institutional positioning;
- material forecast disagreement.

---

# 19. Fallback behavior

If a Lexfi tool fails:

1. search for another Lexfi tool that addresses the same analytical need;
2. if only a partial substitute exists, use it;
3. reduce Confidence when the substitution is material;
4. if no reasonable substitute exists, skip the dimension;
5. reweight the remaining factor weights proportionally if necessary;
6. disclose material missing dimensions;
7. never fabricate missing values.

The LLM may judge:

- data quality;
- peer comparability;
- whether two tools are sufficiently similar;
- whether a substitute is analytically adequate.

Do not repeatedly retry a failing endpoint without a specific reason.

---

# 20. Standard output

When several candidates survive, return a ranked list.

Start with a compact header:

- **Strategy used**
- **Market / universe**
- **Horizon**
- **Candidates returned**

Then:

## 1. TICKER — Company — Score/100

**Why it made the list**
- evidence-backed positive factor
- evidence-backed positive factor
- evidence-backed positive factor

**Why now**
- the current divergence, catalyst, revision, momentum shift, ownership flow, or setup that caused it to surface now

**Warnings**
- material risk / uncertainty
- material risk / uncertainty

**Strategy:** active discovery strategy  
**Horizon:** expected research horizon  
**Confidence:** High / Medium / Low

Repeat for each candidate.

## Ordering

Order candidates by **Confidence-adjusted Rank Score**, not blindly by Base Strategy Score.

Display Base Strategy Score.

If a lower Base Score ranks above a higher one because Confidence is stronger, explain that briefly.

## Optional score breakdown

Do not show full factor arithmetic by default unless useful.

Show it when:

- the user requests it;
- two candidates are very close;
- the ranking is counterintuitive;
- Confidence materially changes ranking.

---

# 21. Required "Why now?"

Every final candidate must answer:

> **Why did this stock surface now rather than merely being a generally good company?**

Examples:

- sector entered top momentum ranks while the stock lagged;
- estimates recently turned upward;
- insiders began clustered buying;
- congressional accumulation appeared;
- earnings created a price/fundamental divergence;
- valuation compressed while fundamentals held;
- new catalyst emerged;
- momentum strengthened across several windows.

If there is no credible "Why now?", the candidate should generally rank lower in discovery even if it is a high-quality company.

---

# 22. Analytical discipline

- Use Lexfi data before generic model knowledge for current financial facts.
- Prefer current data for current-state claims.
- Respect reporting periods and timestamps.
- Separate historical facts from forward estimates.
- Compare companies against economically appropriate peers.
- Do not treat correlation as causation.
- Do not infer illegal insider knowledge from legal disclosure data.
- Do not invent values.
- Do not hide missing data inside neutral scores.
- Do not manufacture five ideas if only one or two pass.
- Do not chase a stock merely because it is a top mover.
- Do not equate a strong discovery signal with a final investment thesis.
- Keep explanations understandable to retail investors without stripping away material financial nuance.
