# ArchRails — Architecture Governance as Code

> **Enforce your architecture at pull request time.**  
> ArchRails verifies PRs against your CALM architecture graph — checking node boundaries, allowed connections, and interface contracts deterministically. Every violation is traceable to a CALM node or relationship. No generic "best practices" guesses.

[![FINOS CALM 1.2](https://img.shields.io/badge/FINOS%20CALM-1.2-blue)](https://calm.finos.org)
[![GitHub App](https://img.shields.io/badge/GitHub-App-black)](https://github.com/marketplace)
[![GitLab CI](https://img.shields.io/badge/GitLab-CI-orange)](https://gitlab.com)
[![No code stored](https://img.shields.io/badge/Source%20Code-Never%20Stored-green)]()

---

## What Is ArchRails?

ArchRails is a PR enforcement layer built on top of [FINOS CALM](https://calm.finos.org) — the open standard for Architecture as Code. Where CALM gives you a machine-readable, version-controlled architecture graph, ArchRails puts that graph to work on every pull request.

When a developer opens a PR, ArchRails:

1. Resolves which CALM nodes are touched by the diff
2. Verifies node boundaries, allowed relationships, and interface contracts deterministically against your `calm.json`
3. Posts inline review comments that cite the exact CALM node ID or relationship that was violated
4. Explains each violation in plain English via an LLM explanation layer

The result: machine-verifiable constraint checking *and* human-readable explanations in the same review comment — with zero guesswork.

---

## Why CALM as the Backbone?

Generic AI review bots are trained on open-source patterns. Their comments are confident but untraceable — no source, no context, no connection to your team's actual decisions.

ArchRails uses CALM as its structural backbone because CALM defines nodes, relationships, and interface contracts precisely, enabling **deterministic** structural verification rather than probabilistic inference. The LLM layer then translates those verified violations into readable PR comments.

If a node, relationship, or interface constraint isn't in your `calm.json`, ArchRails won't enforce it. Full stop.

---

## How It Works

### 1. Contact us to get provisioned

[Reach out](https://archrails.io/contact) and we'll provision your organization, connect your GitHub or GitLab integration, and onboard your first repo.

### 2. Commit your CALM document

Add `calm.json` to your repo describing nodes, relationships, and interface contracts — your enforceable architecture source of truth.

```json
{
  "$schema": "https://calm.finos.org/release/1.2/meta/calm.json",
  "nodes": [
    { "unique-id": "service-api-gateway",  "node-type": "service",  "name": "API Gateway" },
    { "unique-id": "service-order",        "node-type": "service",  "name": "Order Service" },
    { "unique-id": "service-payment",      "node-type": "service",  "name": "Payment Service" },
    { "unique-id": "db-orders",            "node-type": "database", "name": "Order Database",
      "interfaces": [{ "unique-id": "iface-orders-sql", "protocol": "JDBC", "port": 5432 }] }
  ],
  "relationships": [
    {
      "unique-id": "rel-connects-order-payment",
      "relationship-type": {
        "connects": {
          "source":      { "node": "service-order",   "interfaces": ["iface-order-rest"] },
          "destination": { "node": "service-payment", "interfaces": ["iface-payment-rest"] }
        }
      }
    }
  ]
}
```

### 3. Add the config mapping

Commit `.archrails/config.yaml` to map source paths to CALM node IDs. ArchRails scopes each review to the exact nodes touched by the diff.

```yaml
version: 1
default_architecture: layered-services

governance:
  provider: calm
  file: docs/architecture/calm.json

mapping:
  - path: src/ApiGateway
    node: service-api-gateway
  - path: src/OrderService
    node: service-order
  - path: src/PaymentService
    node: service-payment
  - path: src/OrderDatabase
    node: db-orders

review:
  on: pull_request

integrations:
  github:
    app: archrails
```

### 4. Get CALM-backed PR feedback

Every PR gets an architecture review citing the exact CALM node, relationship, or interface violated — with a plain-English explanation of why.

**Example PR comment:**

```
🏗️ CALM Architecture Review

⚡ Relationship violation: `OrderService` calls `PaymentService` directly.
   CALM node `rel-connects-order-payment` requires routing via `service-api-gateway`.

⚡ Interface breach: `OrderDatabase` accessed via HTTP.
   Node `db-orders` declares `JDBC:5432` only.

✅ Node boundary respected: `InventoryService` changes scoped to
   `service-inventory` — no cross-node leakage detected.

CALM sources:
  calm.json · rel-connects-order-payment
  calm.json · db-orders · iface-orders-sql
```

---

## Key Guarantees

### No undocumented rules enforced
If a node, relationship, or interface constraint isn't in your `calm.json`, ArchRails won't enforce it.

### Every comment is CALM-traceable
Feedback cites the exact CALM node ID, relationship ID, or interface that triggered it — so teams align faster and argue less.

### PR-scoped bounded context
Reviews resolve only the CALM nodes touched by the PR diff. Less noise, fewer false positives, no cross-service contamination.

### Least-privilege by default
GitHub App permissions are scoped to PR comment write access only. Your source code is never copied or stored.

---

## Monorepo Support

ArchRails supports both single-repo and monorepo setups.

### Option A — One CALM doc, shared mapping

A single `calm.json` at the repo root describes the full system graph. Each service directory maps to its own nodes. A PR touching `services/payment/` won't trigger Order Service rules.

```yaml
governance:
  provider: calm
  file: docs/architecture/calm.json

mapping:
  - path: services/order/src
    node: service-order
  - path: services/payment/src
    node: service-payment
  - path: services/api-gateway/src
    node: service-api-gateway

scopes:
  - name: "Payment"
    match:
      any_prefix: ["services/payment/"]
  - name: "Gateway"
    match:
      any_prefix: ["services/api-gateway/"]
```

### Option B — Per-service CALM documents

Each service owns its `.calm/architecture.json`. The root config binds all sources under one review run without coupling teams.

```yaml
governance:
  provider: calm
  sources:
    - id: payments
      file: services/payments/.calm/architecture.json
      paths:
        - services/payments/
      owned_by: payments-team

    - id: orders
      file: services/orders/.calm/architecture.json
      paths:
        - services/orders/
      owned_by: orders-team

# Each source is only evaluated when the PR diff
# touches its declared paths.

review:
  on: pull_request
  comment_mode: summary       # one top-level comment per PR
  enforce_governance: false   # advisory — won't block merge
```

Set `enforce_governance: true` on any source to block merges on that service's violations independently of others.

---

## GitLab Support

GitLab integration runs via CI pipeline. Add the job below to your `.gitlab-ci.yml` and set three CI/CD variables.

```yaml
stages:
  - review

archrails_review:
  stage: review
  image: alpine:3.20
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
  before_script:
    - apk add --no-cache curl
  script:
    - |
      curl --silent --fail -X POST "${ARCHRAILS_ENDPOINT}" \
        -H "X-Gitlab-Token: ${GITLAB_WEBHOOK_SECRET}" \
        -H "X-Gitlab-Event: Merge Request Hook" \
        -H "Content-Type: application/json" \
        -d "{
          \"archrails_tenant_id\": \"${ARCHRAILS_TENANT_ID}\",
          \"object_kind\": \"merge_request\",
          \"project\": {
            \"id\": ${CI_PROJECT_ID},
            \"path_with_namespace\": \"${CI_PROJECT_PATH}\"
          },
          \"object_attributes\": {
            \"iid\": ${CI_MERGE_REQUEST_IID},
            \"action\": \"update\",
            \"last_commit\": { \"id\": \"${CI_COMMIT_SHA}\" }
          }
        }"
```

**CI/CD Variables to set:**

| Variable | Source |
|---|---|
| `ARCHRAILS_ENDPOINT` | Provided on provisioning |
| `GITLAB_WEBHOOK_SECRET` | Provided on provisioning |
| `ARCHRAILS_TENANT_ID` | Your organization ID |

---

## CALM Visualizer — Free

Before connecting a repo, validate and visualize your `calm.json` in the browser — no account required.

**[Try CALM Visualizer → archrails.io/calm](https://archrails.io/calm)**

- Validates against FINOS CALM 1.2 schema
- Interactive architecture graph with node detail panel
- Flags relationship and interface violations inline
- Free to use — no login, no limits

---

## FAQ

**What is CALM and why does ArchRails use it?**  
CALM (Cloud Architecture Language and Methodology) is a FINOS open standard for describing architecture as machine-readable JSON. ArchRails uses it as its structural backbone because CALM defines nodes, relationships, and interface contracts precisely — enabling deterministic verification rather than probabilistic guessing. The LLM layer then translates those verified violations into readable PR comments.

**Do I need to write CALM from scratch?**  
No. Use the free [CALM Visualizer](https://archrails.io/calm) to paste and validate an existing architecture description, or start from the example in the Config section above. Even a few nodes and relationships gives ArchRails enough to begin enforcing boundaries.

**Does ArchRails store my source code?**  
No. ArchRails receives PR diff events from GitHub or GitLab, processes them in memory, posts a review comment, and discards the data. Your source code is never copied, indexed, or stored. GitHub App permissions are scoped to PR comment write access only.

**How is this different from GitHub Copilot code review?**  
Copilot and generic AI review bots optimize for general best practices drawn from open-source training data. ArchRails enforces your declared CALM architecture — the specific nodes, allowed connections, and interface contracts in your `calm.json`. Every comment is traceable to a CALM node ID.

**How is this different from CALM itself?**  
CALM is the standard and schema. ArchRails is the enforcement layer that puts CALM to work at PR time — mapping source paths to CALM nodes, resolving the bounded context of each diff, and posting traceable, human-readable review comments automatically.

**How do I get started?**  
[Contact us](https://archrails.io/contact) to get provisioned. We'll connect your GitHub or GitLab integration, walk through your first `calm.json`, and get you to your first enforced PR review. The [CALM Visualizer](https://archrails.io/calm) is free to use immediately — no account needed.

---

## About

ArchRails was built by [Marc Daniel Registre](https://archrails.io) to keep architecture consistent as teams scale — without relying on tribal knowledge. By grounding reviews in a CALM architecture graph rather than generic training data, every violation is provable and every comment is traceable.

Built in Miami. Powered by [FINOS CALM](https://finos.org/calm).

---

[archrails.io](https://archrails.io) · [Request Access](https://archrails.io/contact) · [CALM Visualizer](https://archrails.io/calm) · [Privacy](https://archrails.io/privacy) · [Terms](https://archrails.io/terms)
