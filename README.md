# Confidential Compute Open Network (COCOON) – Decentralized AI Inference on TON

COCOON enables running AI models in trusted execution environments, while earning TON cryptocurrency for compute services.

- GPU owners earn TON by serving models
- App developers plug into low-cost, secure and verifiable AI compute
- Users enjoy AI seamlessly, with full privacy and confidentiality

This repository contains all the necessary tools and documentation to both serve and access models via COCOON.

## What You Can Do with COCOON

### 🖥️ Run a Worker — Earn TON with Your GPU

If you have a server with an NVIDIA H100+ GPU and Intel TDX support, you can run a COCOON worker and earn TON cryptocurrency by serving AI inference requests.

- **Download**: [Latest worker release](https://ci.cocoon.org/cocoon-worker-release-latest.tar.xz) – Ready-to-run TDX image and setup scripts
- **Setup Guide**: Full instructions in the release archive ([preview here](scripts/dist-worker/README.md))

### 🤖 Build Apps — Access Secure AI Inference

If you are a developer who wants to use COCOON's verifiable, privacy-preserving AI inference in your application:

- Send inference requests through the COCOON client via any proxy endpoint
- Payments are handled automatically through TON smart contracts
- All prompts and responses remain private (only visible to your client)
- You can cryptographically verify that the correct model processed your request
- **Developer Guide**: See [docs/developer-guide.md](docs/developer-guide.md) for integration instructions

### 🔍 Verify — Reproduce Builds and Audit the System

Anyone can verify that the published worker images match the open-source code:

- **Reproducible builds**: Rebuild the worker image from source and compare hashes (see below)
- **Smart contracts**: The on-chain registry is public and auditable
- **Technical documentation**: See [docs/](docs/) for architecture, security model, and more

### 🛠️ Contribute — Extend the Platform

The full source code is available for developers who want to run proxies, extend the protocol, or contribute:

- Build and run all components locally with `./scripts/cocoon-launch --local-all`
- Explore the [technical docs](docs/) for architecture details
- See [Deployment guide](docs/deployment.md) for use cases from local testing to production

---

## Reproducible Build

Anyone can verify the worker distribution by rebuilding from source. Note that this step is not needed to run your own workers.

```bash
# 1. Build the VM image (reproducible)
./scripts/build-image prod

# 2. Generate distribution
./scripts/prepare-worker-dist ../cocoon-worker-dist

# 3. Verify the TDX image matches the published release
cd ../cocoon-worker-dist
sha256sum images/prod/{OVMF.fd,image.vmlinuz,image.initrd,image.cmdline}
# Compare with the published checksums
```

The same goes for model images:

```bash
# 1. This will generate a model tar file with the full model name, which includes hash and commit.
./scripts/build-model Qwen/Qwen3-0.6B
# Compare with the published model name
```

## License

See LICENSE file.