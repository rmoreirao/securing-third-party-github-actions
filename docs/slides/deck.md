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
Goal of the next 30 minutes: why third-party actions are a supply-chain risk, the ten principles that address it, the four teams that share the work, and the settings checklist you can use for a gap assessment.
The full guide, with a step-by-step story of a fictional company (Contoso), is linked at the end.
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

# March 2025: `tj-actions/changed-files`

- An attacker **moved the action's version tags** to a malicious commit, after an earlier compromise of `reviewdog/action-setup`.
- Every workflow using a tag ran the payload on its next run. **More than 23,000 repositories** used the action.
- The payload read the runner's memory and printed secrets, double base64-encoded, into the workflow logs. In public repositories, anyone could read them.

**March 2026, Trivy:** teams that had pinned `aquasecurity/trivy-action` to an older SHA were still exposed, because that commit loaded a nested action **by tag**.

<div class="takeaway">Tags move. Commits don't. And a pin stops a reference from moving; it doesn't make the code safe.</div>

<!--
Public record: CVE-2025-30066 / GHSA-mw4p-6x4p-x5m5.
The Trivy case is why "review the code that actually runs" includes nested actions.
-->

---

<!-- _class: dense -->

# What went wrong, and what was missing

| What went wrong | What was missing |
|---|---|
| A moved tag delivered malicious code | An allowlist and SHA pinning |
| Nobody had ever looked at the action | A review before approval |
| The job held secrets it didn't need | Least privilege, and no secrets near third-party code |
| Nothing limited where the payload could send data | Egress-controlled runners |
| Days to find every usage | An inventory with owners |
| The first alert came a day late | Continuous monitoring |
| Teams added any action they liked | A fast, approved route |
| A day to block, weeks to rotate | A rehearsed response plan |

<!--
This is the post-mortem of the fictional Contoso in the guide, built from the real incident. Each row maps to one of the ten principles on the next two slides.
Why it took so long: three teams each held part of the answer, and none of them knew it. Security had flagged unpinned actions in a report nobody else read. The platform team had SHA pinning on its backlog. Dev teams had built their own fixes. On the day, the first three hours went on finding out who could change the Actions policy.
-->

---

# The principles (1/2)

1. **Make the secure path the fastest path.** Self-service requests, a catalog and published turnaround times.
2. **Only run what you've approved, at exact SHAs.** An enforced allowlist. Trusted publishers are the one deliberate exception.
3. **Pin everything to a full commit SHA.** `uses: owner/repo@<40-char-sha> # v2.4.1`, enforced by policy.
4. **Review the code that actually runs.** `action.yml`, `dist/`, `pre` and `post` hooks, nested actions, images and downloads. Not the README.
5. **Keep tokens and secrets on a short leash.** Read-only by default, `permissions: {}`, and no secrets near third-party code.

<!--
1: if approval takes weeks, people copy the code into their repo or pick something worse. Security that slows everyone down gets routed around.
4: wait until the repo is 2 months old and the SHA 7 days old; security fixes skip the wait, never the review.
5: an action can read every secret in its job. Split jobs, environment secrets and OIDC keep secrets away.
-->

---

# The principles (2/2)

6. **Lock down the runner, with the team that runs it.** Ephemeral, default-deny egress, and every new destination approved like a secret.
7. **Know what you run, and who owns it.** A catalog, an inventory and a sponsor team for every action.
8. **Keep watching**, trusted publishers included. Dependabot doesn't alert on SHA-pinned actions, so build your own daily check.
9. **Let every approval expire.** Old SHA: 3 months after a newer version, 30 days for High, the same day for Critical.
10. **Be ready for the day an action goes bad.** Block with `!owner/repo@*` in minutes, rotate everything, rehearse.

<div class="takeaway">Platform runs the process. The requesting team sponsors it. Security watches what runs. Infra owns the runners and egress. None of them can do it alone.</div>

<!--
6: the Infra team runs the runners and owns their egress rules. A new destination is declared in the request, approved, and recorded in the catalog's network: field.
8: pinning switches off Dependabot alerts for actions, so you need a replacement safety net: Advisory Database matching, audit log to SIEM.
10: blocking takes one allowlist entry. Knowing which runs and secrets were exposed takes data you must already be collecting.
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
- **Trusted publishers are a trade-off.** Fewer entries, but unreviewed future SHAs get in. Balance with pinning, daily monitoring and a fast block.
- **1,000 entries is a hard limit.** Generate the list from the catalog, keep at most two live SHAs per untrusted action, alert early.

<!--
Trusted publishers in the Contoso story: GitHub, Microsoft and Azure, allowed per repository rather than with the "Allow actions created by GitHub" checkbox, which also covers archived repositories.
-->

---

<!-- _class: dense -->

# When an action needs a new destination

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

![bg right:38% contain](../assets/third-party-actions-lifecycle.svg)

# One lifecycle, clear owners

<div style="font-size:21px">

**0 · Bootstrap**: inventory, catalog, pin, enforce
**1 · Request**: a dev team asks, and sponsors it
**2 · Onboarding**: 2a trusted fast track · 2b full review
**3 · Adoption**: use the approved SHA, within its limits
**4 · Monitoring**: what workflows declare vs. what actually ran
**5 · Lifecycle**: updates, vulnerabilities, retirement
**6 · Compromise response**: block, scope, rotate, recover
**7 · Break-glass**: time-boxed exceptions, two approvals

</div>

<div class="takeaway">Every new SHA goes back through review, whether it's a new action, an update or a security fix.</div>

<!--
Onboarding targets in the story: 1–2 business days for trusted publishers, 3–5 for everyone else. An egress rule for a new destination lands within the same window.
Break-glass: 72 hours by default, 7 days at most, approved by platform and security. Never for a compromised SHA or unreviewed code.
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

<div class="takeaway">If a step has no owner, it won't happen. If the owners don't talk, it won't work.</div>

<!--
In the story, the platform, security, Infra and dev teams sit in one room for the first time, and share one channel for third-party actions from then on.
Payoff in Chapter 9: two minutes after the alert, Security opened the incident in the shared channel, where Platform and the sponsor already were. The action was blocked in 29 minutes. In 2025, the first hour went on finding out who owned the policy.
-->

---

<!-- _class: dense -->

# Detect and contain a leaked token

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

<div class="takeaway">Approval is the start, not the finish.</div>

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
-->
