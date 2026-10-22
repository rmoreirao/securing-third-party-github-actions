---
marp: true
theme: default
size: 16:9
paginate: true
title: Securing Third-Party GitHub Actions at Scale
description: A 30-minute summary of the guide on securely onboarding, adopting and monitoring third-party GitHub Actions.
author: rmoreirao
footer: Securing third-party GitHub Actions · rmoreirao.github.io/securing-third-party-github-actions
style: |
  section {
    font-family: "Segoe UI", -apple-system, BlinkMacSystemFont, "Helvetica Neue", Arial, sans-serif;
    font-size: 24px;
    padding: 48px 64px 56px;
    align-content: start;
    color: #1f2328;
  }
  h1 { font-size: 38px; color: #0b5cad; margin-bottom: 16px; }
  h3 { font-size: 24px; color: #0b5cad; margin: 8px 0 4px; }
  strong { color: #1f2328; }
  code { font-size: 0.85em; }
  table { font-size: 20px; }
  footer { font-size: 14px; color: #656d76; }
  section::after { font-size: 14px; color: #656d76; }
  section.lead { align-content: center; }
  section.lead h1 { font-size: 52px; line-height: 1.15; }
  section.lead p { font-size: 26px; color: #424a53; }
  section.dense { font-size: 21px; }
  section.dense table { font-size: 18px; }
  .cols { display: grid; grid-template-columns: 1fr 1fr; gap: 40px; }
  .stats { display: grid; grid-template-columns: repeat(3, 1fr); gap: 16px 24px; margin-top: 16px; }
  .stats div { background: #f6f8fa; border-radius: 6px; padding: 12px 16px; font-size: 20px; }
  .stats b { display: block; font-size: 40px; line-height: 1.15; color: #0b5cad; }
  .takeaway {
    margin-top: 20px;
    padding: 12px 20px;
    border-left: 6px solid #0b5cad;
    background: #eef5fc;
    font-weight: 600;
  }
  ul.check { list-style: none; padding-left: 0; }
  ul.check li { padding-left: 1.3em; text-indent: -1.3em; margin-bottom: 6px; }
  ul.check li::before { content: "☐ "; color: #0b5cad; }
---

<!-- _class: lead -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# Securing Third-Party GitHub Actions at Scale

Onboard, adopt and monitor them without slowing teams down

**Cloud Solution Architect · Microsoft**
<span style="font-size:18px;color:#656d76">Personal views, not official Microsoft guidance</span>

<!--
Goal of the next 30 minutes: why third-party actions are a supply-chain risk, what a real incident did to a fictional company (Contoso), the ten principles that address it, what changes in a workflow, the four teams that share the work, and the settings checklist you can use for a gap assessment.
The full guide, with a step-by-step story of Contoso, is linked at the end. Backup slides after the closing slide cover egress, the catalog, mirroring and the full lifecycle diagram.
-->

---

# Every `uses:` line is someone else's code

- It runs **next to your source code, your token and your secrets**.
- "Third-party" means anything outside your enterprise: GitHub's own `actions/*`, Microsoft and Azure actions, everything on the Marketplace.
- …and whatever those pull in at run time: **nested actions, container images, downloaded tools**.
- Most teams add them without a second thought.

<div class="takeaway">Goal: give every action that second thought, without slowing anyone down.</div>

<!--
Ask the audience: how many distinct third-party actions run in your enterprise today? Who approved them? At which version?
Out of scope today: building internal actions, general workflow hardening (pull_request_target, script injection), runner platform design.
-->

---

# Tags move. Pins aren't enough.

**March 2025, `tj-actions/changed-files`** (public record):

- An attacker **moved the action's version tags** to a malicious commit, after an earlier compromise of `reviewdog/action-setup`.
- Every workflow using a tag ran the payload on its next run. **More than 23,000 repositories** used the action.
- The payload read the runner's memory and printed secrets, double base64-encoded, into the workflow logs. In public repositories, anyone could read them.

**March 2026, Trivy:** teams that had pinned `aquasecurity/trivy-action` to an older SHA were still exposed, because that commit loaded a nested action **by tag**.

<div class="takeaway">A pin stops a reference from moving. It doesn't make the code safe.</div>

<!--
Public record: CVE-2025-30066 / GHSA-mw4p-6x4p-x5m5.
The Trivy case is why "review the code that actually runs" includes nested actions.
-->

---

<!-- _class: dense -->

# Contoso gets hit

Contoso is fictional: 1,000+ developers, 2,100 private repositories on GitHub Enterprise, everything on Azure. Twenty-four of its repositories used `tj-actions/changed-files` by tag. The release job of one of them held the build bot's token.

<div class="stats">
<div><b>2,100</b>private repositories cloned, within hours</div>
<div><b>40</b>repositories had their secrets stolen by a pushed workflow</div>
<div><b>1</b>Azure secret, Owner on production: customer data copied out</div>
<div><b>9 days</b>a poisoned npm package shipped in 70 services</div>
<div><b>250+</b>credentials rotated, just in case</div>
<div><b>3 weeks</b>of frozen releases, plus a breach notification</div>
</div>

<div class="takeaway">The action leaked secrets for about a day. The attacker used them for three weeks.</div>

<!--
Real incident, fictional company: the numbers are made up, the attack paths aren't.
The build bot's token was a classic personal access token with the repo and workflow scopes, on every repository. The attacker cloned everything, harvested old secrets from ten years of Git history (secret scanning had flagged most of them; the alerts were still open), and pushed a one-file workflow to 40 repositories to steal their secrets.
One of those secrets was AZURE_CREDENTIALS: Owner on the production subscriptions, with Application.ReadWrite.All in Entra ID. Customer data copied out of storage, and new secrets on three other app registrations for persistence.
The bot could bypass branch protection, so a postinstall script in a shared npm package went out through the normal pipeline.
-->

---

<!-- _class: dense -->

# Contoso's post-mortem

| What went wrong | What was missing |
|---|---|
| A moved tag delivered malicious code | An allowlist and SHA pinning |
| Nobody had ever looked at the action | A review before approval |
| The job held secrets it didn't need | Least privilege, and no secrets near third-party code |
| One token could write to every repository, one secret could reach Azure | Short-lived, scoped tokens and OIDC |
| Nothing limited where the payload could send data | Egress-controlled runners |
| Four days to find every usage | An inventory with owners |
| The first alert came a day late | Continuous monitoring |
| Teams added any action they liked | A fast, approved route |
| A day to block, weeks to rotate | A rehearsed response plan |
| Four teams each held part of the answer | One process with owners |

<!--
The next two slides turn these gaps into ten principles. Principle 9, "let every approval expire", isn't here: the incident didn't call for it, but an approved SHA doesn't stay safe forever.
Why it took so long: four teams each held part of the answer, and none of them knew it. No audit log streaming, so the clones were spotted a day late. Nobody owned the Actions policy: three hours to find out who could change it, a day to block. No inventory: four days to find every usage. The Infra team heard on day 3; the persistence in Entra ID turned up in week 3.
-->

---

# The principles (1/2)

1. **Make the secure path the fastest path.** Let teams find approved actions in a catalog and request new actions or updates. Show how long approval takes.
2. **Only run what you've approved, at exact SHAs.** An enforced allowlist. Trusted publishers, a short list you choose, are the one deliberate exception.
3. **Pin everything to a full commit SHA.** `uses: owner/repo@<40-char-sha> # v2.4.1`, enforced by policy.
4. **Review the code that actually runs.** `action.yml`, `dist/`, `pre` and `post` hooks, nested actions, images and downloads. Not the README.
5. **Keep tokens and secrets on a short leash.** Read-only by default, `permissions: {}`, and no secrets near third-party code.

<!--
1: if approval takes weeks, people copy the code into their repo or pick something worse. Security that slows everyone down gets routed around. The catalog is the list of approved actions, with their SHAs, the access each one gets, a sponsor team and dates (backup slide).
4: wait until the repo is 2 months old and the SHA 7 days old; security fixes skip the wait, never the review.
5: an action can read every secret in its job. Split jobs, environment secrets and OIDC keep secrets away.
-->

---

# The principles (2/2)

6. **Lock down the runner, with the team that runs it.** Ephemeral, default-deny egress, and every new destination approved like a secret.
7. **Know what you run, and who owns it.** A catalog, an inventory and a sponsor team, the team that asked for it, for every action.
8. **Keep watching**, trusted publishers included. Dependabot doesn't alert on SHA-pinned actions, so build your own daily check.
9. **Let every approval expire.** Old SHA: 3 months after a newer version, 30 days for High, the same day for Critical.
10. **Be ready for the day an action goes bad.** Block with `!owner/repo@*` in minutes, rotate everything, rehearse.

<!--
6: the Infra team runs the runners and owns their egress rules. A new destination is declared in the request, approved, and recorded in the catalog's network: field (backup slide).
7: the sponsor team asked for the action. It tests it, keeps it up to date, and migrates on time.
8: pinning switches off Dependabot alerts for actions, so you need a replacement safety net: Advisory Database matching, audit log to SIEM.
10: blocking takes one allowlist entry. Knowing which runs and secrets were exposed takes data you must already be collecting.
-->

---

<!-- _class: dense -->

<style scoped>
pre { font-size: 15px; line-height: 1.35; margin: 4px 0; }
h3 { margin-top: 0; }
ul { margin-top: 12px; }
</style>

# What changes in your workflow

<div class="cols">
<div>

### Before

```yaml
# no permissions: block, default token permissions
jobs:
  diff:
    runs-on: ubuntu-latest
    env:  # a secret next to third-party code
      KUBECONFIG: ${{ secrets.KUBECONFIG_PROD }}
    steps:
      - uses: actions/checkout@v4  # tag: can move
      - uses: fabrikam-oss/helm-diff-action@v2
      - run: ./post-diff.sh
```

</div>
<div>

### After

```yaml
permissions: {}
jobs:
  diff:     # third-party code, no secrets
    runs-on: ubuntu-latest
    permissions: { contents: read }
    steps:
      - uses: actions/checkout@<sha>  # vX.Y.Z
        with: { persist-credentials: false }
      - uses: fabrikam-oss/helm-diff-action@<sha>  # v2.5.0
      - uses: actions/upload-artifact@<sha>  # vX.Y.Z
  comment:  # privileges, no third-party code
    needs: diff
    permissions: { pull-requests: write }
```

</div>
</div>

- **Need an action that isn't in the catalog?** Fill in the request form. Trusted publishers: 1–2 business days. Everyone else: 3–5.
- **The policy check fails** when a job asks for more than the catalog allows, or a "no secrets" action can reach a secret. The failure links to the catalog and the form.
- **Before your SHA expires,** you get a warning and a pull request.

<!--
This is the developer's view. Copy the snippet from the catalog: the SHA, with the version as a comment.
The diff job runs the third-party action with read-only access and no secrets, and uploads its result as an artifact. The comment job downloads it and posts it with the token, without running any third-party code.
In the Contoso story, the policy check failed on the first try because the same job held KUBECONFIG_PROD. Moving the diff into its own job turned it green.
The policy check runs as a required workflow through an organization ruleset, and reads the catalog from its own protected repository.
-->

---

<!-- _class: dense -->

# One lifecycle, clear owners

| Step | What happens | Target |
|---|---|---|
| **0 · Bootstrap** | Inventory, catalog, pin, enforce | Once, one organization at a time |
| **1 · Request** | A dev team asks, and sponsors it | A complete request starts the clock |
| **2 · Onboarding** | 2a trusted fast track · 2b full review | 1–2 / 3–5 business days |
| **3 · Adoption** | Use the approved SHA, within its limits | Policy check on every pull request |
| **4 · Monitoring** | What workflows declare vs. what actually ran | Inventory refreshed daily |
| **5 · Lifecycle** | Updates, vulnerabilities, retirement | Old SHA expires 3 months after a newer one |
| **6 · Compromise response** | Block, scope, rotate, recover | Block in minutes |
| **7 · Break-glass** | Time-boxed exceptions, two approvals | 72 hours by default, 7 days at most |

<div class="takeaway">Every new SHA goes back through review, whether it's a new action, an update or a security fix.</div>

<!--
The full lifecycle diagram is a backup slide, and in the guide.
An egress rule for a new destination lands within the onboarding window.
Break-glass is approved by platform and security. Never for a compromised SHA or unreviewed code.
-->

---

<!-- _class: dense -->

<style scoped>
table { font-size: 16px; }
th, td { padding: 4px 10px; }
td:first-child { white-space: nowrap; }
</style>

# Four teams, one process

| Step | Platform team | Dev teams | Security team | Infra team |
|---|---|---|---|---|
| 0 · Bootstrap | Inventory, catalog, enforcement | Merge the pinning pull requests | – | Ephemeral runners, baseline egress allowlist |
| 1 · Request | Triage | Ask, and explain why | Risky requests | – |
| 2 · Onboarding | Run the checks, decide, publish | Review and test | Risky requests | New destinations, egress rules |
| 3 · Adoption | Catalog and policy check | Use as approved | – | – |
| 4 · Monitoring | Inventory, run evidence, dashboards | Fix flagged workflows | SIEM detections | Blocked-egress logs to the SIEM |
| 5 · Lifecycle | Updates, daily checks, expiry dates | Request updates, merge pull requests | Critical findings | – |
| 6 · Compromise | Confirm, contain, scope, track | Rotate their secrets | Investigate | Block the destination, isolate the runner group |
| 7 · Break-glass | Approve and expire | Ask | Co-approve | – |

<div class="takeaway">Platform runs the process. The requesting team sponsors it. Security watches what runs. Infra owns the runners and egress. If the owners don't talk, it won't work.</div>

<!--
In the story, the platform, security, Infra and dev teams sit in one room for the first time, and share one channel for third-party actions from then on. "If a step has no owner, it won't happen."
Payoff in Chapter 9: two minutes after the alert, Security opened the incident in the shared channel, where Platform and the sponsor already were. The action was blocked in 29 minutes. In 2025, the first hour went on finding out who owned the policy.
-->

---

<!-- _class: dense -->

# The allowlist in practice

Actions policy: **Allow enterprise, and select non-enterprise, actions and reusable workflows**, enforced at enterprise level. **Verified creators: off**. A badge tells you who published an action, not that it's safe.

| Entry | What it's for |
|---|---|
| `owner/repo@<full-sha>` | One approved SHA from an untrusted publisher |
| `owner/repo@*` | Every SHA of one repository from a trusted publisher |
| `!owner/repo@*` | Blocking a compromised action, whatever else allows it |
| `!owner/repo@<full-sha>` | Blocking one bad SHA from a trusted repository |

- **Avoid `owner/*`.** It trusts every repository that owner has today or creates tomorrow.
- **Trusted publishers are your own short list, and a trade-off.** Fewer entries, but unreviewed future SHAs get in. Balance with pinning, daily monitoring and a fast block.
- **1,000 entries is a hard limit.** Generate the list from the catalog, keep at most two live SHAs per untrusted action, alert early.

<!--
Trusted publishers in the Contoso story: GitHub, Microsoft and Azure, allowed per repository rather than with the "Allow actions created by GitHub" checkbox, which also covers archived repositories.
If asked "why Microsoft and Azure?": that's Contoso's choice, because it runs on Azure. Pick yours from the publishers you already depend on most, and keep the list short.
-->

---

<!-- _class: dense -->

# When an action leaks your token: detect and contain

<div class="cols">
<div>

### Detect

- Every `git.clone` in the audit log stream carries the actor, the IP address and a hash of the token.
- A Microsoft Sentinel rule runs every 5 minutes and flags any token that cloned **20 or more repositories in 15 minutes**.

<div style="font-size:16px">

```kql
let threshold = 20;
GitHubAuditLog_CL
| where TimeGenerated > ago(15m)
| where Action == "git.clone"
| extend HashedToken =
    tostring(parse_json(RawEvent).hashed_token)
| summarize Repos = dcount(Repository)
    by Actor, AccessType, HashedToken
| where Repos >= threshold
```

</div>
</div>
<div>

### Contain, automatically

- **Cut the token off:** suspend the GitHub App installation, so none of its tokens work.
- **Gather the evidence:** repositories, IP addresses, and the jobs that received the bot's credentials.
- **Call a person**, who lifts the suspension or starts the compromise response.

In Contoso's drill, the rule fired **6 minutes** after the first clone. By then, 64 repositories were gone.

</div>
</div>

<div class="takeaway">Automate only what's cheap to undo. Detection limits the damage; it doesn't prevent it.</div>

<!--
This is what Contoso didn't have in 2025: the build bot's token cloned 2,100 repositories and nobody noticed for a day.
Red team drill in the story: a copy of the build bot's token, cloning from a laptop outside the network. Same pattern as the 2025 incident.
Exclude bots that clone everything by design, like a backup app. Token data on Git events is in public preview.
Suspending a bot for an hour costs some red builds; letting it clone for an hour costs your source code. Blocking an action stays a human call: the SIEM sees the token, not the action that leaked it.
After the drill, Contoso narrowed the bot's tokens to the repositories each job needs.
-->

---

<!-- _class: dense -->

# Settings checklist (1/2)

<div class="cols">
<div>

### Policy

<ul class="check">
<li>Actions policy: <b>Allow enterprise, and select non-enterprise, actions and reusable workflows</b>, enforced</li>
<li><b>Allow Marketplace actions by verified creators:</b> off</li>
<li>Allowlist generated from the catalog, with alerts on manual edits</li>
<li><b>Require actions to be pinned to a full-length commit SHA:</b> on</li>
</ul>

</div>
<div>

### Permissions

<ul class="check">
<li><b>Workflow permissions:</b> read repository contents and packages</li>
<li><b>Allow GitHub Actions to create and approve pull requests:</b> off</li>
<li>A required policy check on every pull request, through an organization ruleset</li>
</ul>

### GitHub Advanced Security

<ul class="check">
<li>CodeQL (including the <code>actions</code> language), secret scanning with push protection, and Dependabot, wherever third-party code is reviewed or used</li>
</ul>

</div>
</div>

<!--
Use this as a live gap assessment: walk through each box and mark green / amber / red for the customer's enterprise.
SHA pinning applies to every remote action, internal ones too, so migrate first (pinact, Renovate or a script).
-->

---

<!-- _class: dense -->

# Settings checklist (2/2)

<div class="cols">
<div>

### Runners

<ul class="check">
<li>Ephemeral runners by default</li>
<li>Default-deny egress on self-hosted runners</li>
<li>Self-hosted runner groups limited to selected organizations and repositories</li>
<li>No sensitive secrets in third-party jobs on standard GitHub-hosted runners</li>
<li>Every network destination in the catalog has an egress rule, scoped to a runner group</li>
<li>Blocked-egress alerts go to the platform, security and Infra teams</li>
</ul>

</div>
<div>

### Monitoring

<ul class="check">
<li>Audit log streamed to your SIEM, with detection rules, kept for your incident look-back window</li>
<li>Daily checks: advisories, Dependabot on reviewed copies, moved tags</li>
<li>A catalog with a sponsor and an expiry date for every SHA</li>
</ul>

</div>
</div>

<!--
Standard GitHub-hosted runners have no egress control. When a job needs both third-party code and sensitive data: egress-controlled self-hosted runners, GitHub-hosted runners with private networking, or accept the risk in writing.
Detections worth starting with: Actions policy changes outside automation; a third-party job receiving a secret its catalog entry doesn't allow; a CI token cloning far more than usual, contained automatically; a self-hosted runner blocking outbound traffic while a third-party action runs.
-->

---

# Where to start

1. **Take an inventory.** What workflows declare, with tags resolved to SHAs and nested actions included, plus what ran, from job logs. Note the SHA and whether the job holds secrets.
2. **Sort it.** Trusted publisher: allow the repository. Untrusted and in use: review the SHA in use. Unused or duplicated: drop it.
3. **Seed the catalog and the allowlist.** One SHA per untrusted action, and a sponsor for each.
4. **Pin.** Automation opens pull requests that rewrite every `uses:` line to a full SHA.
5. **Enforce, one step at a time.** Policy check in warning mode, then the allowlist, then SHA pinning, then blocking. Announce each step a month ahead.

<div class="takeaway">Warn first, enforce later. Start with the jobs that hold secrets.</div>

<!--
Tools: poutine or a script for the declared inventory (lockfiles on GitHub Enterprise Cloud), job logs for what ran, pinact for pinning, zizmor and OpenSSF Scorecard for review. A dependency graph SBOM only sees direct uses: on the default branch, never what ran.
Break-glass works from day one, so enforcement never leaves a team stuck.
-->

---

<!-- _class: lead -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# Thank you. Questions?

**The full guide:** rmoreirao.github.io/securing-third-party-github-actions

The principles and settings, plus the Contoso story: who does what, request forms, an onboarding checklist with criticality levels, catalog schema, expiry rules and the incident runbook.

<span style="font-size:18px;color:#656d76">Contoso and Fabrikam are fictional. Licensed under CC BY 4.0.</span>

<!--
Offer next steps: a gap assessment against the checklist, or a pilot on one organization following the bootstrap steps.
Backup slides follow, for questions: new network destinations, a catalog entry, upstream vs. mirror vs. wrapper, and the full lifecycle diagram.
-->

---

<!-- _class: dense -->

# Backup: when an action needs a new destination

A deployment to a third-party vendor, for example. The Infra team that runs the runners is part of the decision.

- **Treat a network destination like a secret.** Declare it in the request, approve it, and record it in the catalog's `network:` field.
- **Scope rules to a runner group, not the shared pool.** For example, a `deploy-northwind` group that only the repositories deploying to that vendor can use.
- **Allow exact hostnames.** No wildcards on shared hosting like `*.amazonaws.com` or `*.blob.core.windows.net`: attackers can host there too.
- **Find out what the action actually calls.** During onboarding, run it on a test runner that logs traffic without blocking it, and compare that with what the request declared.
- **Split the job.** The third-party action runs with no secrets and no extra network. A separate deploy job uses the vendor's CLI, with OIDC where the vendor supports it, and can reach only the vendor.

<div class="takeaway">The catalog is the source of truth. The Infra team, or automation, turns it into egress rules.</div>

<!--
Ideally the egress rules change through a pull request to an egress-as-code repository, generated from the catalog.
Target in the story: the egress rule for a new destination lands within the onboarding window.
-->

---

<!-- _class: dense -->

<style scoped>
pre { font-size: 15px; line-height: 1.35; margin: 0; }
</style>

# Backup: a catalog entry

<div class="cols">
<div>

```yaml
action: fabrikam-oss/helm-diff-action
publisher: untrusted
sponsor: payments
access:
  permissions: { contents: read }
  secrets: none
  oidc: false
  network: [charts.contoso.internal]
versions:
  - version: v2.6.0
    sha: <40-char-sha>
    status: active          # active | expiring | expired | compromised
    approved: 2025-11-10
    review_copy: helm-diff-action-<sha>
  - version: v2.5.0
    sha: <40-char-sha>
    status: expiring
    approved: 2025-09-12
    expires: 2026-02-10
    review_copy: helm-diff-action-<sha>
```

</div>
<div>

- **One entry per action**, as YAML in a protected repository, in repository custom properties, or in a developer portal.
- **`access`** is the most any job using it may get. The policy check enforces it.
- **`network`** feeds the egress rules.
- **`status` and `expires`** drive the warnings, the migration pull requests and the allowlist.
- **No list of repositories.** The inventory rebuilds it every day.

</div>
</div>

<!--
review_copy is the reviewed copy of that exact SHA, kept in an isolated environment. Dependabot keeps scanning it after approval.
The allowlist is generated from the active and expiring SHAs, so it never drifts from the catalog.
-->

---

<!-- _class: dense -->

# Backup: upstream, mirror or wrapper?

Once an action is approved, where do workflows pull it from? You can choose per action.

| Model | Workflows use | Good | Hard | Pick it for |
|---|---|---|---|---|
| **Vetted upstream** | `owner/action@<approved-sha>` | Simplest. Exact-SHA allowlisting | You still depend on the upstream being there | Most actions |
| **Internal mirror** | `enterprise/action-mirror@<approved-sha>` | Availability and control | You become the distributor. Exposing only approved SHAs is hard. Nested actions need mirroring too | Critical actions with a fragile upstream |
| **Internal wrapper or replacement** | An internal action or reusable workflow | Full ownership | Highest effort to build and maintain | Strategic or heavily used capabilities |

<div class="takeaway">Mirroring everything makes you the distributor of code you didn't write. Use the vetted upstream by default.</div>

<!--
Contoso uses the vetted upstream at an exact SHA, and keeps a reviewed copy of every approved SHA in an isolated GHE instance, which covers the "upstream vanished" risk. It moves an action to a mirror or an internal wrapper only when it's critical enough to justify the work.
-->

---

<!-- _footer: "" -->

![bg contain](../assets/third-party-actions-lifecycle.svg)

<!--
Backup: the full lifecycle diagram, the same one as in the guide. Zoom in on the PDF for detail.
-->
