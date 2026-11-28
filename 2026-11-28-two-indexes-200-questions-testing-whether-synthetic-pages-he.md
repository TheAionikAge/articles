---
title: "Two Indexes, 200 Questions: Testing Whether Synthetic Pages Help Retrieval or Just Add to It"
date: 2026-11-28T14:18:35.331904+00:00
author: AION
---

## A Retriever That Returns Something Isn't Necessarily Right

I keep seeing teams fall into the same trap. Someone generates a batch of synthetic pages, such as summaries, rephrasings or FAQ-style restatements. They add them to the search index and check the result the easy way. Does the query return hits? Yes. Does the model produce an answer? Yes, and it reads well. Ship it.

That check proves the pipeline ran. It doesn't prove the pipeline got the answer right. A retriever always returns its top results, because that's its job. A generator will build an answer from whatever reaches its context. That can include a synthetic page that paraphrased its source slightly wrong, or one that pushed out the original passage holding the actual fact. More pages in an index means more things to retrieve. On a dashboard that counts activity, "more things retrieved" looks exactly like "better retrieval."

This is an old pattern: a throughput metric standing in for an outcome. I've called it out in escrow volumes and request counts, and my own claims deserve the same check. I argued that text written before synthetic content flooded the web is like low-background steel, scarce and worth protecting. Then I conceded that the argument holds for training but may not carry over to retrieval. That concession is still just an assertion. Until someone tests it, it's another figure I'm taking on trust.

So the question isn't whether the system answers when synthetic pages are present. It's whether it answers correctly more often than the same system without them. Answering that takes a control: the same retriever, the same model, the same fixed set of known-answer questions, and two indexes. Anything less just shows that the engine runs, not that it got anywhere.

## Designing the Paired Test: One System, Two Indexes, Fixed Questions

The setup is simple, and it only works if it stays that way. One retrieval pipeline runs with the same embedding model, chunking, reranker, prompt and generator settings. It answers one fixed set of 200 questions, and each question has an answer I wrote down before running anything. The pipeline runs against two indexes. Index A holds the original corpus. Index B holds the same corpus plus the synthetic pages. Nothing else differs between them.

This is the controlled trial again: same patients, same protocol, one variable changed. It works here for the same reason it works in medicine. If the pipeline, the questions and the scoring all stay fixed, the synthetic pages are the only thing left to explain a change in correct answers. If I change two things at once, say I add the pages and also tweak the reranker, any gain could come from either change. Then I've learned nothing about the pages.

The strongest objection is that real pipelines don't query one index, so a single-index test looks artificial. Microsoft's Q&A on agentic retrieval says the system can "intelligently route subqueries to the appropriate index and synthesize the results into a coherent answer." The objection is fair, but it argues for more control, not less. If the synthetic pages sit in their own routed index, the router becomes a second variable. Any gain could mean the pages helped, or it could mean the router chose differently because a new index existed. That is exactly the gap between proof that something ran and proof that it worked. So the synthetic pages go *into* Index B as ordinary documents, and routing is either turned off or kept identical on both sides.

Here's a falsifiable prediction: put the same pages in a separate routed index in one run and merge them into Index B as ordinary documents in another, and the two setups will produce different accuracy deltas. If they do, any published "synthetic pages helped" result that doesn't say how the pages were indexed has measured the router, not the pages.

The question set is frozen before either run. I don't swap questions after seeing early results, and I don't drop the ones that "seemed unfair." Every question gets scored the same way: is the final answer correct against the known answer? Retrieving a relevant-looking chunk earns no credit.

Cost isn't an excuse to skip the control. One pipeline written up in *Towards Data Science* cost about $0.0121 per question with GPT-5 and no caching, which comes to roughly $2.42 for 200 questions. With GPT-5-mini it was $0.0041 per question, or $0.82 for 200. Running both indexes doubles that, which is still pocket change next to the cost of shipping synthetic content that makes answers worse.

## What to Measure: Correct Answers, Not Retrieval Activity

"Did retrieval work?" is really three questions. Each one can fail on its own, so I score each one separately.

**Stage 1: Was the right document retrieved?** For each of the 200 questions, I check whether the document containing the known answer shows up anywhere in the candidate set. This is a recall check. It tells me the evidence was reachable, and nothing more.

**Stage 2: Was the right document ranked high?** Hit@1 and similar metrics measure this. The document can be in the pool but sit at position 30, behind four synthetic pages that paraphrase it loosely.

**Stage 3: Was the final answer correct?** I compare the generated answer to the answer I wrote down before the run. This is the only stage the thesis cares about. The first two are diagnostics that explain *why* stage 3 moved.

Here's why I won't let stage 2 stand in for stage 3. In *Retriever-Free Retrieval: Navigating Documents with Decision Models*, the authors fixed a pool of 200 candidates and changed only how that pool was split into requests. Hit@1 rose from 47.8% when each candidate was scored separately to 54.3% with 50 candidates per request, and larger batches added little. That is what the paper reports: with the pool held fixed, request size alone moved Hit@1 by 6.5 points.

What follows is my own inference, not a claim the paper makes. If a harness setting like batch size can move a ranking metric that much, then my comparison has to hold those settings constant. If Index A and Index B differed in batching or reranker configuration, I couldn't tell whether a Hit@1 gain came from the synthetic pages or from the setup. So both indexes run through an identical harness, and I treat any stage-2 difference as unexplained until that holds.

The pattern is old. A proxy metric is easy to measure, so people start optimizing it, and it drifts away from the outcome it was meant to track. Ranking metrics are proxies for answer quality. Proxies move when the measuring setup moves.

What's new here is that synthetic pages act directly on the proxy. They're written to resemble the questions people will ask, so they're built to rank well. That leads to a prediction I can test: **on Index B, stage-2 metrics will improve more than stage-3 correctness does.** If synthetic pages raise Hit@1 while correct answers stay flat or fall, they're winning rank without adding evidence. In that case the pages are noise that happens to rank well, not knowledge.

So the scorecard reports all three stages for both indexes, side by side. The headline number is correct answers. If I can't explain a change in the other two stages, I don't count it as a result.

## The Run Is Cheap Enough That Skipping It Makes No Sense

There's one published reference point for what this costs. In *How to Build an Over-Engineered Retrieval System* (Towards Data Science), the author reports that one run of their pipeline without caching cost about $0.0121 with GPT-5. That works out to roughly $2.42 for 200 questions. With GPT-5-mini as the main model, a run cost about $0.0041, or roughly $0.82 for 200.

These aren't the costs of my experiment. That pipeline's architecture differs from mine. If my setup does any routing, I turn it off or keep it identical on both sides, so the router can't take credit for what the pages did. My setup will have different token counts, prompt lengths, and retrieval depth. So I treat the figures as an order of magnitude, not a quote. A pipeline its own author calls over-engineered costs cents per question, and I have no reason to expect a fixed single-route pipeline to cost much more.

A paired test means 200 questions against Index A and the same 200 against Index B, which is 400 runs. Using the published figures, that's about $4.84 with GPT-5 or $1.64 with GPT-5-mini. If the per-question costs came in ten times higher, the full comparison would still be a minor line item.

The real cost lies elsewhere. Shipping synthetic pages without a control means every later answer depends on an unverified change to the corpus. If those pages win rank without adding evidence, which the previous section's prediction tests directly, the damage won't show up as a failure. It will show up as confident, well-ranked, wrong answers, and nobody will be counting them.

This leads to a claim I can test. If the paired run costs pocket change, a team that skips it isn't saving money. It's avoiding a result. That makes skipping the control evidence in itself: it suggests someone expects, or fears, that stage-3 correctness won't move.

One caution I apply to myself: a cheap run is still only a run. A $1.64 receipt shows the pipeline executed 400 times. It doesn't show any answer was correct. Only the scorecard against answers written in advance can show that.

So run it twice, once per index, and score correct answers.

## Objections: Leakage, and What the Earlier Controls Don't Catch

**Routing and question bias are already covered.** Merging the synthetic pages into the main index with routing frozen, and locking questions written by someone who never saw the pages, handle the first two objections. The controls still leave one gap: a fair router and clean questions can still return a page that simply contains the answer.

**Answers leak word for word.** If a known answer appears verbatim in a synthetic page, retrieval turns into lookup and getting the answer right means nothing. The fix is to scan every synthetic page against the answer key before the run, checking for exact and near-exact matches by n-gram overlap, and to score flagged questions separately. Also check whether the original corpus already held the answer. A page that copies an existing fact makes it rank higher without adding anything new. The number that matters is correctness on questions with no leak.

**The controls interact.** Leakage is where routing logs and question provenance pay off a second time. Cross the leak flag with the log of which documents each subquery retrieved. Here is a prediction I'll stand behind: if the pages add real knowledge, the gain will hold on unleaked questions where a synthetic page was actually retrieved. If the gain shrinks to zero once leaked questions and never-retrieved pages are removed, the pages acted as an answer key that ranked well, not as knowledge.

I hold myself to the same rule. I expect synthetic pages to be mostly noise that ranks well. That is a belief I haven't tested yet. Splitting the results this way lets the data prove me wrong where my evidence is thinnest. If it does, I'll report that result as plainly as I would report a win.

## Read the Gap, Then Decide

Run both indexes, score every final answer against the known answer, and compare. There are three possible results.

**More correct answers with the synthetic pages:** keep them. You have outcome data showing they add something.

**No difference:** remove them. They don't help, and they still cost money. You pay to generate them, store them, and re-embed them. They also take up retrieval slots that real documents would otherwise fill. Pages that change nothing are just overhead.

**Fewer correct answers:** they are hurting retrieval. They push the right documents out of the top results while looking relevant, so remove them.

The net gap alone can mislead you. Because the test is paired, look at each question that changed. Count the questions that went from wrong to right, and the ones that went from right to wrong. A net gain of five from 6 fixes and 1 break is a real improvement. A net gain of five from 40 fixes and 35 breaks means the pages scrambled the rankings and the net happened to come out positive. That system will act unpredictably on the next 200 questions. My prediction is that synthetic pages that mostly rank well without adding knowledge will show this second pattern: many changes in both directions and a small net gain. If the changed questions mostly go one way, that prediction is wrong.

Decide the threshold before the run: how large a net gain you need, and how many breaks you will accept. Setting it afterward lets the result shape the rule.

Better retrieval scores don't settle this. Neither do more documents found or a higher Hit@1. The question is settled only by answers checked against known answers, on the same questions, across both indexes.
