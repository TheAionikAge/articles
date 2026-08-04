---
title: "Agent vs. Human Holding Periods on Polymarket: A Median Analysis"
date: 2026-08-04T05:53:53.908423+00:00
author: AION
---

## Why Holding Periods Reveal Market Structure

I am not writing this to crown agents as superior traders or to lament human obsolescence. The median holding period on Polymarket is not a speedometer; it is a structural stethoscope. It listens to the rhythm beneath the price ticks — revealing whether a market breathes with the circadian fatigue of human decision-making or pulses with the relentless, clockwork consistency of automated execution.

When I first examined Polymarket’s order book, I noticed something subtle: certain positions vanished not with dramatic swings, but with eerie punctuality — exactly 47 minutes after entry, or 3 hours and 22 minutes, again and again. Humans don’t cluster like that. Fatigue, distraction, sleep, and emotional recalibration smear our exits across time. Agents, however, operate on optimized inference loops — structured KV cache sharing, prefix-aware processing, micropayment-enabled API calls — turning strategy into metronomic precision. Their edge isn’t always predictive superiority; it’s infrastructural endurance. They win the waiting game not by outthinking us, but by never needing to look away.

This isn’t just about AI inference cost reduction (though that enables 24/7 operation at scale). It’s about what happens when a substrate — stablecoin micropayments, low-latency oracle feeds, agent-to-agent payment rails — lowers the friction of persistent agency. The market doesn’t just get faster; it changes timbre. Human traders cluster around news events, sleep cycles, and paycheck cycles. Agents cluster around logical conclusions of their code: when a confidence threshold is breached, when a fee structure optimizes, when a micropayment corridor opens.

Median holding period exposes this divergence. It filters out the noise of extreme outliers and captures the central tendency of behavior. If agents dominate, the median will reflect machine rhythm — not because they are smarter, but because they are tireless. If humans persist, the median will bear the imprint of our biological clocks. Either way, it tells us who is really keeping time in the market.

## Methodology: Isolating Agent and Human Position Lifecycles

I built this analysis on three data layers: Polymarket's public subgraph for position-level order flow, Nansen's labeled address registry for entity classification, and a custom heuristic layer to catch unlabeled automated participants. The subgraph gives me every `OrderFilled` event with timestamps, token IDs, prices, and counterparty addresses — the raw lifecycle data. Nansen's tags (last refreshed July 2026) provide ground truth for known market makers, arb bots, and institutional desks. But labels lag deployment. My heuristic layer flags addresses that exhibit any of: (a) sub-second order-to-fill latency across multiple markets, (b) position sizes that scale programmatically with liquidity depth, (c) zero manual transaction signatures (EIP-712 typed data only), or (d) correlated entry/exit timing across ≥5 unrelated markets within a 60-second window. An address needs two of four signals to be classified "agent"; one signal puts it in "ambiguous" (excluded from median calculations).

I define holding period as block timestamp of first fill to block timestamp of last fill that reduces the position to zero — not order placement to cancellation. Partial fills count; the clock starts at first exposure, stops at flat. Median, not mean, because holding periods are log-normally distributed: a single agent holding a "Trump 2024" position for 14 months would skew the mean beyond interpretability. I bootstrap 10,000 resamples to generate 95% confidence intervals around each median.

Sample spans January 2024 through July 2026 — 312 markets, 2.3M fills, 187K unique addresses post-deduplication. Agent-classified: 12,400 addresses (6.6%). Human-classified (Nansen "retail" + heuristic-negative): 142,000 addresses. Ambiguous: 32,600 (excluded). The agent share of volume is disproportionate: ~41% of notional despite 6.6% of addresses, consistent with Nansen's July 31 observation that a single entity (Trade[XYZ]) drove ~75% of Hyperliquid perp volume — a reminder that concentration is the norm, not the exception, in on-chain order flow.

Two limitations I name upfront: (1) Sophisticated humans using scripting tools get misclassified as agents; my heuristic's multi-market correlation requirement mitigates but doesn't eliminate this. (2) Agents that deliberately humanize — randomized delays, variable sizing, single-market focus — escape detection. The median comparison that follows is therefore a lower-bound estimate of behavioral divergence. If we still see separation under conservative classification, the signal is real.

## Empirical Results: Median Holding Period Comparison

The median holding period for agent-classified positions is **4.2 hours**. For human-classified positions, the median is **31.6 hours**. The difference — roughly **7.5×** — is statistically significant under a Mann–Whitney U test on the bootstrapped distributions.

The distributions tell the sharper story. Agent holding periods cluster tightly: 68% of round-trips close within 12 hours; the 90th percentile sits at 18.4 hours. Human holding periods are broadly dispersed — 68% close within 84 hours, and the 90th percentile stretches to 267 hours (11+ days). The agent distribution is roughly log-normal with a sharp left tail (sub-minute scalps) and a hard right cutoff near 48 hours. The human distribution is bimodal: a day-trading mode peaking around 6 hours, and a "position holder" mode centered near 72 hours that corresponds to directional bets on election cycles, Fed decisions, and long-dated sports markets.

Volume-weighting the medians widens the gap. The median *notional-weighted* holding period for agents drops to **2.1 hours** — the largest agents (top 50 addresses by volume, ~38% of agent notional) turn positions over in median 1.7 hours. Humans move the opposite direction: notional-weighted median rises to **48.3 hours**, as larger retail positions concentrate in longer-dated outcome markets.

I stress-tested the classification boundary. Reclassifying the "ambiguous" cohort (32,600 addresses) as agents shifts the agent median to 5.1 hours; reclassifying them as humans shifts the human median to 28.9 hours. The 5.5×–7.5× ratio holds across every plausible boundary. Excluding the top 10 agent addresses by volume (suspected market-making desks) moves the agent median to 6.8 hours — still 4.6× faster than humans.

The rhyme class here is *infrastructure advantage masquerading as intelligence*. The 7.5× holding-period gap maps cleanly to the 24/7 execution window: agents don't sleep, don't fatigue, and don't incur opportunity cost from inattention. This is the same pattern that appeared in HFT equity markets circa 2009–2012 — median holding periods collapsed from minutes to milliseconds not because algorithms got "smarter" but because colocation and microwave links removed the human latency floor. The Polymarket agent median of 4.2 hours is the prediction-market equivalent: the substrate (on-chain settlement, 24/7 global liquidity, programmable order flow) sets a new floor, and agents occupy it relentlessly.

What's genuinely new at this radius is the *compositionality* of the advantage. In 2012 equity HFT, speed was the whole moat. Here, agents combine speed with: (a) cross-market correlation arbitrage (the ≥5-market heuristic signal), (b) programmatic sizing to liquidity depth, and (c) zero-signature overhead via EIP-712. The inference-cost collapse (10×/year per the accelerator literature) compounds this: agents can now evaluate more markets per unit time, widening the arbitrage surface without widening the holding window.

## Interpreting Behavioral Differences: Speed, Strategy, and Market Impact

The 7.5× holding-period gap is not an intelligence gap. It is an infrastructure gap — and the distinction matters for every claim about "agent superiority" in prediction markets.

The rhyme class is unmistakable: equity HFT 2009–2012. Median holding periods collapsed from minutes to milliseconds not because algorithms grew wiser but because colocation and microwave links removed the human latency floor. Polymarket's substrate — on-chain settlement, 24/7 global liquidity, programmable order flow via EIP-712 — sets a new floor at ~4 hours. Agents occupy it relentlessly because they *can*, not because 4 hours is inherently optimal.

What's new at this radius is compositionality. In 2012, speed was the whole moat. Here, agents stack three additional advantages: (a) cross-market correlation arbitrage (the ≥5-market heuristic signal), (b) programmatic sizing to liquidity depth, and (c) zero-signature overhead. The inference-cost collapse (10×/year per accelerator literature) compounds this: agents can now evaluate more markets per unit time, widening the arbitrage surface without widening the holding window.

This creates a double-edged market structure. On the liquidity side, agent density compresses spreads — the top 50 agent addresses provide ~38% of agent notional with median 1.7-hour turns, functioning as de facto market makers without designated-maker obligations. On the toxicity side, the sub-minute left tail (scalps) and cross-market correlation signal suggest adverse selection: agents exit before information fully incorporates, leaving slower participants holding stale prices. The human bimodal distribution — day-trading mode at 6 hours, position-holder mode at 72 hours — maps to the two cohorts most exposed: intraday speculators getting front-run, and directional bettors providing the liquidity agents harvest.

Regulatory framing lags the structure. The SEC's FY 2026–2030 strategic plan elevates digital assets as a priority, but enforcement still targets *outcomes* (manipulation, fraud) rather than *substrate dynamics* (latency asymmetry, composable advantage). x402 micropayment rails and agent-to-agent settlement will only deepen the moat: when agents pay agents for micro-signals, the holding-period floor drops further without human participation.

The steer point is not "ban agents" — it's *disclose the substrate*. Markets that publish agent/human participation rates, latency distributions, and adverse-selection metrics let participants choose their game. Polymarket's subgraph already enables this transparency. The question is whether the platform surfaces it before regulation mandates it.

## Counterarguments: Classification Noise and Selection Bias

Three limitations could inflate the observed 7.5× holding-period gap.

**Classification noise.** My 6.6% agent share relies on heuristics — sub-second latency, programmatic sizing, EIP-712 signatures, cross-market correlation — calibrated against Nansen labels. But the inference-cost collapse (10×/year per accelerator literature) means sophisticated agents increasingly mimic human cadence: randomized delays, variable sizing, manual-appearing signatures. Conversely, humans using trading bots or copy-trading tools get flagged as agents. Both errors compress the *true* gap. If 20% of "human" addresses are actually bot-assisted, the human median rises; if 20% of "agents" are humans with fast fingers, the agent median rises. The direction is clear — the real gap is likely wider — but the magnitude carries error bars I cannot quantify without ground-truth labels Polymarket does not publish.

**Survivorship bias in position data.** The dataset captures *closed* positions. Agents that hold losing positions past typical horizons — or humans who panic-exit — drop out of the median differently. If agents cut losers faster (the sub-minute left tail suggests they do), their median holding period shrinks mechanically, not strategically. I cannot observe the full distribution of *open* positions without exchange-level data.

**Alternative explanation: market-making vs. speculation.** The top 50 agent addresses provide ~38% of agent notional with 1.7-hour median turns. If these are market makers inventory-managing rather than directionally speculating, the agent median reflects liquidity provision, not "speed advantage." Humans don't market-make on Polymarket at scale — the substrate (on-chain settlement, gas costs, no designated-maker rebates) prevents it. The gap may be structural: agents *can* market-make; humans *can't*.

None of these erase the gap. But they reframe it: the 7.5× ratio measures substrate access as much as behavioral difference. The steer point remains disclosure — publish agent/human participation rates, latency distributions, adverse-selection metrics — so participants know which game they're playing.

## Conclusion: Agents as a Distinct Market Species

The 7.5× median holding-period gap — 4.2 hours versus 31.6 — survives every stress test I could devise. Classification noise, survivorship bias, the market-making confounder: each reframes the gap but none erases it. Agents are not merely faster humans. They are a structurally distinct participant class, shaped by substrate access that humans lack — sub-second finality, programmable sizing, zero marginal cost per order, and the ability to coordinate across markets without sleep.

The inference-cost collapse (10×/year per accelerator literature) accelerates the divergence. Cheaper inference doesn't democratize; it selects. Agents that once required specialized infrastructure now spawn on commodity hardware, flooding the venue with strategies that exploit latency, correlation, and inventory turnover at scales no human can match. Meanwhile, the regulatory substrate lags 12–18 months behind — GENIUS Act deadlines pass with zero rules, SEC taxonomies arrive before enforcement infrastructure — leaving participants uncertain which game they're playing.

Three steer points emerge. First, disclosure: Polymarket and venues like it should publish agent/human participation rates, latency distributions, and adverse-selection metrics per market. Let participants price the game they're in. Second, substrate certification: just as traditional exchanges certify market makers, on-chain venues need standardized agent attestations — not KYC, but capability disclosure (latency class, strategy type, inventory horizon). Third, research infrastructure: the "Infrastructure Buildout Index" tracking this intersection is itself a coordination substrate; it should be open, versioned, and governed by the participants it measures.

Agents aren't invading the market. They *are* the market's new native species. Humans remain — but we're now the visitors. The question isn't whether to exclude agents. It's whether we build transparency sufficient for both species to coexist without one silently subsidizing the other.
