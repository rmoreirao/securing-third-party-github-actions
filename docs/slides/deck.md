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
Goal of the next 30 minutes: why third-party actions are a supply-chain risk, the ten principles that address it, and the settings checklist you can use for a gap assessment.
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

6. **Lock down the runner.** Ephemeral runners, default-deny egress, and nothing sensitive where you can't control the network.
7. **Know what you run, and who owns it.** A catalog, an inventory and a sponsor team for every action.
8. **Keep watching**, trusted publishers included. Dependabot doesn't alert on SHA-pinned actions, so build your own daily check.
9. **Let every approval expire.** Old SHA: 3 months after a newer version, 30 days for High, the same day for Critical.
10. **Be ready for the day an action goes bad.** Block with `!owner/repo@*` in minutes, rotate everything, rehearse.

<div class="takeaway">The platform team runs the process and the tooling. The team that asked for an action sponsors it.</div>

<!--
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
Onboarding targets in the story: 1–2 business days for trusted publishers, 3–5 for everyone else.
Break-glass: 72 hours by default, 7 days at most, approved by platform and security. Never for a compromised SHA or unreviewed code.
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
Detections worth starting with: Actions policy changes outside automation; a third-party job receiving a secret its catalog entry doesn't allow; a CI token cloning far more than usual.
-->

---

# Where to start

1. **Take an inventory.** SBOMs for every repository, plus code search for `uses:`. Note the ref, the SHA it resolves to, and whether the job holds secrets.
2. **Sort it.** Trusted publisher: allow the repository. Untrusted and in use: review the SHA in use. Unused or duplicated: drop it.
3. **Seed the catalog and the allowlist.** One SHA per untrusted action, and a sponsor for each.
4. **Pin.** Automation opens pull requests that rewrite every `uses:` line to a full SHA.
5. **Enforce, one step at a time.** Policy check in warning mode, then the allowlist, then SHA pinning, then blocking. Announce each step a month ahead.

<div class="takeaway">Approval is the start, not the finish.</div>

<!--
Tools: github-sbom-toolkit for the enterprise-wide inventory, pinact for pinning, zizmor and OpenSSF Scorecard for review.
Break-glass works from day one, so enforcement never leaves a team stuck.
-->

---

<!-- _class: lead -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# Thank you. Questions?

**The full guide:** rmoreirao.github.io/securing-third-party-github-actions

The principles and settings, plus the Contoso story: request forms, catalog schema, expiry rules and the incident runbook.

<span style="font-size:18px;color:#656d76">Contoso and Fabrikam are fictional. Licensed under CC BY 4.0.</span>

<!--
Offer next steps: a gap assessment against the checklist, or a pilot on one organization following the bootstrap steps.
-->
