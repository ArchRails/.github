# ArchRails

**Automated architecture reviews at pull request time.**

ArchRails enforces your team’s **actual architecture standards**—not generic best practices—by reviewing pull requests against **repo-defined rules, documentation, diagrams, ADRs, and gold-standard PRs**, then leaving **inline, actionable feedback** directly on the code.

---

## Why ArchRails?

Code reviews often catch style and correctness issues, but **architecture drift** slips through:
- ViewModels start doing UI work
- Boundaries blur between modules
- “Temporary” shortcuts become permanent patterns

ArchRails exists to make architectural intent **explicit, enforceable, and scalable**.

---

## How It Works

1. **You define your architecture**
   - Architecture rules (e.g. MVVM, Clean, MVI, or internal patterns)
   - Documentation, diagrams, ADRs
   - Gold-standard pull requests that represent “the right way”

2. **ArchRails runs on every pull request**
   - Reads your architecture config and repo context
   - Reviews only the files changed in the PR
   - Evaluates changes *in architectural context*

3. **Inline feedback is posted**
   - Comments appear on the exact lines that matter
   - Each finding explains **what**, **why**, and **how to fix it**
   - Feedback is evidence-backed, not opinionated

---

## What Makes ArchRails Different

| ArchRails | Typical Tools |
|---------|---------------|
| Learns from *your* repo | Hard-coded rules |
| Uses docs, ADRs, diagrams, PRs | Ignores architectural intent |
| Architecture-aware | Syntax & style focused |
| Evidence-backed feedback | Generic suggestions |

ArchRails does **not** replace human reviewers—it removes repetitive architectural policing so reviewers can focus on design, tradeoffs, and mentorship.

---

## What You Can Enforce

- MVVM, Clean Architecture, MVI, or custom internal patterns  
- Layer and module boundaries  
- Ownership rules (what can depend on what)  
- Architectural constraints defined by your team  
- Consistency across repos and contributors  

---

## Example Feedback

- ❌ ViewModel contains navigation logic (violates MVVM constraint)
- ⚠️ Domain module depends on UI module (boundary violation)
- ✅ Correct use of state holder aligned with gold-standard PR

Each comment includes context from your docs or reference PRs.

---

## Who It’s For

- Engineering teams scaling beyond “everyone knows the architecture”
- Tech leads tired of repeating the same architectural comments
- Organizations that care about **long-term maintainability**

---

## Status

🚧 ArchRails is under active development.  
Early adopters are helping shape:
- Configuration format
- Supported architecture patterns
- Enterprise deployment options

---

## Get Involved

- ⭐ Star the repo to follow progress
- 💬 Open discussions with feedback or ideas
- 📩 Reach out if you want to pilot ArchRails with your team

---

**Enforce your architecture.  
At pull request time.**

## Early Access / Provisioning

ArchRails is under active development and currently provisioned manually.

If you’d like to pilot ArchRails with your team or discuss enterprise use cases, reach out directly:

📧 **hello@archrails.io**
