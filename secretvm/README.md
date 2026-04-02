# SecretVM Skill

An [agent skill](https://skills.sh/) for creating and managing **confidential Virtual Machines** on [secretai.scrtlabs.com](https://secretai.scrtlabs.com) using `secretvm-cli`.

## What It Does

This skill gives AI coding agents (Claude Code, Cursor, Windsurf, etc.) the knowledge to:

- **Create** confidential VMs with AMD SEV-SNP or Intel TDX hardware isolation
- **Manage** the full VM lifecycle: start, stop, monitor, edit, remove
- **Deploy** workloads via Docker Compose templates or custom configurations
- **Verify** CPU/GPU attestation and TLS binding using the `secretvm-verify` SDK
- **Register** VMs with EIP-8004 on-chain attestation

## Install

```bash
npx skills add scrtlabs/secretvm-skill
```

## Prerequisites

- [secretvm-cli](https://www.npmjs.com/package/secretvm-cli): `npm install -g secretvm-cli`
- [secretvm-verify](https://www.npmjs.com/package/secretvm-verify): `npm install -g secretvm-verify`
- An API key from [secretai.scrtlabs.com](https://secretai.scrtlabs.com)

## Usage

Once installed, ask your AI agent things like:

- "Create a small SecretVM using the ollama template"
- "List my running VMs"
- "Verify the attestation for my-vm.vm.scrtlabs.com"
- "Deploy this docker-compose.yaml to a confidential VM"
- "Show me the logs for VM 42"

The agent will use `secretvm-cli` commands with your API key to manage VMs on your behalf.

## Supported VM Sizes

| Size | Use Case |
|------|----------|
| `small` | Light workloads, testing |
| `medium` | Standard workloads |
| `large` | Heavy compute |

## Links

- [SecretVM Platform](https://secretai.scrtlabs.com)
- [secretvm-cli on npm](https://www.npmjs.com/package/secretvm-cli)
- [secretvm-verify on npm](https://www.npmjs.com/package/secretvm-verify)
- [Skills.sh](https://skills.sh/)

## License

MIT
