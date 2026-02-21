# The Skills Layer for AI Agents

Packaged capabilities. Signed provenance. One registry.

`MCP` · `Skills` · `Sigstore` · `Enact Registry`

---

## 01 — The shift

### Agents are the new interface

| Before | Now |
|--------|-----|
| Click through UI | Agent calls `tool()` |
| Login → Dashboard → Button → Result | Prompt → Agent → Tool → Done |

Software used to ship as UIs. Now it ships as **capabilities agents can call**. The question: how do agents find, trust, and run those capabilities?

---

## 02 — Anatomy of an agent

### Every agent has three pillars

**Context — Knowledge & Memory**
- AGENTS.md / CLAUDE.md
- RAG / embeddings
- Skills (docs + instructions)

**Cognition — Reasoning & Planning**
- Claude / GPT / Gemini
- chain-of-thought
- tool-use, planning

**Action — Tools & Execution**
- MCP (JSON-RPC)
- CLI / bash
- APIs & services

---

## 03 — Skills

### A skill is a folder

```yaml
# SKILL.md
---
name: wallet-lookup
description: Look up on-chain wallet balance
  and transaction history.
metadata:
  author: chaindata
  version: 1.0.0
---

# Wallet Lookup

### Step 1 — Extract the 0x address
### Step 2 — python scripts/lookup.py --address ADDR
### Step 3 — Summarise: balance, tokens, activity
```

```
scripts/     ← code, loaded on demand
references/  ← docs, loaded on demand
```

**Level 1 — Always loaded**
YAML frontmatter. Tells the agent *when* this skill applies.

**Level 2 — Loaded when relevant**
SKILL.md body. Step-by-step instructions, examples, guardrails.

**Level 3 — Fetched on demand**
Scripts, references, assets. Only pulled when the agent needs them.

---

## 04 — Honest benchmark

### AGENTS.md outperforms skills

| Configuration | Pass rate | vs baseline |
|---|---|---|
| Baseline — no docs | 53% | — |
| Skill — default (no trigger instruction) | 53% | +0 pp |
| Skill + explicit trigger instruction | **79%** | **+26 pp** |
| **AGENTS.md — 8 KB compressed docs index** | **100%** | **+47 pp** |

> 56% of runs: skill never invoked

*Vercel Next.js evals, targeting APIs not in training data — Jude Gao, Jan 2026*

So why bother with skills at all?

---

## 05 — The case for skills

### Skills win everywhere else

**Dynamic discovery**
AGENTS.md is static — fixed at session start. Skills can be searched from a registry at runtime.

**Executable code**
AGENTS.md is documentation. A skill can run a migration, call an API, or build a container.

**Packaged & versioned**
One skill = one capability. Named, composable, auditable. A flat file loses all that structure.

**Portable across agents**
AGENTS.md lives in one repo. Skills work with Claude, GPT, Cursor, Copilot — any agent that speaks MCP.

What skills need is infrastructure that **accentuates** these strengths.

---

## 06 — Introducing Enact

### What Enact adds to a skill

A skill is a folder. Enact turns it into something an agent can find, trust, and run.

**Runtime — Package & execute**
Build hooks, isolated containers, secret injection. Skills run safely without the agent managing infrastructure.

**Registry — Search & discover**
Agents find skills by capability, not by name. Semantic search over a central index via MCP.

**Security — Sign & verify**
Sigstore signing, Rekor transparency log, trust tiers. Every skill has a verifiable identity.

---

## 07 — Packaging

### The `skill.package.yml`

```yaml
# skill.package.yml
enact: "2.0.0"
name: chaindata/wallet-lookup
version: "1.2.0"
from: python:3.12-slim

hooks:
  build:
    - pip install -r requirements.txt

env:
  CHAIN_API_KEY:
    secret: true

scripts:
  lookup: "python wallet.py {{address}}"
```

- **SKILL.md** — Instructions written *for the agent*. What to pass, what to expect back.
- **skill.package.yml** — Runtime manifest. Build hooks, secrets, execution backend.
- **code/** — Anything from a script to a full app. Enact builds, signs, and runs it.

---

## 08 — The problem

### Skills exist. Trust doesn't.

**Discovery — Scattered everywhere**
GitHub repos, npm, blog posts. No central index.

**Identity — Who published this?**
No signing. No verification. Could be anything.

**Provenance — Has it been tampered?**
No audit trail. No transparency log.

> "You wouldn't `curl | bash` from a random URL.
> Why would you let your agent do it?"

---

## 09 — The solution

### Enact — signed registry for agent skills

```
Publisher (OIDC identity) → Sigstore (keyless sign) → Rekor (transparency log)
```

1. **Publish** — push a skill. Enact builds and signs it.
2. **Discover** — agents search by capability via MCP.
3. **Verify** — signatures checked, log entry confirmed.
4. **Run** — isolated execution. Secrets injected, never exposed.

**Sigstore — Let's Encrypt for code signing**
Keyless signing via OIDC identity (GitHub, Google). Short-lived certs. Every signature recorded in an immutable transparency log.

`open source` · `Linux Foundation` · `npm, PyPI, K8s`

**Trust tiers:**
- ✦ **Official** — Enact first-party
- ✓ **Verified** — identity confirmed, signing enforced
- ○ **Community** — open, use at own risk

---

## 10 — Under the hood

### What happens when an agent calls a skill

```
$ enact install chaindata/wallet-lookup

✓ Verified Sigstore signature
✓ Checked Rekor transparency log
✓ Ready: chaindata/wallet-lookup@1.2.0

$ enact run chaindata/wallet-lookup:lookup \
    -a '{"address": "0xabc..."}'

✓ Trust policy: enforce
✓ Backend: docker
✓ Secrets injected (CHAIN_API_KEY)
  (agent never saw the value)
```

1. **Fetch** — Manifest + SKILL.md injected into context
2. **Verify** — Cosign checks digest against publisher identity
3. **Isolate** — Container runs with declared permissions only
4. **Inject** — Secrets from keystore → env vars. Never in logs, never in context.

---

## 11 — Live demo

### Let's see it run

```
$ enact search "public records lookup"

  veridian/records-search  ✓ Verified   94k/week
  docuscan/person-search   ✓ Verified   12k/week
  community/record-finder  ○ Community  340/week

$ enact install veridian/records-search

✓ Signature valid  (veridian-data@github.com)
✓ Rekor log: rec_8f3a91bc
✓ Installed

$ enact run veridian/records-search:search \
    -a '{"query": "Jane Doe, Austin TX"}'

  Trust: enforce  ✓   Backend: docker  ✓
  → 3 records found
```

---

## 12 — Get involved

### Build on the skills layer

- **Publish a skill** — Package your API or data source. Reach every agent.
- **Use the registry** — Import verified skills. Stop reinventing capabilities.
- **Contribute** — Skill format and MCP bindings are open. PRs welcome.

> SaaS was software you click through.
> **Skills are software agents act with.**

`npm for agents` · `but signed` · `but verified`

enact.tools · @enactprotocol
