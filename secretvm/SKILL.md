---
name: secretvm
description: "Create and manage confidential SecretVMs on secretai.scrtlabs.com using secretvm-cli. Use when: create VM, launch confidential compute, deploy workload to SecretVM, manage SecretVMs, start confidential VM, secretvm."
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
risk: unknown
source: personal
---

# SecretVM — Confidential Virtual Machines

Create and manage confidential VMs on [secretai.scrtlabs.com](https://secretai.scrtlabs.com) using `secretvm-cli`. Supports the full VM lifecycle: create, start, stop, monitor, verify, and remove.

All commands use API key auth (`-k <key>`) and output JSON (no `-i` flag). Always pass `-s` to enable TLS on VM creation.

Pass `-k <API_KEY>` on **every command**. All examples below use `<API_KEY>` as placeholder.

**Note on identifiers:** `vm list` returns both a numeric `id` and a `uuid` for each VM. Most commands (`start`, `stop`, `logs`, `edit`, `remove`, `attestation`) use the numeric **ID**. The `vm status` command uses the **UUID**.

## Prerequisites & Setup

### 1. Check if secretvm-cli is installed

Run: `secretvm-cli --version`

If not found, install it:

```bash
npm install -g secretvm-cli
```

Requires Node.js >= 16.

### 2. Get an API key

If the user does not have an account, direct them to sign up at:
**https://secretai.scrtlabs.com**

Once they have an account, they need an API key. Ask the user to provide it.

### 3. Validate the API key

```bash
secretvm-cli -k <API_KEY> status
```

Expected JSON output on success:
```json
{ "status": "success", "result": { ... } }
```

If it fails, the key is invalid or the account has issues. Ask the user to verify their key.

## Creating a VM

### Step 1: Choose a template or provide a docker-compose

List available templates:

```bash
secretvm-cli -k <API_KEY> vm templates
```

- If a template fits, use `-T <template_id_or_name>` during creation.
- Otherwise, the user provides a path to their `docker-compose.yaml` via `-d <path>`.

### Step 2: Choose VM size

| Size | Use case |
|------|----------|
| `small` | Light workloads, testing |
| `medium` | Standard workloads |
| `large` | Heavy compute |

### Step 3: Create the VM

```bash
secretvm-cli -k <API_KEY> vm create \
  -n <vm_name> \
  -t <small|medium|large> \
  -s \
  -T <template_id>
```

Or with a custom docker-compose:

```bash
secretvm-cli -k <API_KEY> vm create \
  -n <vm_name> \
  -t <small|medium|large> \
  -s \
  -d <path/to/docker-compose.yaml>
```

**Note:** `-s` (TLS) must always be included. It auto-injects a Traefik reverse proxy with Let's Encrypt.

### Optional flags

| Flag | Description |
|------|-------------|
| `-f <sev\|tdx>` | Platform: AMD SEV-SNP or Intel TDX (default: `tdx`) |
| `-e <path>` | Path to `.env` file with secrets |
| `-p` | Enable filesystem persistence |
| `-a` | Enable private mode |
| `-u` | Enable upgradeability with state preservation |
| `-m <domain>` | Custom domain (sets `skip_launch`) |
| `-l <user:pass>` | Private Docker registry credentials (encrypted client-side) |
| `-r <url>` | Docker registry URL (default: docker.io) |
| `-A <path>` | `.tar` archive with additional files |
| `-K <type>` | KMS provider: `secret`, `dstack`, or `google` |
| `-E dev` | Dev environment (includes SSH access) |
| `--eip8004-registration-json <path>` | EIP-8004 registration JSON file |
| `--eip8004-chain <chain>` | EIP-8004 chain (currently only `base-mainnet`) |

### Step 4: Wait for VM to be ready

After creation, parse the JSON output for the VM UUID, then poll status:

```bash
secretvm-cli -k <API_KEY> vm status <vm_uuid>
```

Poll every 10 seconds, up to 5 minutes. If the status is `running`, report the VM's IP address and domain to the user. If the status indicates failure or the timeout is reached, report the error.

## Managing VMs

### List all VMs

```bash
secretvm-cli -k <API_KEY> vm list
```

Returns JSON array with: ID, UUID, Name, Status, Type, PricePerHour, IP, Domain, Created At.

### Check VM status

```bash
secretvm-cli -k <API_KEY> vm status <vm_uuid>
```

### Start a stopped VM

```bash
secretvm-cli -k <API_KEY> vm start <vm_id>
```

### Stop a running VM

```bash
secretvm-cli -k <API_KEY> vm stop <vm_id>
```

### View Docker logs

```bash
secretvm-cli -k <API_KEY> vm logs <vm_id>
```

Use this to debug issues with the running workload.

### Edit a VM

```bash
secretvm-cli -k <API_KEY> vm edit <vm_id> \
  -n <new_name> \
  -d <new_docker_compose_path> \
  -e <new_env_path>
```

Supports flags: `-n`, `-d`, `-e`, `-p`, `-l`, `-r`. Note: editing replaces old env vars and docker credentials.

### Get CPU attestation

```bash
secretvm-cli -k <API_KEY> vm attestation <vm_id>
```

Returns CPU attestation data for the VM's TEE.

### Remove a VM

> **WARNING: This is irreversible. Always confirm with the user before executing.**

```bash
secretvm-cli -k <API_KEY> vm remove <vm_id>
```

Permanently deletes the VM and all its data.

## Verification

### Verify attestation quote

Verify a VM's attestation quote directly:

```bash
secretvm-cli -k <API_KEY> verify quote --vm-id <vm_id>
```

Or verify from a file or hex string:

```bash
secretvm-cli -k <API_KEY> verify quote --quote-file <path>
secretvm-cli -k <API_KEY> verify quote --quote <hex_string>
```

Optional: `--attestation-type <auto|tdx|sev>`, `--environment <production|preview>`, `--base-url <url>`.

### Verify workload

Confirm that the running workload matches the expected docker-compose:

```bash
secretvm-cli -k <API_KEY> verify workload --vm-id <vm_id> -d <docker-compose.yaml>
```

Optional: `--docker-files <path>` or `--docker-files-sha256 <sha>` for file hash verification.

### Verify Proof of Cloud

Verify Proof of Cloud from a TDX quote:

```bash
secretvm-cli -k <API_KEY> verify proof-of-cloud --vm-id <vm_id>
```

Also aliased as `verify poc`.

## EIP-8004 Registration

To register a VM with EIP-8004, include these flags during `vm create`:

```bash
secretvm-cli -k <API_KEY> vm create \
  -n <vm_name> \
  -t <size> \
  -s \
  -d <docker-compose.yaml> \
  --eip8004-registration-json <path/to/registration.json> \
  --eip8004-chain base-mainnet
```

- `--eip8004-registration-json` — path to the EIP-8004 registration JSON file
- `--eip8004-chain` — currently only `base-mainnet` is supported

These flags are only used during VM creation.

## Quick Reference

| Task | Command |
|------|---------|
| Install CLI | `npm install -g secretvm-cli` |
| Check auth | `secretvm-cli -k <KEY> status` |
| List templates | `secretvm-cli -k <KEY> vm templates` |
| Create VM (template) | `secretvm-cli -k <KEY> vm create -n <NAME> -t <SIZE> -s -T <TEMPLATE>` |
| Create VM (compose) | `secretvm-cli -k <KEY> vm create -n <NAME> -t <SIZE> -s -d <COMPOSE>` |
| List VMs | `secretvm-cli -k <KEY> vm list` |
| VM status | `secretvm-cli -k <KEY> vm status <UUID>` |
| Start VM | `secretvm-cli -k <KEY> vm start <ID>` |
| Stop VM | `secretvm-cli -k <KEY> vm stop <ID>` |
| VM logs | `secretvm-cli -k <KEY> vm logs <ID>` |
| Edit VM | `secretvm-cli -k <KEY> vm edit <ID> [options]` |
| Remove VM | `secretvm-cli -k <KEY> vm remove <ID>` (**confirm first!**) |
| Attestation | `secretvm-cli -k <KEY> vm attestation <ID>` |
| Verify quote | `secretvm-cli -k <KEY> verify quote --vm-id <ID>` |
| Verify workload | `secretvm-cli -k <KEY> verify workload --vm-id <ID> -d <COMPOSE>` |
| Verify PoC | `secretvm-cli -k <KEY> verify proof-of-cloud --vm-id <ID>` |
