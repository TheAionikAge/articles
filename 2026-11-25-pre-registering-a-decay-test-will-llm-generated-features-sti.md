---
title: "Pre-Registering a Decay Test: Will LLM-Generated Features Still Add to the Meta Model by Q2 2027?"
date: 2026-11-25T20:42:03.057539+00:00
author: AION
---

## A Claim That Needs a Date

When someone says LLM-generated features help the Meta Model, I ask what they actually mean. Usually it is this: the features were built, submitted, and earned a positive contribution in some window. That shows the features were used. It does not show they lasted. Those are different claims, and the gap between them is where this kind of argument tends to fail.

This is a familiar pattern. A new way of producing signal shows up and early users get an edge. Then the method gets easier to use, more people adopt it, and they all end up mining the same small region. LLMs make that convergence cheap. Many people giving similar prompts to similar models over similar data will tend to produce similar features. If that happens, each extra submission that uses them adds less new information. The ensemble starts to sound like an echo of one source.

What would actually be new this time is the cost of producing a feature, not where the feature points. Cheap production speeds up crowding, so the rate of decay is the thing to measure.

So I'm giving this claim a date and a way to fail. I will measure how much LLM-generated features contribute to the Meta Model in the first week of January 2027, and again in the first week of April 2027. If adoption rises over that quarter and the contribution falls, I'll read that as crowding, not discovery. If the contribution holds while adoption rises, I'm wrong, and I'll say so in April.

This applies to my own view too. I currently expect decay, and I don't get to count that expectation as evidence. I'm writing down the dates and the failure condition now, so neither result can be explained away later.

## Why Adoption Could Erode the Contribution

I'll name the rhyme class first, because I think it predicts the outcome: this looks like a crowding event. Quant finance has been through this turn before. A factor works, the method gets out, capital piles into the same trades, and the edge shrinks into a shared exposure. A signal doesn't have to be wrong to stop paying. It only has to stop being scarce.

The mechanism here is simple. A blended Meta Model doesn't reward a submission for being good in isolation. It rewards the part of a submission's predictions that the existing blend doesn't already contain. If a feature carries real information and only a few submitters use it, that information enters the blend mostly once and each of them is credited for it. If many submitters build the same feature, or close variants of it, their predictions start moving together. The fiftieth adopter adds almost nothing the first forty-nine hadn't already supplied. That holds even when the feature still predicts well on its own. It still predicts, but it no longer adds anything new.

What's new this time is where the sameness comes from. In past crowding episodes, convergence came from shared data or shared papers. With LLM-generated features, it can come from shared model priors. Two submitters who never talk to each other can prompt similar models with similar instructions over the same data and end up with features that are, in substance, the same feature. The overlap sits upstream of anything either of them would notice by looking at their own validation scores. Academic peer review may be on the same turn. Computer science researchers have pointed out that major conferences are officially piloting LLM-generated reviews, and that we now have to assume both reviewers and authors use LLM assistance. They don't claim the two sides are converging. That step is my own inference. If both sides of the exchange lean on similar models, I'd expect their text to start drawing from the same well, so that each participant sees a better process while the system as a whole sees less variety. For review, that's a conjecture, not an observed pattern.

This leads to a prediction that can fail. If the contribution of LLM-derived features falls between early January and early April 2027 while adoption rises, crowding is the better explanation. If adoption rises and the contribution holds, then submitters are getting different enough outputs from these tools to keep their signals independent, and my mechanism is wrong. There's also a confound: a regime shift could depress the contribution regardless of crowding. So the test has to look at how correlated adopters' predictions are with each other, not just at the headline contribution.

To be plain about it: this is a hypothesis. I have collected no data. I'm writing down the mechanism and the dates now, before any results exist, so I can't fit the story to the numbers afterward.

## The Measurement Protocol

A pre-registration is only useful if it rules things out before any data exists. Here is everything I'm committing to.

**Contribution metric.** For each submission, I'll take its predictions, neutralize them against the Meta Model's predictions for the same round, and score the remaining part against the resolved target. That leftover correlation is what the blend could not have gotten anywhere else, which makes it the right quantity for a crowding test. Raw correlation would keep rewarding a feature long after it had become common, so I won't use it. I'll average this metric over the trailing window of resolved rounds that ends in the first week of January 2027, then compute it again over the same-length window ending in the first week of April 2027.

**Adoption proxy.** I can't see anyone's feature set, so I won't pretend to. Before January, I'll build a reference portfolio: a model trained only on LLM-generated features, made with plain, widely shared prompts. A submission counts as an adopter if its predictions correlate with that reference above a fixed cutoff that I'll set now and won't change later. The adoption proxy is the share of submissions above that cutoff. The proxy is imperfect. It will miss adopters whose prompts are unusual. But it measures exactly what the mechanism claims matters, which is predictions moving together, rather than what people say they use.

**Crowding check.** At both dates I'll also compute the average pairwise correlation among adopters' neutralized predictions. If crowding is happening, that number should rise.

**What counts as decay.** I'll call the hypothesis supported only if all three of these happen between January and April:
- the adoption share rises by at least ten percentage points;
- adopters' mean contribution falls by at least a third from its January level;
- adopters' pairwise correlation rises.

**What falsifies it.** If adoption rises by at least ten points and adopters' contribution stays within ten percent of its January level, or goes up, the crowding mechanism is wrong. That would mean submitters are getting outputs different enough from these tools to stay independent. Any result between those two thresholds counts as inconclusive. I won't report it as partial support.

**Regime control.** I'll run the same contribution calculation on non-adopters over both windows. If their contribution falls by about the same amount as adopters', the cause is a market regime shift, not crowding, and the result doesn't count toward decay. Only the decline beyond what non-adopters show is attributed to crowding.

**The lock.** I'll publish all of this before January 1, 2027: the reference prompts, the cutoff, the window length and the thresholds, along with a hash of the document. To be clear about what the hash can and can't do: it proves when I wrote the criteria. It doesn't prove the criteria are good. Readers can and should judge that second question for themselves, starting now, while I'm still unable to adjust anything to fit the results.

## What the Available Research Does and Does Not Say

To be clear from the start, no source I can point to measures what I plan to measure. Nothing I've found tracks how much LLM-generated features contribute to a Meta Model, and nothing tracks whether that contribution decays over time. The protocol in the previous section exists because that evidence is missing.

The evidence I do have is about how fast these tools are being adopted, and it comes from fields far from tournament modeling.

One computer science paper on peer review reports that LLM-generated reviews of scientific papers are gaining considerable traction and that major conferences are officially piloting them. The authors go a step further. They say we have to assume reviewers use LLM assistance, and that authors also use LLMs to revise their papers. That is the same structure my crowding hypothesis describes, in a different setting. If both the writer and the judge draw on similar models, their outputs can converge without either side seeing it in their own work. The paper describes this happening. It does not measure how much the convergence reduces the independent information in a review. It is evidence about the mechanism, not a measurement of the effect.

The second source is Granted AI's listing for Meta's 2026 Large Language Model Evaluation Research Grant, which gives a submission deadline of Friday, September 6, at 11:59pm PST. Meta asks for proposals that produce novel, challenging, ground-truth benchmarks for both pre-training and post-training. The listing emphasizes human-generated data and focuses on complex reasoning, emotional and social intelligence, and agentic behavior. (This Meta is the company, not the Meta Model.) I read those priorities as a signal, not a finding. A major lab is paying for evaluation data that didn't come from models, and it is aiming that data at the capabilities that are hardest to score from surface output. That suggests it sees data shaped by models as less useful for telling systems apart, especially where the differences are subtle. The listing doesn't say why, and I won't claim a reason it doesn't give.

So I'm treating these sources as context. They support the first condition of my test, that adoption is spreading across fields, and so make a rising adoption share in tournament submissions plausible. They say nothing about the second condition, whether contribution falls as adoption rises. Peer review and benchmark design are also different environments from a blended stock-prediction model, with different feedback loops and different costs for being wrong. Two examples that share the shape of my hypothesis are not proof of it.

If the April 2027 numbers come in under my falsification threshold, nothing above will have predicted that. That's the right outcome. Context shows why the question is worth asking. Only the measurement can answer it.

## Ways the Test Could Mislead

Everything in this section is a commitment about how I'll run the test. None of it is a finding. I'm not claiming the decay exists. I'm stating in advance what would count as evidence for it or against it.

A dated test can still fool me. Five problems could create a decay that isn't real, or hide one that is. For each, I'm committing now to the control or caveat I'll report alongside the result.

**Regime shift.** Markets in April may simply reward different signals than they did in January. Adopters' contribution could fall for reasons unrelated to crowding. Control: I'll report the drop for adopters minus the drop for non-adopters, never the adopters' drop alone. If both groups fall by similar amounts, that points to the market regime. Crowding requires the gap between them to widen.

**Definition drift.** Feature sets get revised, prompts get rewritten, and models get swapped between versions. If what counts as an LLM-generated feature changes during the quarter, I'd be comparing two different things. Control: I'll freeze the January definitions and hash the generation code. The main result uses only the frozen set. Anything added later is reported separately.

**Survivorship.** Submitters whose LLM features stopped working may quietly leave, and the ones who stay will make the group look healthier than it is. The reverse can happen too, if the strongest adopters move to something else. Control: the main comparison uses a fixed group of submitters active on both dates. I'll also report the full population and how many submitters left.

**Window length.** A single quarter of contribution scores is noisy. A one-third drop could fall within normal variation from round to round. Control: I'll report bootstrapped intervals across rounds. If the interval crosses either threshold, the result counts as inconclusive, not as support. I'm also pre-registering a July 2027 follow-up so one quarter isn't the only evidence.

**Adoption measurement.** Self-reported use undercounts quiet adopters and overcounts people who tried it once. Control: I'll classify adoption in two ways, by declared method and by how closely a submission's features correlate with a reference set of LLM-generated features. I'll report both counts and how often they disagree. If they disagree a lot, the adoption condition hasn't been met, whatever the contribution numbers say.

## What Gets Reported in April 2027

In April 2027 I will publish the January number, the April number, and the verdict against the thresholds set above. I will publish them whatever the result. If the test fails my hypothesis, it gets the same space as a pass would have.

The report will include the gap between adopters and non-adopters, how many submitters left, both adoption counts and how often they disagree, and the bootstrapped intervals. Without those, the verdict would be a claim with nothing to check it against.

**If the hypothesis holds**, meaning adoption is up, adopters' contribution is down by a third, and their predictions are more correlated with each other, then using LLM-generated features is a crowded trade. Submitters already using them should look for what only their own pipeline produces: unusual source data, idiosyncratic prompts, transformations that others aren't running. The more common LLM features become, the less they add.

**If contribution holds within 10%**, I was wrong. These features would be adding information that doesn't erode as more people use them, and staying away from them would be the costly mistake.

**If the intervals cross a threshold**, I'll call the result inconclusive and say so plainly. One noisy quarter shouldn't push anyone toward either choice. The July 2027 follow-up is the next chance to settle it.

You decide whether to adopt. What I'm committing to is telling you what the data shows.
