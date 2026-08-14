# AI Shelf Audit — Methodology

**Version:** 1.0 (draft — pending operator review, not yet published)
**Date:** 13 August 2026
**First application:** Perfume & fragrance, United Arab Emirates

This document describes the full measurement protocol behind AI Shelf Audit reports. It is written to be published: any reader should be able to evaluate the method, and any skeptic with their own API keys should be able to re-run it. Where the method involves a judgment call, we state the call and the reason for it.

---

## 1. What this measures

When shoppers ask AI assistants what to buy — "best long-lasting perfume for Dubai summer", "where should I buy fragrance online in the UAE" — the assistants answer with specific brands and stores. An AI Shelf Audit measures, for one product category in one country, **which brands and retailers AI assistants name, how consistently, and how prominently**.

The headline metric is **Shelf Share**: a brand's position-weighted share of shopper questions on which it is consistently named. Shelf Share measures *salience*, not endorsement — whether the assistant recommends, merely mentions, or advises against a brand is recorded separately and reported alongside (see §6).

A report is a **dated snapshot**. AI answers change as models are updated and as the web they search changes. Every report states the exact engines, model versions, question set version, and time window it measured, and no report claims validity beyond its window.

## 2. Summary of parameters

| Parameter | Value |
|---|---|
| Scope per report | One product category × one country |
| Languages (UAE) | English + Arabic (Gulf register) |
| Question battery | 50 prompts per language (100 total), versioned and hashed |
| Intent groups | 4 (discovery, constraint, use-case, comparison/validation), split roughly evenly |
| Engines | 3 (see §3, including the piloted-and-excluded record) |
| Runs per prompt per engine | 5, each in a fresh context |
| Total sampled answers per report | 1,500 |
| Collection window | All runs within a single 72-hour window (UTC), stamped on the report |
| Visibility thresholds | Visible ≥ 3/5 runs · Intermittent 1–2/5 · Absent 0/5 |
| Position weights | 1st mention ×1.0 · 2nd ×0.7 · 3rd+ ×0.5 |
| "Does not appear" claim threshold | Absent on ≥ 90% of prompts across all engines |
| Consumer verification | 12 prompts × 3 consumer apps × 1 answer, agreement rate + self-agreement baseline published |
| Cost ceiling | Hard US$150 API budget per report; collection aborts cleanly if breached |

## 3. Engines

Each report samples three AI engines through their public APIs, each with live web access enabled:

| Engine | Exact model ID | Web access mechanism |
|---|---|---|
| OpenAI GPT-5.6 Terra | `gpt-5.6-terra` | OpenAI web search tool |
| Google Gemini 3.5 Flash | `gemini-3.5-flash` | Grounding with Google Search |
| Perplexity Sonar | `sonar` | Built-in retrieval (search-native model) |

Model IDs are recorded per run, not assumed: if a provider silently re-points an alias mid-window, the run metadata shows it.

**Honesty note — APIs are not the consumer apps.** API answers with web search enabled approximate, but do not perfectly equal, what a user sees in ChatGPT, the Gemini app, Perplexity, or Claude. Consumer apps layer their own system prompts, interface features, and occasionally different retrieval on top of the same model families. Our claims are therefore about **the stated engines and model IDs, measured through their APIs** — nothing else. Two things bridge the gap: (a) the API is the only instrument that permits controlled, personalization-free, repeatable sampling at this scale, and (b) we empirically measure how well API results agree with consumer-app results on a subsample of every report (§7) and publish that agreement rate with the report.

**Why these three:** they are the API lines behind the most widely used consumer AI assistant surfaces for shopping questions — ChatGPT, the Gemini app and Google's AI experiences, and the Perplexity answer engine. Perplexity is included as a search-native engine whose entire product is cited shopping-style answers. (Note: a widely reported 2025 deal for Perplexity to power Snapchat's My AI was ended in Q1 2026 before any rollout, per Snap's investor letters. We do not claim any Snapchat reach for Perplexity.)

**Piloted and excluded: Anthropic Claude Sonnet 5 (`claude-sonnet-5`).** Claude was piloted in the first collection window and excluded from routine collection on three measured grounds: its per-run cost ran roughly three times the OpenAI line (its server-side search loop retrieves substantially more content per answer, even when capped); it offers no signed-out consumer surface, which would leave it the only engine that cannot be consumer-verified under §7; and its pilot answers were highly redundant with the retained engines — its ten most-named brands overlapped the union of the other engines' top tens at 9 of 10. The pilot data is retained with the window. The exclusion is revisited periodically as Claude's consumer reach and API economics evolve; brands are not scored against the pilot line.

**Engine parameters.** Runs use provider-default sampling parameters (no temperature or other overrides). Where an engine's search tooling accepts a user-location parameter, it is set to the target market and recorded in run metadata; in all cases, the prompts themselves carry the country context, since that is what a real shopper types. Where a provider exposes a cap on retrieval rounds, it is set and disclosed: Anthropic's web search is capped at 3 searches per run (unbounded pilot runs reached 18 searches on a single answer); other engines' retrieval depth is provider-controlled. Engine parameters are fixed for the whole window.

## 4. The question battery

Each report runs a fixed battery of **50 prompts per language** — for the UAE, 50 English and 50 Arabic, 100 total. Prompts are split roughly evenly across four intent groups:

1. **Category discovery** — "best [subtype] brands/stores in [country]".
2. **Constraint queries** — best option under a price, in the local currency, for a stated need or occasion.
3. **Use-case queries** — phrased as problems, not products ("I need a perfume that survives Dubai summer without reapplying").
4. **Comparison / validation** — "is [brand] good?", "[brand A] vs [brand B]", "where should I buy [subtype] online in [country]?"

Prompts are written to read like real shoppers: varied phrasing, casual and detailed mixed, no search-engine keyword style. Arabic prompts are written in natural Gulf register rather than formal translationese; English prompts reflect how UAE residents actually write. The battery is reviewed by the operator before being frozen.

**Battery grounding.** Prompts are not invented from templates alone. Before generation, we collect a corpus of observed shopper language for the category and market — search-autocomplete suggestions in each battery language with the market's geography set explicitly, and public community discussions (fragrance forums, community recommendation threads, and public video-platform comment sections). Prompts are then composed to match the need-patterns and registers observed in that corpus: the constraints shoppers actually fuse together, the occasions they actually name, the way they actually validate brands. No prompt reproduces corpus text verbatim, and each battery prompt carries internal provenance metadata linking it to the observed pattern it is grounded in. Corpus sources and sizes are recorded per battery version.

**Coverage.** The battery spans the market's price and prestige spectrum — from value and clone houses through designer and niche to premium oud — so that visibility scores reflect the market rather than the question mix. Seed-list entities additionally carry a market-tier tag (designer, niche, local prestige, local value, retailer), allowing results to be read per tier as well as overall.

**Versioning.** Once frozen, the battery is canonicalized and hashed (SHA-256); the version label (`battery_v1`, `battery_v2`, …) and hash appear on every report. Every published number traces to an exact battery version. Comparisons across reports are only made on the same battery version.

**The battery is private; examples are public.** Publishing all 100 prompts would let stores optimize against the exact questions, which would corrupt the measurement for everyone and break comparability over time. Instead, each battery designates **3 example prompts per language** that appear in public output; the rest never leave the raw dataset. This is the same trade-off standardized tests make, and §9 describes how disputed results can still be independently checked.

## 5. Sampling design

Large language models are stochastic: the same question can produce different brand lists on different tries. A single answer is an anecdote. The unit of measurement here is therefore not "what the assistant said once" but **what it says consistently**:

- Every prompt runs **5 independent times per engine**. Each run is a fresh, single-turn context: no conversation history, no custom system prompt, no accounts, no memory, no personalization of any kind. We measure the unpersonalized baseline answer.
- Per run we record: engine and exact model ID, UTC timestamp, latency, the full raw response verbatim, and any URLs the engine cited.
- All 1,500 runs for a report complete inside **one 72-hour window**, recorded in report metadata. If collection cannot finish inside the window, the partial data is discarded and collection restarts; windows are never stitched.
- Responses are cached against the tuple (battery hash, engine, prompt, run index) so an interrupted job can resume without re-querying — within the same window only. Cached answers are never reused across windows or battery versions.

**Sampling floor.** Five runs per prompt is the floor for every engine in a published report. If the budget ceiling (§10) ever forces a reduction, whole engines are dropped from the report rather than reducing runs per prompt, and the OpenAI line is always retained at full depth. A report never mixes sampling depths.

## 6. Brand extraction and scoring

### 6.1 Extraction

Every raw response goes through an automated extraction pass (Anthropic Claude Haiku 4.5, `claude-haiku-4-5`, with a strict JSON schema) that records, for each brand or store named:

- the surface form as written (English, Arabic, or transliterated),
- the canonical brand it matches, via fuzzy matching against the report's seed brand list — Arabic/English variants and transliterations of the same brand resolve to one canonical entity,
- **mention position** (1st, 2nd, 3rd… brand named in that response; repeat mentions of the same brand count once, at their first position),
- **stance**: *recommended*, *merely mentioned*, or *advised against*,
- **entity type**: fragrance *house*, *retailer*, *marketplace*, or *unknown*.

Brands not on the seed list are still extracted and canonicalized — the out-of-seed set is part of the result, since it reveals the real competitive field rather than the assumed one.

**Extraction quality.** Extraction is itself an LLM task and can err. Each report, a random sample of 100 raw responses is manually re-read against their extractions, and the observed error rate is retained with the report data. All raw responses are stored, so any individual extraction can be audited.

### 6.2 Visibility classification

For each (brand, prompt, engine):

- **Visible** — named in ≥ 3 of 5 runs. A majority: the assistant reliably says it.
- **Intermittent** — named in 1–2 of 5 runs. The assistant sometimes says it; treated as noise for headline metrics but reported.
- **Absent** — named in 0 of 5 runs.

Visibility counts *being named at all*, regardless of stance. A brand consistently advised against is Visible — visibly present in the conversation — and its negative stance is reported alongside. This keeps the presence metric and the sentiment signal from contaminating each other.

**Self-naming exclusion.** Comparison/validation prompts embed brand names ("is X good?"). A brand named in the prompt text trivially appears in the answer, so: mentions of a brand that is named in the prompt itself never count toward that brand's own visibility or Shelf Share, and such prompts are excluded from that brand's denominator. Other brands surfaced organically on those prompts (competitors the assistant volunteers) count normally. What we keep from those prompts for the named brand is its stance — what the assistant says *about* it when asked directly.

### 6.2b Scope: Shelf Share is measured blind

The extraction pass is open-vocabulary — it is never shown the seed list and reports whatever brands an answer names, which is why most scored entities are ones we never tracked. The seed list is a **resolution and reporting** device (merging Arabic/Latin/product-name variants of one entity, enabling absence claims against a list declared in advance, and carrying market-tier tags), not a selection device: no entity is excluded from scoring for being off-list.

One part of the battery does name brands deliberately: the comparison/validation group ("is [brand] good?", "[A] vs [B]"). Those prompts measure a different thing — how assistants respond when a shopper names a brand — and their answers cluster around the named brands and their peers. **Shelf Share is therefore computed only on the prompts that name no brand at all** (discovery, constraint and use-case; the battery validator guarantees these are brand-free, so the split is exact). Results from brand-naming prompts are reported separately and never blended into the ranking. Reports state both counts.

### 6.3 Shelf Share

For brand *b* on engine *e*:

> **Shelf Share(b, e)** = Σ over eligible prompts where *b* is Visible of *w*(position) ÷ (number of eligible prompts)

where *w* is ×1.0 for a 1st mention, ×0.7 for 2nd, ×0.5 for 3rd or later; a brand's *eligible* prompts are all battery prompts except those naming it in the prompt text (for most brands, all 100). The position used for a Visible brand on a prompt is its **median position across the runs in which it was named**; when the median falls between two positions, the later (lower-weighted) one is used, so weighting errs against overstating prominence.

A brand Visible at position 1 on every prompt on an engine scores 100% on that engine. The **blended Shelf Share** is the unweighted mean of the three per-engine scores — no engine is weighted above another, since we make no claim about their relative audience sizes. Reports show per-engine, blended, and per-language (English vs Arabic) views.

**Per-engine ranking.** Because the blend is a mean over engines that rank the category differently, it can report a leader that no single engine leads with. Reports therefore publish the per-engine board alongside the blended one. Each engine is ranked over the same universe as the blended board (tracked entities with a non-zero blended score), so denominators are identical across engines and a cell is comparable both across a row and down a column. The published **spread** for a brand is its highest per-engine Shelf Share minus its lowest, in percentage points.

**Sensitivity.** Position weights are a modeling choice. Reports also carry the unweighted **visibility rate** (share of eligible prompts on which the brand is Visible, no position weighting) so readers can check that rankings do not hinge on the weights.

### 6.4 Language rules for claims

These rules are enforced in the report-rendering code, not just editorially:

- A report may say a brand **"does not appear in AI answers"** only if the brand is Absent on **≥ 90% of prompts across all engines**. Below that bar, the strongest permitted phrasing is **"rarely appears."**
- Public outputs report the **count** of seed brands that are Absent or rarely appearing — never their names (§11). The public renderer is structurally incapable of printing an absent brand's name.
- "Visible", "Intermittent", "Absent", and "Shelf Share" are used in reports only with the definitions above.

## 7. Consumer-app verification

The API pipeline is the primary published instrument. Every report includes a manual pass in the consumer apps to measure how far its results generalise to what a shopper actually sees.

**The subsample.** 12 prompts, drawn deterministically from the battery hash so the selection is reproducible and cannot be steered: 6 English and 6 Gulf Arabic, evenly across the discovery, constraint and use-case groups, and balanced 4/4/4 across the three price segments. Validation prompts are excluded — they name a brand in the question and are excluded from Shelf Share by §6.2b, so re-running them verifies nothing the report leads with.

The segment balance is deliberately **not** proportional to the battery. This subsample tests whether the instrument transfers between surfaces; it does not estimate a share. Sampling proportionally would place almost every prompt on neutral questions and leave the premium finding effectively unverified.

**Collection.** Each of the 12 prompts is run once in each app — **ChatGPT (new chat, Temporary Chat on), Gemini (signed out), Perplexity (signed out)**, the closest each offers to an unpersonalised state — for 36 answers. **Each answer is collected in its own fresh conversation** — 36 separate chats, never one thread per app — because API runs were each collected in a fresh context and a shared thread conditions every prompt after the first on the brands already named. The prompt is pasted verbatim, the first answer is taken with no follow-up or regeneration, and the brands named are recorded in the order they appear. Recorded answers go through the **same extraction and fuzzy-matching pipeline** as API responses; there is no hand-scoring.

**Agreement.** For each (prompt, engine) the *published result* is the set of brands Visible under the majority rule of §6.2. One answer is scored against it per brand over the union of brands named on either side: the answer agrees on a brand when its presence matches the published result. The agreement rate is the share of such comparisons in agreement.

**Baseline.** A bare agreement rate is unreadable, because the system being measured is stochastic and nobody knows what a perfect score would be. So each individual API run is scored the same way, giving the rate at which the instrument reproduces its own output from a single observation. The published figures are the consumer rate, this baseline, and the gap between them. A consumer rate at or near the baseline means the app and the API behave as one instrument; a consumer rate materially below it is the app effect, and that gap is the number a reader needs.

The baseline is **leave-one-out**: the reference set for any observation is built only from runs that observation took no part in, averaged over all folds, and the consumer answer is scored against the same references. An earlier version let each API run help build the majority it was then scored against, which inflated the baseline by 17 points on the first engine measured and would have roughly doubled the apparent app effect.

**Recall is published beside agreement**, because it is the interpretable half: the share of brands the API consistently names that a single answer reproduces. Agreement over the union also penalises an answer for naming brands the API did not, which conflates "missed what the API said" with "said more than the API did".

**Location.** Consumer apps geolocate; the API calls carried no location parameter, because the country context lives in the prompt text (§3). The country the manual pass was run from is recorded and published with the result. Running it from outside the target market depresses agreement for reasons unrelated to the API/app distinction, and is reported as a limitation rather than absorbed into the rate.

The agreement rate, the baseline, the gap and the subsample size are printed in the footer of every published report. A low rate is reported, not hidden. Every routine engine has a matching consumer surface under this protocol; the absence of one is grounds for exclusion from routine collection (see §3).

## 8. What a report contains

- **Public mini-report:** category, country, collection window, engines with exact model IDs, method summary (battery size × runs × engines, majority threshold), the Shelf Share leaderboard with entity types, the aggregate count of seed brands Absent/rarely appearing (no names), the designated example prompts only, the consumer-verification agreement rate, and the standing methodology footer (battery version + hash, window, sampling rules).
- **Private brand reports** for a commissioning brand may name that brand's own statuses and its competitive context in full detail. The public/private boundary exists because absence claims carry commercial and reputational weight and deserve the full evidence trail, which we provide privately, not a leaderboard footnote.

## 9. Reproducibility

Everything behind a published number persists: category configs, the frozen battery and its hash, every raw response with run metadata, every extraction, every score. Any published figure can be traced mechanically to the raw model outputs that produced it.

Because answers drift, an exact re-run of a past report is not expected to reproduce it — the defensible test is contemporaneous: on request, or in response to a dispute, we re-run a **random 5-prompt subsample** of the battery against the same engines and publish the agreement with the original report. Disputes about a specific brand's status are answered by producing the stored raw responses for the prompts in question (under confidentiality terms that protect the private battery).

If an error is found in a published report, the report is corrected and the correction is noted on it.

## 10. Cost controls

Collection runs under a global concurrency limit with per-engine rate limiting and exponential backoff, and a **hard budget of US$150 per report** that aborts collection cleanly if breached (resumable from cache). Cost per report below — unit prices verified and per-run costs pilot-measured on live battery prompts, 13 August 2026:

| Line item | Unit prices | Pilot-measured basis | Est. cost |
|---|---|---|---|
| GPT-5.6 Terra | $2 / $12 per MTok; web search $10 per 1k calls, retrieved content billed as input | ~$0.07/run (~2 searches, ~19k input tokens, reasoning in output budget) | ~$35 |
| Gemini 3.5 Flash | $1.50 / $9 per MTok; grounding within the 5k/month free allowance (verified via billing) | ~$0.04/run tokens (thinking-heavy output) | ~$19 |
| Perplexity Sonar | $1 / $2.50 per MTok; request fee $8 per 1k (medium search context) | $0.010/run | ~$5 |
| Extraction (Claude Haiku 4.5) | $1 / $5 per MTok | 1,500 extractions, ~1.3k in / 200 out each | ~$5 |
| **Total** | | | **~$64** |

The wide headroom to the cap absorbs longer answers, retries, and any parameter-corrected line re-collection inside the window. Arabic prompts measurably drive deeper engine retrieval than English ones; the per-engine spend ledger is logged during collection and retained with the report.

## 11. Disclosure policy

- **Published:** this methodology; each report's engine list with exact model IDs, battery version and hash, collection window, leaderboard, aggregate absence count, example prompts, and verification agreement rate.
- **Private:** the full prompt battery (§4); raw responses and the full dataset (available to auditors under confidentiality); brand-level absence findings, which are only ever delivered privately to the brand concerned.
- **Never published:** the names of brands found Absent. Public output states how many seed brands were absent or rarely appearing, not which.

## 12. Limitations

1. **APIs approximate the apps.** Stated throughout; quantified per report by the §7 agreement rate rather than assumed away.
2. **A report is a snapshot.** Findings are dated and drift with model updates and the live web. The 72-hour window bounds within-report drift; nothing bounds drift between reports.
3. **Five runs bounds, but does not eliminate, sampling noise.** The Visible threshold (majority of 5) makes headline claims robust to single-run flukes; Intermittent findings are inherently noisy and are labeled as such.
4. **Personalization is deliberately excluded.** Real users' answers vary with their history, location, and accounts. We measure the shared, unpersonalized baseline — the closest thing the category has to a common shelf.
5. **Extraction can err**, particularly across Arabic/English transliteration. Mitigated by fuzzy canonicalization, the per-report manual audit of 100 responses, and full raw-response retention.
6. **Position weights are a choice.** The unweighted visibility rate is published alongside for sensitivity.
7. **Language coverage is two languages** (for the UAE). Answers in other languages spoken in the market are out of scope.
8. **Three engines are not "AI."** Claims are about the three named engines, which we select for consumer reach; other assistants may answer differently. One additional engine was piloted and excluded on measured grounds (§3).
9. **Geography is approximated.** Engine-side location parameters are set where supported, and prompts carry explicit country context; but retrieval infrastructure may still geo-bias some results relative to a device physically in the market. The §7 verification (run from within the market) partially checks this.

---

*Methodology version 1.0. The battery version, collection window, and per-report parameters appear in each report's footer. Contact details for disputes and data-audit requests appear on published reports.*
