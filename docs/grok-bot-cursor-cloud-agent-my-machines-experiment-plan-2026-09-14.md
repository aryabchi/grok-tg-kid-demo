# Experiment Setup Plan: Grok Bot → Cursor Cloud Agent → My Machines Worker

*Corrected version — 2026-09-15.*

**Goal:** Verify, on a controlled repository, the Level-3 agentic pipeline discussed with Grok Bot as the outer/orchestrator agent, Cursor Cloud Agent as the coding agent, and a Cursor **My Machines** worker on the Windows laptop as the execution host.

## Revision notes (2026-09-15)

Changes from the 2026-09-14 draft, based on a documentation review against current Cursor and xAI sources:

- Removed the optional model-comparison phase that was §10 — out of scope for now. Section numbers from the old §11 onward have shifted up by one.
- §6's first Grok Bot task now explicitly names the target worker, repository, and branch, and asks Grok Bot to self-report which environment it actually used. Cursor's own My Machines docs only document `worker=`/`machine=` targeting for Slack, GitHub, and Linear triggers — Grok Bot isn't in that list — so targeting the right worker can't be assumed to happen implicitly.
- §7 Layer D is now split into a **success path** and a **retry path**, verified separately, instead of one deliberate-failure test.
- §3's networking checklist now includes `downloads.cursor.com`, a third required host Cursor documents that the original draft omitted.
- §1 and §7 flag a documented reliability concern: recent Cursor community-forum reports describe Cloud Agents — including ones spawned by Grok Bot — not respecting a pinned default model/speed setting. Added a step to verify the actually-billed model rather than trusting the configured default.
- Reference links updated from `prod.cursor.com` to the canonical `cursor.com` — no public documentation confirms `prod.cursor.com` as a stable public alias.
- §10 (test the value of Grok as manager) is left unchanged pending further discussion — it still needs a concrete test design before it's actionable.

**Reference architecture**

```text
You
  │
  ▼
Grok Bot
  │  create/manage
  ▼
Cursor Cloud Agent
  │  agent loop, inference, planning
  │
  ▼
Cursor My Machines worker
  │  file edits, shell commands, tests, local tools/MCP
  ▼
Your Windows laptop / selected repo
```

The key separation is:
- **Grok Bot:** outer manager/orchestrator.
- **Cursor Cloud Agent:** coding-agent loop.
- **My Machines worker:** local execution environment.

---

## 1. Preconditions

### Account / plan

- [ ] Cursor account is on **Pro**.
- [ ] Grok Bot is available on the same Cursor account.
- [ ] The target repository is accessible to the Cursor account.
- [ ] Source-control integration required by Cloud Agent is configured (GitHub, GitLab, Azure DevOps, or Bitbucket as applicable).
- [ ] Use a **test repository or disposable branch** for the first experiment.

Cursor currently lists **Cloud Agents** and **Grok Bot access** on Pro. Pro also includes the Cursor Models pool containing Cursor Grok and Composer models.
References:
- https://cursor.com/pricing
- https://cursor.com/docs/models-and-pricing
- https://cursor.com/docs/grok-bot

### Billing safety

**Recommended initial setting:**

- [ ] **On-Demand Usage = OFF**

Purpose: ensure that when included usage is exhausted, the experiment stops rather than silently producing paid overage.

Cursor says on-demand usage must be explicitly enabled; when disabled, requests stop once included usage runs out.

Reference:
- https://cursor.com/help/account-and-billing/overages

**Do not enable on-demand merely to make Cloud Agent work.** Cloud Agents are available on paid plans; on-demand is for usage beyond the included allowance.

Cursor also prompts you to set a spend limit the first time you use Cloud Agents at all — treat that prompt as part of this step, not a separate one.

### Cloud Agent model

- [ ] Open Cursor Cloud Agent settings.
- [ ] **Pin an explicit default Cloud Agent model before the first run.**
- [ ] Do not rely on a changing/default routing choice for this experiment.
- [ ] Use the model actually exposed in your current Cloud Agent UI. Examples may include **Composer Fast** or **Grok High**.
- [ ] Record the exact model/variant shown in the UI.
- [ ] **After each run, verify the actually-billed model in the Spending dashboard or run transcript — do not assume the pinned default was honored.**

Cursor documents a Cloud Agent **Default model** setting: the selected model is used when a run does not specify one.

⚠️ **Known reliability concern:** recent Cursor community-forum bug reports (Aug–Sep 2026) describe Cloud Agent runs not respecting the configured default model or Fast/effort setting — including at least one report specifically about Cloud Agents spawned by Grok Bot running in Fast mode despite a non-Fast default. These are unconfirmed forum reports, not acknowledged Cursor bugs, but given this experiment depends on knowing exactly which model ran, don't skip the verification step above.

Reference:
- https://cursor.com/docs/cloud-agent/settings

### Context window

- [ ] Leave context-window size at the default initially unless you have a reason to change it.
- [ ] Record the value used.

Cursor notes that a larger context window can increase token usage/cost.

Reference:
- https://cursor.com/docs/cloud-agent

---

## 2. Prepare the worker host

The host for this experiment is the **Windows laptop**.

### Important terminology

Do **not** confuse:

- your existing interactive **Cursor IDE Agent**, and
- the **Cursor My Machines worker**.

Your laptop can have both:

```text
Windows laptop
├── Cursor IDE
│   └── your normal local Agent
│
└── Cursor CLI
    └── My Machines worker
```

The My Machines worker is the execution endpoint for Cloud Agent tool calls. It is not another AI model.

Cursor says the Cloud Agent loop/inference/planning runs in Cursor's cloud, while the worker performs file edits, terminal commands, browser actions, and local MCP operations.

Reference:
- https://cursor.com/docs/cloud-agent/self-hosted
- https://cursor.com/docs/cloud-agent/self-hosted/my-machines

### Install Cursor CLI

On Windows PowerShell, Cursor currently documents:

```powershell
irm 'https://cursor.com/install?win32=true' | iex
```

Verify:

```powershell
agent --version
```

Reference:
- https://cursor.com/docs/cloud-agent/self-hosted/my-machines

### Sign in

For a personal My Machines worker:

```powershell
agent login
```

Use the same personal Cursor account that owns the Pro subscription.

Cursor also documents a personal user API key as an alternative, but browser login is the simplest first experiment.

### Start and name the worker

Use an explicit name so the execution environment is unambiguous:

```powershell
agent worker start --name "my-windows-laptop"
```

Keep the worker process running during the experiment.

Cursor describes My Machines workers as long-lived and reusable for future Cloud Agent sessions.

### Register the repository

Prefer registering the exact checkout used by the experiment.

Example:

```powershell
agent worker start `
  --name "my-windows-laptop" `
  --worker-dir "C:\path\to\test-repo"
```

⚠️ Cursor's own examples show `--worker-dir` placed both after `start` (single directory) and before `start` (when the flag is repeated for multiple directories) — the combined `--name` + `--worker-dir` form above isn't shown verbatim in the docs. Run `agent worker start --help` once to confirm accepted flag order before the real attempt.

The repo should have a valid Git remote. Cursor uses the worker's repository metadata when matching requests to a checkout.

If the machine later serves multiple repositories, register each checkout explicitly.

---

## 3. Verify worker networking before involving Grok Bot

The worker establishes the connection **outbound** to Cursor.

Expected network model:

```text
Windows laptop
      │
      │ outbound HTTPS
      ▼
Cursor
```

No inbound port, public IP, or inbound VPN tunnel is required for the worker connection.

Cursor currently lists these worker endpoints:

```text
api2.cursor.sh                                    — agent session
api2direct.cursor.sh                               — agent session
downloads.cursor.com                               — CLI updates, first-time Computer Use install
cloud-agent-artifacts.s3.us-east-1.amazonaws.com   — artifact uploads
```

`downloads.cursor.com` was missing from the original host list — add it to the allowlist check. If blocked, the worker session itself keeps working, but CLI auto-updates fail silently, which can be confusing to debug later on a corporate network.

The exact artifact host is needed for artifact uploads; the agent can otherwise continue operating if artifact upload is blocked, but screenshots/log references may be unavailable in PRs/dashboard.

Reference:
- https://cursor.com/docs/cloud-agent/self-hosted
- https://cursor.com/docs/cloud-agent/self-hosted/my-machines

### Corporate-network check

Because this is a corporate Windows laptop/network:

- [ ] Verify outbound HTTPS access to all four required Cursor hosts listed above.
- [ ] Verify whether an HTTP(S) proxy is required.
- [ ] If a proxy is required, configure `HTTPS_PROXY` / `https_proxy` in the worker environment.
- [ ] Do not open inbound firewall ports for the worker.

---

## 4. Verify My Machines independently of Grok Bot

Before testing Grok, prove that Cursor Cloud Agent can use the worker **by itself**.

Go to:

**cursor.com/agents**

Then:

1. [ ] Confirm the machine appears in the environment/run-on selector.
2. [ ] Select `my-windows-laptop`.
3. [ ] Use the test repository.
4. [ ] Run a tiny task.

Suggested task:

> In this test repository, create a file named `cloud_agent_worker_test.txt` containing the current hostname and a short sentence saying this file was created by a Cursor Cloud Agent running on the My Machines worker. Then show the terminal command used.

Expected result:

```text
Cursor Cloud Agent
        │
        ▼
My Machines worker
        │
        ▼
Windows filesystem changes
```

Verify on the laptop that the file was actually created locally.

This step isolates **Cursor Cloud Agent ↔ worker** from the Grok Bot integration.

---

## 5. Verify that the agent really executed locally

Do not rely only on the final textual answer.

Have the Cloud Agent perform something whose result proves local execution.

Good proof signals:

- [ ] The created file is physically present in the Windows checkout.
- [ ] `hostname` or another local machine identifier matches the laptop.
- [ ] A locally installed tool is available to the agent.
- [ ] A local Python environment or project command executes successfully.
- [ ] A local-only path/resource is accessed.

Avoid using credentials or confidential files just to prove locality.

Example:

```powershell
hostname
python --version
git remote -v
```

The point is to establish:

> Cloud Agent reasoning happens in Cursor's cloud, but the tool action actually happens on this Windows worker.

---

## 6. Only after that: bring Grok Bot into the experiment

Once the direct Cursor path works:

```text
Cursor Cloud Agent
       │
       ▼
My Machines worker
```

test:

```text
Grok Bot
   │
   ▼
Cursor Cloud Agent
   │
   ▼
My Machines worker
```

xAI's current engineering guide explicitly describes Grok Bot creating/managing Cursor Cloud Agents and says Grok Bot can start cloud agents on users' own worker machines, including private Cursor workers.

Reference:
- https://x.ai/bot/guides/grok-bot-for-engineering
- https://cursor.com/docs/grok-bot/teams

### First Grok task

Keep the first task tiny and easy to verify — but make every part of the execution target explicit. Cursor's own docs only document `worker=`/`machine=` targeting for Slack, GitHub, and Linear triggers, not for Grok Bot, so don't leave the target implicit and hope Grok Bot infers it correctly.

> Using Cursor Cloud Agent, start a run on my registered My Machines worker named `my-windows-laptop` — not a Cursor-hosted VM. Target repository `<git remote URL>`, branch `<branch name>`. In that checkout, create a file named `grok_cloud_worker_test.txt` containing: (1) the machine hostname, (2) the current git branch, (3) the git remote URL, (4) a one-line description of what you changed. Then run `<test command>` and report its exact output. Before you finish, explicitly confirm which worker/environment the Cloud Agent actually executed on.

Fill in `<git remote URL>`, `<branch name>`, and `<test command>` with the values from your test repository before running this.

The objective is not useful software yet. The objective is to prove the entire chain, including that Grok Bot actually targeted the named worker rather than defaulting to a Cursor-hosted VM.

---

## 7. Verify the complete chain

Collect evidence at each layer.

### Layer A — Grok Bot

- [ ] Grok Bot accepted the engineering task.
- [ ] Grok Bot created/managed a Cursor Cloud Agent.
- [ ] Grok Bot could inspect the Cloud Agent result/transcript/artifacts.
- [ ] Grok Bot could issue a follow-up if needed.

### Layer B — Cursor Cloud Agent

- [ ] Cloud Agent run exists in Cursor.
- [ ] Exact model/variant is visible/recorded.
- [ ] **Recorded model/variant matches the pinned default** — cross-checked against the Spending dashboard or run transcript, not just the settings page (see §1's known reliability concern).
- [ ] Cloud Agent performed reasoning and issued tool calls.
- [ ] Cloud Agent was associated with the intended repository.

### Layer C — My Machines worker

- [ ] `my-windows-laptop` was selected/used — confirmed both by Grok Bot's self-report (per §6's task prompt) and independently in the Cursor dashboard's run details.
- [ ] Files changed on the physical laptop.
- [ ] Shell commands executed on the laptop.
- [ ] Test results came from the laptop environment.

### Layer D — Final feedback loop

Test the success path and the retry path separately. A system that's simply eager to declare victory could pass a single naive test — you want evidence it correctly recognizes *both* "this succeeded" and "this needs fixing."

#### Success path

- [ ] Task has an unambiguous, checkable success criterion.
- [ ] Cloud Agent completes on the first attempt, with no errors.
- [ ] Grok Bot verifies success against the stated proof requirement — not just "the agent said it's done."
- [ ] Grok Bot reports completion with proof (output, diff, or screenshot).
- [ ] Grok Bot does **not** trigger unnecessary retries or follow-ups on a run that already succeeded.

#### Retry path (deliberate failure)

- [ ] Task is designed to deterministically fail on the first attempt — reproducible, not flaky.
- [ ] First attempt fails as expected.
- [ ] Failure output reaches Grok Bot (transcript/artifacts).
- [ ] Grok Bot correctly identifies it as a failure, not a success.
- [ ] Grok Bot reasons about the cause and sends a correction without you intervening.
- [ ] Retry succeeds.
- [ ] Grok Bot confirms the retry against the original proof requirement.
- [ ] Record: number of retries, whether Grok Bot ever gave up and handed back to you, and total time.

Expected pattern for the retry path:

```text
Grok
  ↓
Cursor Agent
  ↓
run command on worker
  ↓
failure
  ↓
observe output
  ↓
reason
  ↓
fix / retry
  ↓
success
  ↓
Grok observes proof
```

This demonstrates the actual Level-3 feedback loop rather than merely remote task submission — and, tested against the success path too, shows the loop isn't just biased toward reporting success.

---

## 8. Recommended safety constraints for the first experiment

- [ ] Test repository only.
- [ ] No production credentials.
- [ ] No write access to important repositories.
- [ ] No destructive shell commands.
- [ ] No broad filesystem access unless required.
- [ ] Keep On-Demand Usage **OFF**.
- [ ] Use a small, explicit task.
- [ ] Pin a specific Cloud Agent model/variant.
- [ ] Verify the actually-billed model/variant after each run rather than trusting the pinned setting.
- [ ] Record the exact model, worker name, repo, branch, and result.

---

## 9. Cost controls and interpretation

Cursor currently describes Cloud Agent usage as being charged according to the selected model's API pricing. However, Pro includes a **Cursor Models usage pool** that covers first-party Cursor models such as Grok 4.6, Grok 4.5, and Composer 2.5.

Therefore distinguish:

```text
Included Pro usage
    ↓
used by Cloud Agent
    ↓
included pool decreases

When included usage is exhausted:
    │
    ├── On-Demand OFF → stop
    │
    └── On-Demand ON → additional paid usage
```

For first-party Cursor models, the relevant usage is in the Cursor Models pool. On-demand charges apply only after included usage is exhausted and on-demand is explicitly enabled.

References:
- https://cursor.com/docs/models-and-pricing
- https://cursor.com/help/account-and-billing/overages
- https://cursor.com/docs/cloud-agent

---

## 10. Optional phase: test the value of Grok as manager

*(Unchanged pending further discussion — needs a concrete test design before this is actionable.)*

Once the pipeline works, compare:

### A. Direct Cloud Agent

```text
You → Cursor Cloud Agent → My Machines
```

### B. Level 3

```text
You → Grok Bot → Cursor Cloud Agent → My Machines
```

Use the same task.

Measure whether Grok adds value through:

- better task decomposition
- better context gathering
- better verification
- useful follow-ups
- recovery from failures
- reduced human supervision

This is the experiment that actually tests whether the **agent-managing-an-agent** architecture is useful for you, rather than merely technically possible.

---

## 11. Expected final architecture

After successful setup:

```text
                         YOU
                          │
                          ▼
                    🤖 GROK BOT
                    outer agent
                          │
                          │ creates/manages
                          ▼
                 ☁️ CURSOR CLOUD AGENT
                    inner coding agent
                          │
             reasoning / planning / tool calls
                          │
                          ▼
                🖥️ MY MACHINES WORKER
                   "my-windows-laptop"
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
          repo files    terminal      tests
             │            │            │
             └────────────┼────────────┘
                          ▼
                       results
                          │
                          ▼
                 Cursor Cloud Agent
                          │
                          ▼
                      Grok Bot
                          │
                          ▼
                         YOU
```

### Critical conceptual point

The worker is **not** the existing interactive Cursor IDE Agent.

It is a **Cursor worker process on the laptop** that provides the execution environment for a **Cloud Agent**.

Your existing Cursor IDE can remain unchanged.

---

## 12. Success criteria

The experiment is successful only if all of these are demonstrated:

- [ ] Pro account can create a Cloud Agent.
- [ ] A specific Cloud Agent model/variant is pinned, and confirmed via the Spending dashboard/transcript rather than assumed.
- [ ] On-Demand Usage remains OFF.
- [ ] Windows laptop is registered as a My Machines worker.
- [ ] Cursor can run a Cloud Agent directly on that worker.
- [ ] The worker executes commands/files/tests locally.
- [ ] Grok Bot can create/manage the Cursor Cloud Agent.
- [ ] Grok Bot can target the named worker explicitly and confirms in its own report that it did so.
- [ ] Grok Bot can cause useful work to be completed through that Cloud Agent on the worker.
- [ ] Grok Bot can observe results, correctly distinguish success from failure, and perform at least one follow-up/recovery cycle when needed.
- [ ] No inbound connection to the laptop is required.
- [ ] No unexpected paid usage occurs.

---

## Current documentation to verify independently

**xAI**
- Grok Bot for Engineering: https://x.ai/bot/guides/grok-bot-for-engineering

**Cursor**
- Cloud Agents: https://cursor.com/docs/cloud-agent
- My Machines: https://cursor.com/docs/cloud-agent/self-hosted/my-machines
- Self-Hosted Machines: https://cursor.com/docs/cloud-agent/self-hosted
- Cloud Agent Settings: https://cursor.com/docs/cloud-agent/settings
- Models & Pricing: https://cursor.com/docs/models-and-pricing
- Grok Bot overview: https://cursor.com/docs/grok-bot
- Grok Bot for Teams (Cloud Agent delegation toggle): https://cursor.com/docs/grok-bot/teams
- Usage-based charges: https://cursor.com/help/account-and-billing/overages
- Spend limits: https://cursor.com/help/account-and-billing/spend-limits

**Documentation status:** original draft checked against Cursor/xAI pages on September 14, 2026; corrections in this revision checked against current pages on September 15, 2026.
