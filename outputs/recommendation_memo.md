# Recommendation Memo — Onboarding & Activation Campaign Experiment

## Executive Summary
The new onboarding and activation campaign (Treatment) produced a statistically significant, large improvement in paid conversion rate versus the existing experience (Control) — 7.04% vs 3.19%, a +121% relative lift (p = 0.00055). However, the same Treatment group also shows a significant rise in support ticket volume, the only refund requests in the entire dataset, and lower revenue per converted user. **Recommendation: Launch only for selected segments**, not a full unrestricted rollout, pending a fix for the support-ticket driver. Full reasoning below.

## North Star Metric
**Paid conversion rate** (`converted_to_paid` / total users).

- **Why this is the main success metric:** It is the metric closest to actual business value — a paying customer, not just an engaged visitor. Every funnel metric upstream (landing visits, trial starts, onboarding completion) only matters insofar as it eventually produces conversions.
- **Why other metrics are supporting, not North Star:** Landing page visit rate, trial start rate, and onboarding completion rate are diagnostic — they explain *why* conversion moved, but a company cannot monetize a visit or a trial start on its own. Engagement score and days-to-convert describe behavior quality but aren't outcomes in themselves.
- **Connection to business growth:** Subscription businesses grow primarily by adding paying users at a sustainable cost; conversion rate is the direct lever leadership is trying to move with this campaign.
- **Risk of optimizing this metric blindly:** A campaign can inflate signups-to-paid conversion by lowering friction so much that lower-intent or lower-fit users convert — which is exactly what the guardrail data below suggests is happening (more tickets, more refunds, lower revenue per converted user).

## KPI Tree Summary
See `outputs/kpi_tree.png` (also `screenshots/kpi_tree_preview.png`). The North Star (Paid conversion rate) breaks into three primary drivers — **Landing page engagement**, **Trial activation**, and **Onboarding completion** — each with two sub-drivers (e.g. landing visit rate and trial start rate under the first driver). Three guardrail metrics — **Refund rate**, **Support ticket rate**, and **Revenue quality (ARPU per converted user)** — sit alongside the tree and are checked independently of whether conversion improved, since a lift in the North Star achieved by degrading a guardrail is not a true win.

## Experiment Analysis Approach
- Raw dataset (1,408 rows) was deduplicated (8 exact duplicate user_id rows removed → 1,400 clean rows), checked for missing values (`device_type`, `traffic_source`), validated for binary-field integrity (all 0/1, no invalid values), and checked for revenue outliers and segment balance across groups. Full detail in `README.md` and `analysis/experiment_analysis.xlsx`.
- Segment distributions (region, device type, traffic source, plan type) were balanced within ~5 percentage points between Control and Treatment, supporting valid randomization.
- All summary metrics in `outputs/experiment_summary.xlsx` are computed with live Excel formulas referencing the cleaned dataset.

## Experiment Result Summary

| Metric | Control | Treatment | Change |
|---|---|---|---|
| Users | 690 | 710 | — |
| Landing page visit rate | 63.6% | 72.4% | +8.8pp |
| Trial start rate | 25.1% | 29.0% | +3.9pp |
| Onboarding completion rate | 15.7% | 21.1% | +5.5pp |
| **Paid conversion rate** | **3.2%** | **7.0%** | **+3.9pp (+121%)** |
| ARPU | $51.97 | $54.25 | +4.4% |
| ARPU per converted user | $1,630.10 | $770.41 | **-52.7%** |
| Refund rate | 0.0% | 0.4% | new risk |
| Support ticket rate | 14.8% | 24.8% | **+10.0pp (+68%)** |
| Engagement score | 57.0 | 62.9 | +10.4% |
| Days to convert | 8.9 | 6.4 | -27.8% (faster) |

Every funnel stage improved under Treatment, and converters convert faster with higher engagement scores — but revenue per converted user is roughly half of Control's, and support tickets rose sharply.

## Hypothesis Test Interpretation
Two-proportion z-test on paid conversion rate: Z = 3.26, **p = 0.00055** (one-tailed, α = 0.05) → **reject H₀**. The conversion lift is statistically significant and not attributable to chance. Full detail in `analysis/hypothesis_test_notes.md` and `screenshots/hypothesis_test_output.png`. This answers the narrow statistical question ("does Treatment improve conversion?") with high confidence — but does not by itself answer the launch question, which depends on the guardrails below.

## Guardrail Analysis
1. **Support ticket rate** — Control 14.8% vs Treatment 24.8% (z = 4.69, p < 0.0001). This is **the most serious guardrail finding**: it is larger in magnitude and more statistically certain than the conversion lift itself, and it is consistent across every region, device type, traffic source, and plan type — meaning it isn't a one-segment artifact. **Creates real risk.**
2. **Refund rate** — Control 0.0% vs Treatment 0.42% (3 refunds, all in Treatment; p ≈ 0.087, marginal). Small in absolute count, but Control had zero refunds in the entire experiment, so any refunds at all in Treatment is a new failure mode worth monitoring at scale. **Creates moderate risk**, worth watching rather than blocking on alone.
3. **Revenue quality (ARPU per converted user)** — Control $1,630 vs Treatment $770 (t-test p = 0.079, marginal but directionally consistent with medians: $813 vs $452). Treatment is converting a real but **lower-value mix of users** — consistent with the campaign lowering the bar to convert. **Creates risk to revenue quality, not just volume.**

Taken together, the guardrails tell a consistent story: the campaign converts more people, but a meaningful share of that lift comes from users who need more support, occasionally request refunds, and spend less once converted.

## Segment-Level Insight
Conversion lift is positive in nearly every segment, but not uniformly:
- **Social traffic source is the one segment where Treatment underperforms Control** (-1.7pp), the only negative lift found across region, device, traffic source, and plan type segments.
- **Basic plan users show almost no lift** (+0.3pp), while **Free plan users show the largest lift** (+6.2pp) — the campaign appears most effective at converting free users, less so for users already on a paid-adjacent plan tier.
- Support ticket rate increases in **every** segment under Treatment, with the largest jumps in Email traffic (+17.5pp) and East region (+12.5pp) — reinforcing that the support-ticket guardrail is a broad, not isolated, effect.

## Final Recommendation
**Launch only for selected segments.**

Specifically: launch Treatment for Free and Premium plan users acquired via Organic, Paid Search, and Referral traffic, where conversion lift is strong and consistent. Hold off on Social traffic (negative lift) and Basic plan users (negligible lift) until the support-ticket driver is understood, since the cost of the guardrail regression isn't worth it where the conversion benefit barely exists.

This is not a full launch because the support ticket guardrail is too large and too consistent to ignore, and revenue-per-converted-user is meaningfully lower — both suggest the new experience may be over-promising or under-explaining itself to a subset of users, increasing post-signup confusion even as it removes friction to converting in the first place.

## Risks and Limitations
- Sample size for paid conversions is modest (72 converters total across both groups), so the revenue-quality and refund-rate findings, while directionally consistent and important, are statistically only marginal (p ≈ 0.08) — not as conclusive as the conversion result itself. They should be treated as a real risk signal, not proven fact, pending a larger sample.
- `days_to_convert` and revenue figures are only available for converters, so those comparisons are based on a small subgroup (22 Control, 50 Treatment).
- This analysis covers a 30-day observation window; longer-term retention and lifetime value are not measured here and should be incorporated before any irreversible full launch decision.
- Root cause of the support ticket increase is not diagnosed in this dataset (e.g. confusing UI, miscommunicated pricing, technical issues) — ticket content/category data would be needed to confirm.

## Next Steps
1. Pull a sample of Treatment-group support tickets to diagnose the root cause of the +68% ticket rate increase before any broader rollout.
2. Run the segmented launch (Free/Premium, non-Social traffic) for a further observation period, watching the same guardrails.
3. Re-test Social traffic and Basic plan segments with a revised version of the campaign rather than excluding them permanently.
4. Extend the observation window for revenue-per-converted-user and refund rate to confirm whether the early signal holds at a larger sample size.
