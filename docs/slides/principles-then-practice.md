---
marp: true
theme: default
size: 16:9
paginate: true
title: "Securing Third-Party GitHub Actions at Scale: Principles, Then Practice"
description: "A 30-minute talk in two clear parts: the security principles, then Contoso's application of them."
author: rmoreirao
footer: Securing third-party GitHub Actions · Principles, then practice
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
  table { display: table; width: 100%; font-size: 22px; }
  table code { font-size: 0.9em; white-space: nowrap; }
  footer { font-size: 14px; color: #656d76; }
  section::after { font-size: 14px; color: #656d76; }
  section.lead { align-content: center; }
  section.lead h1 { font-size: 52px; line-height: 1.15; }
  section.lead p { font-size: 26px; color: #424a53; }
  section.dense { font-size: 21px; }
  section.dense table { font-size: 22px; }
  .section-label { font-size: 16px; font-weight: 600; color: #656d76; margin: 0 0 8px; }
  .cols { display: grid; grid-template-columns: 1fr 1fr; gap: 32px; }
  .cards { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 20px; }
  .cards > div { background: #f6f8fa; border-radius: 6px; padding: 16px 20px; }
  .cards p { margin: 10px 0 0; }
  .metric { display: block; font-size: 42px; line-height: 1.2; color: #0b5cad; }
  .route { background: #eef5fc; border-radius: 6px; padding: 16px 20px; margin-top: 20px; }
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

**Principles, then practice**

Onboard, adopt and monitor them without slowing teams down

**Cloud Solution Architect · Microsoft**
<span style="font-size:18px;color:#656d76">Personal views, not official Microsoft guidance</span>

<!--
Time: 0:30. Elapsed: 0:30.
This is an alternative to the original deck, not its replacement.
The talk has two separate parts. First explain the general framework without the Contoso story. Then apply it in chronological order, without stopping for another principles lecture.
Presentation: 27 minutes. Questions: 3 minutes. Backup slides are outside that time.
-->

---

# Two parts. One clear route.

<div class="cols">
<div class="route">

### Part 1 · Principles

**The framework, for any organization**

- Approve what runs.
- Limit what it can reach.
- Keep managing it.

Make the secure path the fastest path.

</div>
<div class="route">

### Part 2 · Practice

**Contoso applies the framework**

- Clean up the existing fleet.
- Follow one action into production.
- Update, detect, contain and recover.

A fictional story with concrete decisions.

</div>
</div>

<!--
Time: 0:30. Elapsed: 1:00.
Three minutes of context, nine minutes on principles, twelve minutes on practice, three minutes to bring it back to the audience, then questions.
The guide uses the same two-part structure. The complete settings checklist is in backup, not a third section in the middle of the story.
-->

---

# Every `uses:` line is someone else's code

It runs next to **your source code, your token and your secrets**.

<div class="cards">
<div>

### The reference

**Tags can move.**

**March 2025:**

`tj-actions/changed-files`

An attacker moved its version tags to malicious code.

</div>
<div>

### The execution

**The entry point isn't all the code.**

Nested actions, bundles, hooks, images and downloaded tools run too.

</div>
<div>

### The access

**A pin isn't a sandbox.**

Malicious code can use whatever the job and runner make available.

</div>
</div>

<div class="takeaway">Control the code, its access and what happens after approval.</div>

<!--
Time: 2:00. Elapsed: 3:00.
Public record: CVE-2025-30066 / GHSA-mw4p-6x4p-x5m5. The payload read runner memory and printed secrets, double base64-encoded, into workflow logs. More than 23,000 repositories used the action; that is not a count of confirmed secret leaks.
Third-party means outside your enterprise, including GitHub's actions/*, Microsoft and Azure actions.
The March 2026 Trivy incident is another example: an older pinned commit loaded a nested action by tag. Use that only as a brief illustration of the execution column, not a second incident story.
Out of scope: building internal actions, general workflow hardening and runner platform design.
-->

---

<!-- _class: lead -->
<!-- _footer: "Part 1 · Principles" -->

# Part 1 · Principles

The framework, before the story

<div class="route">Approve what runs → Limit its access → Keep managing it</div>

<!--
Time: 0:15. Elapsed: 3:15.
No fictional characters in this part. These are the ten principles from the guide, grouped for the talk. Their original numbers are retained so the audience can find them in the guide.
-->

---

<p class="section-label">Part 1 · Principles</p>

# 1. Make the secure path the fastest path

Search the catalog → Request what's missing → Get a decision → Copy the snippet

| Team | Its part of the secure path |
|---|---|
| **Platform** | Runs the process, catalog, checks and policy |
| **Dev teams** | Sponsor requests, test, adopt and migrate |
| **Security** | Watches what runs; joins risky requests and incidents |
| **Infra team** | Runs the runners and owns their egress rules |

- **Publish turnaround times:** 1–2 business days for trusted publishers, 3–5 for everyone else.
- Automate checks that don't need a human. Link every policy failure to the catalog and the request form.

<div class="takeaway">If the approved route is slower than a workaround, people take the workaround.</div>

<!--
Time: 1:45. Elapsed: 5:00.
Principle 1 underpins all three groups on the next slides.
The requesting dev team is the sponsor: it shares responsibility for review, testing and keeping the action current. Other consuming teams must use it within its limits and migrate on time.
This is a self-service product, not a security ticket queue. A portal, GitHub issue forms or an existing ITSM tool can provide the front door.
The two turnaround times come from two onboarding routes. The next slide explains them.
-->

---

<style scoped>
.route code { white-space: nowrap; }
</style>

<p class="section-label">Part 1 · Principles · Approve what runs</p>

# Two doors: trusted publishers and everyone else

<div class="cols">
<div class="route">

### Trusted publishers

- A **short list you choose**: organizations, never individuals.
- Allow the repository: `owner/repo@*`
- **Fast track:** 1–2 business days.
- **The price:** future SHAs run without review.

</div>
<div class="route">

### Everyone else

- Any other publisher, verified badge or not.
- Allow one SHA: `owner/repo@<full-sha>`
- **Full review:** 3–5 business days.
- **Every new version** goes back through review.

</div>
</div>

<div class="takeaway">Trust a few publishers for speed. Balance it with pinning, daily monitoring and a fast block.</div>

<!--
Time: 1:00. Elapsed: 6:00.
This is the one deliberate exception to principle 2. It's what makes the fast path possible for the actions teams use most.
Allowing owner/repo@* saves review work and allowlist entries, but it lets in future SHAs nobody reviewed. Accept that only for a short list of organizations.
Pick the publishers you already depend on most. Good criteria: a verified identity, a security policy, enforced 2FA, and protected tags or immutable releases. Review the list twice a year.
Allow trusted publishers per repository, not with owner/*, which trusts repositories that don't exist yet. The "Allow actions created by GitHub" checkbox also covers archived repositories.
A Marketplace verified-creator badge tells you who published an action, not that it's safe. It isn't a reason to trust a publisher.
Trusted doesn't mean unwatched: SHA pinning still applies, daily monitoring covers these repositories too, and one deny entry blocks a bad SHA or the whole repository.
In Part 2, the Infra team's azure/login request takes the fast track and Fabrikam's action gets the full review.
-->

---

<p class="section-label">Part 1 · Principles · Approve what runs</p>

# Approve the code, not the badge

<div class="cards">
<div>

### 2. Only run what you've approved, at exact SHAs

Enforce an allowlist generated from the catalog.

Exact SHAs, except for your trusted publishers.

</div>
<div>

### 3. Pin everything to a full commit SHA

Tags move. Commits don't.

Keep the version as a comment for humans.

</div>
<div>

### 4. Review the code that actually runs

Read the bundle, hooks and nested actions.

Check images and downloads too.

</div>
</div>

<div class="takeaway">Pinning fixes the reference. Review evaluates the code. Neither replaces the other.</div>

<!--
Time: 1:30. Elapsed: 7:30.
An exact-SHA allowlist entry authorizes one reviewed revision; the pinning setting requires workflows to name a full SHA. They are separate controls.
Trusted publishers were covered on the previous slide; don't repeat the trade-off here.
Review the exact SHA in isolation, including action.yml, dist/, pre/post hooks, nested actions, images and runtime downloads. Use GHAS, the Advisory Database, Scorecard and zizmor. AI advice is not an approval.
Default minimum ages: repository 2 months, SHA 7 days. Security fixes skip the wait, never the review.
Full allowlist syntax is in backup.
-->

---

<p class="section-label">Part 1 · Principles · Limit what it can reach</p>

# Assume the review can miss something

<div class="cols">
<div class="route">

### 5. Keep tokens and secrets on a short leash

- Read-only by default; grant permissions per job.
- Keep secrets away from third-party code.
- Split jobs; use scoped, short-lived tokens and OIDC.
- Don't persist checkout credentials.

</div>
<div class="route">

### 6. Lock down the runner, with the team that runs it

- One job, then discard the runner.
- Default-deny egress on self-hosted runners.
- Approve network destinations like secrets.
- Scope runner groups to selected repositories.

</div>
</div>

<div class="takeaway">When code turns malicious, its access determines the blast radius.</div>

<!--
Time: 2:00. Elapsed: 9:30.
An action can reach secrets and environment variables available to its job, not just the values intended for its own step.
Start with permissions: {}, grant per job, set persist-credentials: false on checkout, and avoid secrets: inherit and toJSON(secrets).
Standard GitHub-hosted runners are ephemeral but have no egress control. Keep sensitive secrets and data out of third-party jobs there. For jobs needing sensitive data, use egress-controlled self-hosted runners, private networking, or written risk acceptance.
The Infra team owns egress. The catalog declares exact destinations, approved rules are scoped to a runner group, and blocked calls go to the SIEM.
-->

---

<!-- _class: dense -->

<p class="section-label">Part 1 · Principles · Keep managing it</p>

# Approval is the start, not the finish

| Principle | What must keep happening |
|---|---|
| **7. Know what you run, and who owns it** | Catalog, daily inventory, runtime evidence and a sponsor |
| **8. Keep watching** | Daily advisory checks; detections on what actually ran, trusted publishers included |
| **9. Let every approval expire** | Migrate off old SHAs; remove them on the date; expire exceptions too |
| **10. Be ready for the day an action goes bad** | Block, scope, rotate and recover with a rehearsed runbook |

<div class="route">Approved SHA → Watched while in use → Updated or blocked → Retired</div>

<div class="takeaway">You need evidence and owners before the incident, not after it.</div>

<!--
Time: 2:30. Elapsed: 12:00.
These are the remaining four principles. Part 1 is now complete.
Distinguish the catalog (what is approved), the inventory (what workflows declare), and run evidence (what executed, with which access).
Dependabot doesn't raise alerts for SHA-pinned actions. Replace that safety net with daily advisory matching and dependency scanning on reviewed copies.
Defaults: an old SHA expires 3 months after a newer version is approved, 30 days after a High finding, the same day for Critical; compromise is blocked immediately. Warn and open migration PRs.
Keep audit logs for the incident look-back window. Rehearse a tabletop twice a year and a technical drill quarterly.
Transition: the framework is complete. From here, follow Contoso chronologically; don't repeat the principles as another checklist.
-->

---

<!-- _class: lead -->
<!-- _footer: "Part 2 · Practice" -->

# Part 2 · Practice

How Contoso puts the framework to work

<div class="route">Bootstrap → Request → Onboard → Adopt → Monitor → Update → Respond → Recover</div>

<!--
Time: 0:15. Elapsed: 12:15.
Clear boundary: the general guidance is finished. Contoso, Fabrikam, the people and the numbers in this part are fictional, based on real incidents and practices.
The scenes follow steps 0–7 from the guide. They demonstrate decisions and results, not a second explanation of the whole framework.
-->

---

<style scoped>
.cards + p { margin-top: 24px; }
</style>

<p class="section-label">Part 2 · Practice · March 2025</p>

# Contoso has a bad day

Twenty-four repositories used `tj-actions/changed-files` **by tag**.
One release job held the build bot's broadly privileged token.

<div class="cards">
<div>

<b class="metric">2,100</b>
private repositories cloned

</div>
<div>

<b class="metric">250+</b>
credentials rotated, just in case

</div>
<div>

<b class="metric">3 weeks</b>
of frozen releases

</div>
</div>

**Marta (Platform)**, **Sam (Security)**, **Ana (Payments)** and the **Infra team** agree on one process.

<div class="takeaway">The brief: make third-party actions safer, without slowing teams down.</div>

<!--
Time: 1:15. Elapsed: 13:30.
All company outcomes on this slide are fictional. The underlying tj-actions incident is public record, described in the opening.
Contoso has 1,000+ developers in 100+ teams, with 2,100 private repositories and Azure infrastructure.
The leaked classic PAT could clone every repository and write workflows. The attacker stole more secrets, reached Azure and poisoned a shared npm package. Finding usage took four days; blocking took a day.
Four teams held parts of the answer but had no shared owner or channel. Don't retell every attack path; establish why the operating model must change.
-->

---

<p class="section-label">Part 2 · Practice · 0 · Bootstrap</p>

# First, clean up what's already running

**April 2025:** Marta's inventory finds 350 third-party actions.
Twenty-two do the same thing: post to Slack.

| Order | What Contoso does |
|---|---|
| **Discover** | Read workflows and job logs; resolve tags and nested actions |
| **Reduce** | Drop duplicates; review untrusted SHAs in use; assign sponsors |
| **Prepare** | Seed the catalog and allowlist; open pinning pull requests |
| **Enforce** | Warn first, then allowlist, require SHA pins, then block policy failures |

The **Infra team** prepares ephemeral runners and baseline egress rules.

<div class="takeaway">Existing teams get migration help before the restrictions arrive.</div>

<!--
Time: 1:15. Elapsed: 14:45.
The inventory found 1,800 uses lines; two thirds used tags. The declared inventory and the last 90 days of job logs answer different questions.
Start with jobs holding secrets. Trusted repositories are allowlisted; untrusted SHAs get reviewed. Every retained action gets a sponsor.
Announce each enforcement step a month ahead, one organization at a time. Break-glass works from day one.
Do not turn this into the detailed bootstrap procedure; the result is a controlled starting point for the new request.
-->

---

<p class="section-label">Part 2 · Practice · 1 · Request → 2 · Onboarding</p>

# Ana needs Helm diffs on pull requests

**September 2025:** the catalog has no match.
**Ana (Sponsor)** requests `fabrikam-oss/helm-diff-action`.

<div class="cols">
<div class="route">

### 2a · Trusted Publisher Onboarding

The Infra team's `azure/login` request takes the fast track.

Identity and advisory checks, then allowlist that repository.

**Next morning. Target: 1–2 business days.**

</div>
<div class="route">

### 2b · Untrusted Publisher Onboarding

Fabrikam isn't trusted. Platform and Payments review the exact SHA in isolation.

The catalog records its access, network and sponsor.

**Four days. Target: 3–5 business days.**

</div>
</div>

<!--
Time: 1:30. Elapsed: 16:15.
Ana first searches the catalog, then records the need, alternatives, repositories, permissions, secrets, OIDC and network. A complete request starts the clock.
Contoso's trusted publishers are actions, github, microsoft and azure, allowed per repository. This is Contoso's choice, not a universal recommended list.
For Fabrikam v2.5.0, Platform keeps a copy at the upstream SHA in an isolated Review GHE, scans it and reviews with the sponsor. Security joins risky requests. Infra applies any new destination rule within the onboarding window.
The approved access contract is contents: read, no secrets, no OIDC, network charts.contoso.internal. Workflows use the upstream at the approved SHA.
Keep the focus on why the two requests take different paths; the review checklist is in the guide.
-->

---

<!-- _class: dense -->

<style scoped>
pre { font-size: 18px; line-height: 1.35; margin: 4px 0; }
pre code { font-size: inherit; }
</style>

<p class="section-label">Part 2 · Practice · 3 · Adoption</p>

# The first policy check fails. The second passes.

<div class="cols">
<div>

### First attempt (excerpt)

```yaml
jobs:
  diff:
    env:
      KUBECONFIG: ${{ secrets.KUBECONFIG_PROD }}
    steps:
      - uses: fabrikam-oss/helm-diff-action@<sha>
```

**Blocked:** the catalog says no secrets.

</div>
<div>

### Split jobs (excerpt)

```yaml
permissions: {}
jobs:
  diff:      # Helm action; no secrets
    permissions: { contents: read }
    # Upload the diff artifact.
  comment:   # No Helm action
    needs: diff
    permissions: { pull-requests: write }
    # Download diff; post with gh.
```

**Passes:** the action stays within its limits.

</div>
</div>

<div class="takeaway">Same developer outcome. The Helm action no longer sits next to production credentials.</div>

<!--
Time: 1:45. Elapsed: 18:00.
Both snippets are excerpts, not complete runnable workflows. The full example is in Chapter 6 of the guide.
Use the catalog SHA with a version comment; checkout uses persist-credentials: false. All remote actions, including artifact helpers, are pinned.
The diff job runs the untrusted Helm action without secrets. The comment job downloads the artifact using the approved trusted helper and posts it with gh; it doesn't execute the Helm action.
The required policy check reads the protected catalog. It checks permissions and secret paths including env, with, secrets: inherit and toJSON(secrets). A failure links Ana to the approved snippet and request route.
Ana tests in a non-production workflow before other teams adopt it.
-->

---

<p class="section-label">Part 2 · Practice · 4 · Monitoring usage</p>

# October: the workflow passed, but a token leaked in

A workflow-level `env:` gave `NPM_TOKEN` to every job.
The policy check had only inspected job-level `env:`.

| Evidence | What it tells Marta and Sam |
|---|---|
| **Catalog** | This action is allowed no secrets |
| **Daily inventory** | Which repositories declare the action and SHA |
| **Job logs + audit log** | Which action ran and which secrets its job received |
| **Runner network logs** | Which outbound calls were blocked |

**Sam (Security)** flags the mismatch. Payments fixes the workflow that afternoon.
**Marta (Platform)** fixes the check the next day.

<div class="takeaway">Runtime evidence catches a mistake the pull-request check missed.</div>

<!--
Time: 1:15. Elapsed: 19:15.
Job logs list downloaded actions and SHAs, including nested actions. workflows.prepared_workflow_job lists the secrets the job received. Join by run; neither source alone answers both questions.
Security owns SIEM detections, Platform owns collection and inventory, and Infra sends blocked-egress logs. The shared channel gets the alert to the team that can act.
Don't introduce the separate December token-cloning drill here. It is another example in Chapter 7, not another storyline for this talk.
-->

---

<p class="section-label">Part 2 · Practice · 5 · Lifecycle management</p>

# November: a new version enters. The old one leaves.

<div class="cards">
<div>

### 5a · Version Updates

Ana requests **v2.6.0**.

A fresh review approves its SHA. The catalog gives v2.5.0 an expiry date.

</div>
<div>

### 5b · Vulnerability Management

Daily checks keep covering **both live SHAs**.

A security fix skips the wait, never the review.

</div>
<div>

### 5c · Deprecation and Removal

Teams get warnings and migration pull requests.

By February, nobody uses v2.5.0. It leaves the allowlist.

</div>
</div>

<div class="route">New approval → Migration window → Old SHA removed</div>

<!--
Time: 1:15. Elapsed: 20:30.
The new feature is OCI chart support. v2.6.0 is approved in three days; migration PRs follow a week after the release. The v2.5.0 SHA expires three months after the new approval.
High findings shorten the window to 30 days; Critical to the same day. Suspected compromise goes directly to Step 6.
Daily checks cover advisories for every live action, Dependabot on review copies and moved upstream tags. Keep at most two live SHAs per untrusted action.
Archive a retired SHA's review copy after 30 days. Expiry is a planned migration, not a surprise red build.
-->

---

<!-- _class: dense -->

<p class="section-label">Part 2 · Practice · 6 · Compromise response</p>

# February: the approved action turns malicious

**10:12** Egress alerts → **10:14** Shared incident → **10:41** Action blocked

A bundled dependency in v2.6.0 carried a dormant payload.
**The review missed it.**

| Fictional Contoso outcomes | March 2025 | February 2026 |
|---|---|---|
| Time to block | A day | **29 minutes** |
| Finding affected runs | Four days | **One query** |
| Credentials rotated | 250+ | **1** |
| What leaked | Source code, Azure access, customer data | **Some Helm charts** |

<div class="takeaway">Review reduced risk. Restricted access, egress controls, evidence and owners limited the damage.</div>

<!--
Time: 1:45. Elapsed: 22:15.
Sam sees blocked calls to an unknown domain. Fabrikam confirms the payload at 10:30. Platform blocks the action, cancels running jobs and uses run evidence to scope exposure since the SHA was approved, not just since the alert.
Self-hosted runners blocked all outbound calls. Two repositories used standard GitHub-hosted runners and leaked some Helm charts; their jobs had no secrets and only a job-scoped read-only token.
One other job received SONAR_TOKEN through a reusable workflow that the policy check didn't cover. It was rotated within the hour. The process isn't perfect, and the story shouldn't claim zero exposure.
Teams rotate exposed credentials and revoke old ones, and rebuild affected outputs. Security investigates; Infra owns network containment. The action's dormant payload was not caught by the review.
These are different fictional incidents, not a controlled benchmark. The comparison illustrates reduced blast radius, not a guarantee.
-->

---

<p class="section-label">Part 2 · Practice · 7 · Break-glass</p>

# Payments is blocked. The answer isn't "bypass it."

**Ana (Sponsor)** asks to use clean, reviewed **v2.5.0**, now expired.

<div class="cols">
<div class="route">

### A bounded exception

- Platform **and** Security approve.
- Payments repositories only.
- No secrets; egress-controlled runners.
- Seven days, with automatic removal.

</div>
<div class="route">

### A way back to normal

- Narrow the block to the bad SHA.
- Fabrikam releases clean **v2.6.1**.
- Skip the wait, not the review.
- Migrate; let the exception expire.

</div>
</div>

<div class="takeaway">Delivery resumes without allowing compromised or unreviewed code.</div>

<!--
Time: 1:45. Elapsed: 24:00.
The next morning, the required Helm diff check is red across Payments. Platform and Security approve the already-reviewed, clean v2.5.0 SHA for seven days.
Default break-glass is 72 hours, seven days at most; two approvals within four hours. Automation removes the entry on the date and the policy check restricts the repositories.
To allow the older SHA, replace the repository-wide deny with a deny for the bad v2.6.0 SHA, then add the scoped, temporary approval.
Fabrikam releases v2.6.1 three days later. As a security fix it skips the queue and age wait, but it still gets the full review. Payments migrates and the exception expires.
Never use break-glass for a compromised SHA or unreviewed code. Record the request, approvers and dates; review exceptions monthly.
Part 2 ends here. We now leave Contoso and address the audience directly.
-->

---

<!-- _footer: "Takeaways · From principles to practice" -->

# The framework, demonstrated

| Part 1: the principle group | Part 2: what it changed |
|---|---|
| **Approve what runs** | Ana gets a reviewed SHA through a published route |
| **Limit what it can reach** | The later payload doesn't inherit production credentials |
| **Keep managing it** | Old versions leave; evidence supports a fast, scoped response |

<div class="cards">
<div>

### Platform + dev teams

A usable path, with shared ownership

</div>
<div>

### Security + Infra team

Detection and containment, already connected

</div>
<div>

### All four teams

One channel, one process, rehearsed

</div>
</div>

<!--
Time: 1:00. Elapsed: 25:00.
This is the single deliberate recap linking the two parts. Don't replay the ten principles or the lifecycle table.
Principle 1 connects everything: speed is a security feature. A fast approved path reduces workarounds; recovery keeps it usable when something goes wrong.
The compromise still happened. The improvement was a smaller blast radius, quicker scoping and a safe route back to delivery.
-->

---

# Your first pilot: one organization, clear owners

| Start with | Make the result visible |
|---|---|
| **Name the four teams** | A shared channel and a policy owner |
| **Inventory the risky jobs first** | Actions, SHAs, nested code, secrets and runner access |
| **Seed a small catalog** | Approved SHAs, sponsors, access limits and request times |
| **Help teams migrate** | Pinning pull requests and useful policy messages |
| **Enforce and rehearse** | Warning mode first; then test blocking, scoping and recovery |

<div class="takeaway">Start small. Make the secure route work before making it mandatory.</div>

<!--
Time: 1:30. Elapsed: 26:30.
This is the audience's next step, not another telling of Contoso's bootstrap.
Use the settings checklist in backup for a gap assessment. Prioritize jobs holding secrets, and include Infra from day one.
Keep the normal request route and break-glass operational before enforcing. Announce each enforcement stage a month ahead.
The guide contains the full checklist, request form, schema, timing defaults and incident runbook.
-->

---

<!-- _class: lead -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# Principles first. Practice second.

**Approve what runs. Limit its access. Keep managing it.**

The full guide:
rmoreirao.github.io/securing-third-party-github-actions

**Questions?**

<span style="font-size:18px;color:#656d76">Contoso and Fabrikam are fictional. Licensed under CC BY 4.0.</span>

<!--
Time: 0:30. Elapsed: 27:00. Questions: 3:00, ending at 30:00.
Stop the main presentation here. Backup slides are reference material, not a third section of the talk.
The guide and this talk express personal views, not official Microsoft guidance.
-->

---

<!-- _class: dense -->
<!-- _footer: "Backup · Reference only" -->

# Backup: settings checklist (1/2)

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
Reference only; outside the 27-minute main presentation.
This and the next slide reproduce the guide's settings checklist item for item.
SHA pinning applies to all remote actions, internal ones too. Migrate before enabling it.
-->

---

<!-- _class: dense -->

# Backup: settings checklist (2/2)

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
Reference only. Standard GitHub-hosted runners have no egress control.
When third-party code needs sensitive data, use egress-controlled self-hosted runners, private networking, or written risk acceptance.
New network destinations are declared in the request, approved and recorded in the catalog, then applied by Infra or automation within the onboarding window. Use exact hostnames and scope rules to runner groups.
-->

---

<!-- _class: dense -->

# Backup: the allowlist in practice

| Entry | What it permits or blocks |
|---|---|
| `owner/repo@<full-sha>` | One approved SHA from an untrusted publisher |
| `owner/repo@*` | Every SHA of one repository from a trusted publisher |
| `!owner/repo@*` | A compromised action, whatever else allows it |
| `!owner/repo@<full-sha>` | One bad SHA from a trusted repository |

- **Avoid `owner/*`.** It includes repositories that don't exist yet.
- **A verified creator isn't a review.** Keep the Marketplace checkbox off.
- **1,000 entries is the hard limit.** Warn at 800.
- Generate from the catalog; keep at most two live SHAs per untrusted action.

<div class="takeaway">Trusted publishers save review work and entries, but allow unreviewed future SHAs.</div>

<!--
Reference only. Actions policy: Allow enterprise, and select non-enterprise, actions and reusable workflows, enforced at enterprise level.
Contoso chooses GitHub, Microsoft and Azure repositories. Per-repository entries give more control than the checkbox for every action created by GitHub, archived repositories included.
Pinning, daily monitoring and a fast deny entry still apply to trusted publishers.
-->

---

<!-- _footer: "" -->

![bg contain](../assets/third-party-actions-lifecycle.svg)

<!--
Reference only: the full lifecycle diagram from the guide. Main-story steps preserve its numbers 0–7 and the 2a/2b and 5a/5b/5c branches.
Use the PDF to zoom into the detail.
-->
