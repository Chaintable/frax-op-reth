# Chaintable write node

> Fork of [fraxfinance/fraxtal-op-reth](https://github.com/fraxfinance/fraxtal-op-reth) (an OP-Stack reth fork), with Chaintable pipeline patches.

## Architecture

This repo runs the chain's execution layer with the [Chaintable pipeline](https://github.com/Chaintable/pipeline) tracer embedded. The tracer extracts block data — block headers, transactions, call traces, receipts, events, and state diffs — and ships it to **S3 + Kafka** (see pipeline's [architecture](https://github.com/Chaintable/pipeline/blob/main/docs/architecture.md)). Two consumption paths:

- **Block headers + state diffs** → Kafka + S3 → [leafage-evm](https://github.com/Chaintable/leafage-evm): a lightweight EVM executor serving state queries (`eth_call`, `eth_estimateGas`, …), no P2P sync, no tx storage (see its [architecture](https://github.com/Chaintable/leafage-evm#architecture)).
- **Block files** (transactions · call traces · receipts · events) → S3 → Chaintable's transaction/trace indexing pipeline.

```
Chaintable write node (this repo · producer, embeds pipeline tracer)
        │
        ├─ block headers + state diffs ──────────────────→ Kafka + S3 ─→ leafage-evm (EVM state queries)
        │
        └─ block files (tx · trace · receipts · events) ──→ S3 ─→ Chaintable indexing pipeline (tx/trace data)
```

---

## Build

```bash
git clone https://github.com/Chaintable/frax-op-reth
cd frax-op-reth
cargo build --release --bin fraxtal-op-reth
```

CI publishes multi-arch images to public ECR `public.ecr.aws/b2h7a5c4/chaintable/fraxtal-writer`. See upstream [fraxfinance/fraxtal-op-reth](https://github.com/fraxfinance/fraxtal-op-reth) for chain-specific details.
