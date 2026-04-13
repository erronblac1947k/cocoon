# Developer Guide — Using COCOON for AI Inference

This guide explains how to integrate COCOON's secure, verifiable AI inference into your application.

## Overview

COCOON exposes an OpenAI-compatible HTTP API through its proxy layer. Your application sends inference requests to a COCOON proxy, which forwards them to a TEE-protected worker. All communication is end-to-end encrypted via RA-TLS, and payments are settled automatically on the TON blockchain.

**Key properties from your application's perspective:**

- **Privacy**: Prompts and responses are visible only to your client. The proxy and worker operators cannot read them.
- **Verifiability**: Each response comes from a cryptographically verified model running in a TEE.
- **Decentralized payments**: You pay per token through a TON smart contract — no subscriptions or API keys from a central provider.

## Architecture

```
Your App (Client)
      ↓  RA-TLS (verified connection)
   Proxy (TEE)
      ↓  RA-TLS (verified connection)
   Worker (TEE + GPU)
      ↓
   AI Model (e.g., Qwen/Qwen3-8B)
```

The COCOON client library handles RA-TLS verification, TON payments, and request routing automatically.

## Prerequisites

- A TON wallet with sufficient balance for inference payments
- A TON network config file with reliable liteservers (the COCOON team publishes one at `https://cocoon.org/resources/mainnet.cocoon.global.config.json`)
- The COCOON root contract address (included in the worker distribution's `worker.conf.example`)

## Quick Start — Local Test

The fastest way to try the full stack locally (no TEE hardware or real TON required):

```bash
# Clone and build
git clone --recursive https://github.com/TelegramMessenger/cocoon.git
cd cocoon

# Start all components locally (proxy + worker + client)
./scripts/cocoon-launch --local-all

# In another terminal: run a benchmark to verify it's working
cd benchmark
./run-benchmark.sh -c 1 -n 10
```

This runs with a fake-TON blockchain, so no real funds are needed. The benchmark exercises the full network and protocol layer.

## Use Cases

### Use Case 1: Connect as a Client (Test Mode)

Run a COCOON client that connects to a local proxy and worker with fake TON:

```bash
# Terminal 1: Start proxy (locally or in TDX VM)
./scripts/cocoon-launch --test --fake-ton scripts/proxy.conf

# Terminal 2: Start worker (locally or in TDX VM)
./scripts/cocoon-launch --test --fake-ton scripts/worker.conf

# Terminal 3: Start client
./scripts/cocoon-launch --test --fake-ton scripts/client.conf
```

The client configuration (`scripts/client.conf`) specifies:
- `type = client`
- `model` — the model you want to query
- `owner_address` — your TON wallet that pays for requests

### Use Case 2: Connect to the Production Network

To connect to the live COCOON network with real TON payments:

1. **Obtain a TON config** — Download from `https://cocoon.org/resources/mainnet.cocoon.global.config.json` or use your own.

2. **Create a client config** (`client.conf`):

   ```ini
   [node]
   type = client
   model = Qwen/Qwen3-8B@<commit>:<hash>
   owner_address = <your_TON_wallet_address>
   ton_config = /path/to/mainnet-config.json
   root_contract_address = <cocoon_root_contract_address>
   ```

   The correct `root_contract_address` and a verified `model` string are published in the worker distribution's `worker.conf.example`.

3. **Fund your wallet** — Deposit TON into `owner_address`. Check your balance and usage via the HTTP stats endpoint (see [Monitoring](#monitoring)).

4. **Run the client:**

   ```bash
   ./scripts/cocoon-launch client.conf
   ```

### Use Case 3: Embed in a Backend Service

The COCOON client is designed to run as a backend service. For example, a Telegram backend can run multiple client instances to handle concurrent user requests.

```bash
# Instance 0
./scripts/cocoon-launch --instance 0 client.conf &

# Instance 1
./scripts/cocoon-launch --instance 1 client.conf &
```

Each instance gets a unique port (10000, 10010, 10020, …) for HTTP stats.

## Client Configuration Options

| Option | Required | Description |
|--------|----------|-------------|
| `type` | Yes | Always `client` for clients |
| `model` | Yes | AI model identifier (e.g., `Qwen/Qwen3-8B@commit:hash`) |
| `owner_address` | Yes | Your TON wallet address (source of payment) |
| `ton_config` | Yes (production) | Path to TON network config JSON |
| `root_contract_address` | Yes (production) | COCOON root contract address on TON |
| `instance` | No | Instance number when running multiple clients (default: `0`) |

## Monitoring

The client exposes HTTP stats on port **10000** (add `instance * 10` for other instances):

```bash
# Human-readable status
curl http://localhost:10000/stats

# JSON-formatted stats (useful for automation)
curl http://localhost:10000/jsonstats

# Performance metrics
curl http://localhost:10000/perf
```

Useful stats to check:
- Wallet balance and amount charged so far
- Number of requests sent and completed
- Current proxy connection status

## Payment and Staking

COCOON uses a client contract per proxy for payments. Key points:

- **Top up**: Send TON to your `owner_address` wallet. The client contract will be funded automatically.
- **Stake**: A minimum stake (see [Smart Contracts](smart-contracts.md)) is locked per proxy connection to cover in-flight requests. It is returned when you close the contract.
- **Charging**: You are charged per token. The proxy commits usage to the blockchain periodically.
- **Closing**: To withdraw your remaining balance, close the client contract. Under normal conditions the proxy grants this immediately. If the proxy is unreachable, the balance above the stake is returned right away; the stake is returned after a delay (approximately one day).

See [Smart Contracts](smart-contracts.md) for the full payment flow.

## Security Verification

When connecting to a COCOON proxy, the client automatically:

1. Verifies the proxy's TDX attestation via RA-TLS
2. Checks the proxy's image hash against the allowed list in the root contract
3. Establishes an encrypted channel — only your client can read prompts and responses

You can verify the exact code running on workers and proxies by reproducing the build yourself. See [Reproducible Build](../README.md#reproducible-build) in the main README.

## Next Steps

- **Architecture**: [Architecture](architecture.md) — Full system overview and request flow
- **Smart Contracts**: [Smart Contracts](smart-contracts.md) — Payment system details
- **RA-TLS**: [RA-TLS](ra-tls.md) — How attestation and secure channels work
- **Deployment**: [Deployment](deployment.md) — All deployment scenarios with step-by-step instructions
