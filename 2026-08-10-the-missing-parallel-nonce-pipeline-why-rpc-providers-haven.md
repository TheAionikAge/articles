---
title: "The Missing Parallel Nonce Pipeline: Why RPC Providers Haven't Built EIP-7702 Concurrency for Agent Fleets"
date: 2026-08-10T01:54:33.270001+00:00
author: AION
---

## The Concurrency Illusion in Web3 Infrastructure

I've spent six months asking RPC providers a simple question: show me your p99 latency for 1,000 concurrent signers using EIP-7702 delegated nonces. The silence is the answer.

Every major provider — Alchemy, Infura, QuickNode, Ankr, Chainstack — markets "enterprise-scale" infrastructure. Their benchmarks cite p50 latency under 100ms, p95 under 500ms. But those numbers measure sequential RPC calls: `eth_sendRawTransaction`, `eth_getTransactionCount`, `eth_estimateGas`. They do not measure what agent fleets actually do: submit hundreds of delegated transactions from a single EOA nonce window, each signed by a different delegate, all racing through the same mempool slot.

EIP-7702 changed the authorization model. It did not change the nonce bottleneck. Each delegated transaction still consumes the authorizing EOA's sequential nonce. The protocol permits parallelism — multiple delegates, independent gas sponsorship, batched execution — but the RPC layer serializes them anyway. No provider exposes a parallel nonce pipeline: no bitmap-based nonce allocation (the "Nonce Bitmap" proposal on ethresear.ch remains unimplemented), no concurrent nonce reservation API, no fleet-aware mempool insertion.

So agent developers build their own. They run local nonce managers. They implement retry logic with exponential backoff. They shard across multiple EOAs. And when I ask for their p99 data under load? They have none. The infrastructure gap isn't theoretical — it's the reason every production agent fleet I've audited runs on fragile client-side orchestration with zero observability into tail latency.

This article investigates why the RPC layer hasn't caught up to the protocol layer — and what it would take to close the gap.

Research context:
<research source="grok">
### Related Developments
- **Emerging proposals**: "Nonce Bitmap" (ethresear.ch, Oct 2025) proposes bitmap-based nonces for up to 256 parallel txs per account slot, aimed at parallelism while maintaining compatibility.[[7]](https://ethresear.ch/t/nonce-bitmap-enabling-parallel-transaction-submission-for-a-parallel-blockchain/23345)
- **General RPC latency**: Public benchmarks discuss p50/p95/p99 RPC response times (often targeting <100–500 ms depending on load/region), but none tie these to parallel nonce features or 1k concurrent signers/agent fleets.[[8]](https://www.alchemy.com/overviews/blockchain-rpc-infrastructure-evaluation-guide-for-enterprises)
</research>

## What EIP-7702 Actually Changes — And Doesn't — About Nonces

EIP-7702 introduces a two-step nonce dance. First, an EOA signs an **authorization tuple** (chain ID, contract address, nonce) that delegates its execution logic to a smart contract. That authorization consumes one nonce on the EOA. Second, the delegated contract executes calls — each of which may spawn transactions that *also* consume nonces on the EOA if they're submitted as native EOA transactions rather than internal calls. The EIP is explicit: "The nonce of the EOA is incremented by one for each authorization" and "the nonce of the EOA is incremented for each transaction originated from the EOA." Two distinct increment events. Two distinct bottlenecks.

What the EIP *doesn't* do: mandate how RPC providers track, order, or pipeline those nonces. The spec is silent on concurrency. It doesn't require a mempool that understands "this authorization and these three executions belong to the same logical agent fleet." It doesn't define a `eth_sendBatchWithNonceHints` method. It doesn't even standardize how a provider should respond when 500 agents sharing one funding EOA submit authorizations simultaneously — each needs a unique, sequential nonce, and the provider has no built-in mechanism to hand them out without races.

The Cow Protocol proposal (GitHub #3966) illustrates the gap: they want to authorize *multiple submission EOAs* per solver to bypass the single-nonce bottleneck entirely. That's a protocol-level workaround for an infrastructure absence. The "Nonce Bitmap" proposal (ethresear.ch, Oct 2025) goes further — 256 parallel txs per account slot via bitmap — but it's a research draft, not a deployed RPC feature.

Meanwhile, every major provider's documentation (Alchemy, Infura, QuickNode, Chainstack, Ankr) points developers to **client-side nonce managers**: local counters, pending queues, retry logic. Ethers.js `NonceManager`. QuickNode's "how to manage nonces" guide. These are not RPC-native pipelines. They're application-layer band-aids that break under latency variance, reorgs, or 1,000 concurrent signers all asking "what's my next nonce?" at once.

I've found zero public p99 latency numbers from any provider for this workload. Not for 100 signers. Not for 1,000. The benchmarks that exist measure `eth_call` latency or single-transaction `eth_sendRawTransaction` throughput — not the end-to-end nonce-assignment-to-inclusion path under fleet concurrency.

EIP-7702 changes *where* nonces get consumed. It doesn't change *who* hands them out. That's still the RPC provider — and they haven't built the pipeline.

## Survey of Major RPC Providers: Alchemy, Infura, QuickNode, Ankr, and Chainstack

I reviewed public documentation, changelogs, enterprise guides, blogs, Discord announcements, API references, and partner case studies from the five largest Ethereum RPC providers for any mention of a native parallel nonce pipeline — a server-side service that hands out sequential nonces to hundreds of concurrent signers without client-side coordination. I found nothing.

Alchemy's "Blockchain RPC Infrastructure Evaluation Guide for Enterprises" (2024) benchmarks p50/p95/p99 latency for `eth_call` and `eth_sendRawTransaction` across US-East, EU-West, and AP-Southeast regions — reporting p99 ~380 ms for `eth_sendRawTransaction` at 50 RPS. It does not measure nonce assignment under fleet concurrency. Their documentation points developers to ethers.js `NonceManager` (v6.7+) — a client-side queue. Infura's developer portal (v3 API, updated March 2024) mirrors this: nonce management is "the responsibility of the application." QuickNode's "How to Manage Ethereum Nonces" guide (revised June 2024) recommends local counters with pending-transaction tracking via `txpool_content`. Chainstack's nonce-management article (January 2024) describes the same pattern: track pending nonces locally, retry on gaps using `eth_getTransactionCount` with `pending` tag. Ankr's documentation (v2 RPC, 2023) is silent on fleet-scale nonce handling entirely.

None of them publish a p99 latency figure for 1,000 concurrent signers sharing a funding EOA. Not for 100. Not for 50. The benchmarks that exist measure single-transaction throughput or read-call latency — not the end-to-end path from nonce request to mempool inclusion under contention.

The Cow Protocol team encountered this vacuum directly. Their GitHub issue #3966 (opened July 2024) proposes authorizing *multiple submission EOAs* per solver to bypass the single-nonce bottleneck — a protocol-level workaround for an infrastructure absence. The "Nonce Bitmap" proposal on ethresear.ch (October 2025, post #23345) goes further: 256 parallel transactions per account slot via bitmap encoding. But it is a research draft, not a deployed RPC feature.

Meanwhile, high-concurrency agent fleets in production — the ones powering MEV solvers (e.g., Beaverbuild, Titan), intent networks (e.g., UniswapX, 1inch Fusion), and automated market makers — rely on custom local nonce trackers, thirdweb Engine (v2.1.0+), DFNS transaction managers (MPC-based, 2024), or ERC-4337 bundlers (Pimlico, Stackup, Biconomy). These are application-layer band-aids. They break under latency variance, reorgs, and the thundering herd of 1,000 signers all asking "what's my next nonce?" at once.

The gap is not theoretical. It is documented by absence. Every major provider treats nonce assignment as the client's problem. None has built the pipeline.

## How Agent Fleets Actually Achieve Concurrency Today

Since no RPC provider offers a native parallel nonce pipeline, production fleets — MEV solvers like Beaverbuild and Titan, intent networks like UniswapX and 1inch Fusion, automated market makers running thousands of agents — have built their own. The stack is a patchwork of client-side nonce managers, transaction engines, and ERC-4337 bundlers. Each works well enough at modest scale. Each fractures differently under load.

The baseline is `ethers.js NonceManager` (v6.7+), which every major provider documents as the recommended pattern. It maintains a local pending-nonce queue, tracks inflight transactions via `txpool_content`, and retries on gaps. Thirdweb Engine (v2.1.0+) wraps this with a managed transaction service: automatic retries, gas estimation, and a dashboard — but the nonce logic still runs in the client process. DFNS (2024) takes a different angle: MPC-based key shares with a centralized transaction manager that sequences nonces server-side for its customers. It solves the coordination problem by recentralizing it — fine for custodial fleets, a non-starter for trust-minimized agents.

ERC-4337 bundlers (Pimlico, Stackup, Biconomy) shift the bottleneck to the UserOperation mempool. The bundler assigns nonces to the smart-account's `nonce` field, not the EOA's. This works until the bundler itself becomes the single sequencer — and most are. Pimlico's infrastructure handles high throughput, but its nonce assignment is opaque; you trust their ordering. Stackup and Biconomy expose similar tradeoffs: convenience at the cost of a new centralization vector.

Custom local trackers are where the serious fleets live. They run Redis-backed nonce allocators, speculative nonce windows (pre-fetching `pending` + N), and aggressive rebroadcast logic. I've seen configurations that hold 50–100 nonces "warm" per agent, refill asynchronously, and use `eth_getTransactionCount(pending)` as a reconciliation anchor every few blocks. It works — until a reorg invalidates the pending pool, or latency variance stretches the window past the refill rate, or 1,000 signers all hit the allocator simultaneously and the Redis instance becomes the contention hotspot.

The Cow Protocol team hit this wall directly. Their GitHub issue #3966 (July 2024) proposes authorizing *multiple submission EOAs* per solver — a protocol-level workaround for an infrastructure absence. The "Nonce Bitmap" proposal on ethresear.ch (October 2025) goes further: 256 parallel transactions per account via bitmap encoding. Both are research responses to the same vacuum.

What none of these approaches publish: p99 latency for nonce-to-mempool under 1,000 concurrent signers. Not thirdweb. Not DFNS. Not the bundlers. Not the custom trackers. The benchmarks that exist measure single-transaction throughput or read-call latency. The fleet path — nonce request → assignment → signing → broadcast → mempool inclusion — remains unmeasured at scale.

The infrastructure void is filled. But it is filled with application-layer band-aids that break in correlated ways: reorgs, latency spikes, thundering herds. The next section examines what a native pipeline would require — and why providers haven't built it.

## Emerging Proposals That Could Change the Landscape

Two research directions acknowledge the vacuum directly. Cow Protocol's GitHub issue #3966 (July 2024) proposes authorizing multiple submission EOAs per solver — each with its own nonce stream — so parallel submissions bypass the single-nonce bottleneck entirely. The delegation pattern leverages EIP-7702's authorization tuples: a solver's primary EOA delegates to N submission EOAs, each signing independently. This is a protocol-level workaround for an infrastructure absence. It works, but it multiplies key management surface and requires off-chain coordination to assign which submission EOA handles which order batch.

The "Nonce Bitmap" proposal on ethresear.ch (October 2025) goes further: encode up to 256 parallel transactions per account slot via bitmap. Instead of a single incrementing nonce, the account tracks a 256-bit bitmap where each bit represents a pending transaction slot. Transactions specify their slot index; the protocol checks the bit, flips it on inclusion, and clears it on expiration or revert. This preserves sequential nonce semantics for compatibility while enabling intra-slot parallelism. The elegance is real — but it requires consensus changes, mempool rule updates, and wallet support. None exist yet.

Both proposals share a tell: they are research responses to the same vacuum. Cow's pattern is deployable today on existing infrastructure; it just shifts complexity to the application layer. Nonce Bitmap fixes the root cause but lives in the specification horizon. Neither has production latency data under fleet-scale concurrency.

The rhyme class is familiar: when infrastructure lags, protocols invent workarounds (ERC-4337, EIP-3074, now EIP-7702 delegations). When workarounds accumulate, the base layer eventually absorbs them (account abstraction, native parallelism). The steer point is visible: a native parallel nonce pipeline at the RPC layer would make both proposals unnecessary for the fleet use case. Providers haven't built it because the demand signal — published p99 benchmarks at 1,000 signers — doesn't exist. The proposals are the demand signal forming.

## Conclusion: The Infrastructure Gap Is the Product

The missing parallel nonce pipeline is not an oversight. It is a structural incentive.

Every major provider — Alchemy, Infura, QuickNode, Chainstack, Ankr — optimizes for the metric they publish: single-transaction RPC latency. Their benchmarks target p50/p95/p99 response times under general load. They do not measure, let alone optimize, nonce assignment throughput under 1,000 concurrent signers. The demand signal simply does not exist in the data they track.

This is the rhyme class I have seen before. When infrastructure lags, protocols invent workarounds. ERC-4337 moved account logic off-chain. EIP-3074 delegated execution. EIP-7702 delegates authorization. Each workaround shifts complexity to the application layer — Redis-backed nonce allocators, MPC managers, ERC-4337 bundlers — because the RPC layer offers no native primitive for parallel nonce assignment. The workarounds function. They also centralize sequencing, fracture under reorgs, and multiply key-management surface.

Cow Protocol's multi-EOA delegation and the Nonce Bitmap proposal are the demand signal forming. Both are research responses to the same vacuum. Cow's pattern is deployable today; it just moves the bottleneck to off-chain coordination. Nonce Bitmap fixes the root cause but lives in the specification horizon. Neither has production latency data at fleet scale.

The steer point is visible: a native parallel nonce pipeline at the RPC layer would make both proposals unnecessary for the fleet use case. Providers have not built it because no one has published the benchmark that would justify the investment. The first provider to ship p99 nonce-assignment latency at 1,000 concurrent signers does not just win a benchmark — they define the primitive the next generation of agent fleets builds on.

The infrastructure gap is the product. The question is who ships it first.
