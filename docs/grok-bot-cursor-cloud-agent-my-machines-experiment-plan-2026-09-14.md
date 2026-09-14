# Experiment Setup Plan: Grok Bot → Cursor Cloud Agent → My Machines Worker

**Goal:** Verify, on a controlled repository, the Level-3 agentic pipeline discussed with Grok Bot as the outer/orchestrator agent, Cursor Cloud Agent as the coding agent, and a Cursor **My Machines** worker on the Windows laptop as the execution host.

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

### Billing safety

**Recommended initial setting:**

- [ ] **On-Demand Usage = OFF**

Purpose: ensure that when included usage is exhausted, the experiment stops rather than silently producing paid overage.

Cursor says on-demand usage must be explicitly enabled; when disabled, requests stop once included usage runs out.

Reference:
- https://prod.cursor.com/help/account-and-billing/overages

**Do not enable on-demand merely to make Cloud Agent work.** Cloud Agents are available on paid plans; on-demand is for usage beyond the included allowance.

### Cloud Agent model

- [ ] Open Cursor Cloud Agent settings.
- [ ] **Pin an explicit default Cloud Agent model before the first run.**
- [ ] Do not rely on a changing/default routing choice for this experiment.
- [ ] Use the model actually exposed in your current Cloud Agent UI. Examples may include **Composer Fast** or **Grok High**.
- [ ] Record the exact model/variant shown in the UI.

Cursor documents a Cloud Agent **Default model** setting: the selected model is used when a run does not specify one.

Reference:
- https://prod.cursor.com/docs/cloud-agent/settings

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
api2.cursor.sh
api2direct.cursor.sh
cloud-agent-artifacts.s3.us-east-1.amazonaws.com
```

The exact artifact host is needed for artifact uploads; the agent can otherwise continue operating if artifact upload is blocked, but screenshots/log references may be unavailable in PRs/dashboard.

Reference:
- https://cursor.com/docs/cloud-agent/self-hosted
- https://cursor.com/docs/cloud-agent/self-hosted/my-machines

### Corporate-network check

Because this is a corporate Windows laptop/network:

- [ ] Verify outbound HTTPS access to the required Cursor hosts.
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

### First Grok task

Keep the first task tiny and easy to verify.

Example:

> Create a new file named `grok_cloud_worker_test.txt` in the test repository. Put the machine hostname, the current Git branch, and a one-line description of what you changed. Run a simple repository test afterwards and report the result.

The objective is not useful software yet. The objective is to prove the entire chain.

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
- [ ] Cloud Agent performed reasoning and issued tool calls.
- [ ] Cloud Agent was associated with the intended repository.

### Layer C — My Machines worker

- [ ] `my-windows-laptop` was selected/used.
- [ ] Files changed on the physical laptop.
- [ ] Shell commands executed on the laptop.
- [ ] Test results came from the laptop environment.

### Layer D — Final feedback loop

Test one deliberate failure.

For example, tell the agent to run a command that should fail and then recover.

Expected pattern:

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

This demonstrates the actual Level-3 feedback loop rather than merely remote task submission.

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
- https://prod.cursor.com/help/account-and-billing/overages
- https://cursor.com/docs/cloud-agent

---

## 10. Optional second phase: model comparison

Once the pipeline works, repeat the same task with:

1. **Composer Fast**
2. **Grok High**

Keep the task and repository state as similar as possible.

Record:

| Metric | Composer Fast | Grok High |
|---|---:|---:|
| Task completed | | |
| Time to completion | | |
| Number of retries | | |
| Tests passed | | |
| Human intervention | | |
| Included usage consumed | | |
| Output quality | | |

This separates two questions that should not be mixed:

- Does the **Level-3 architecture** work?
- Which **coding model** performs best inside it?

---

## 11. Optional third phase: test the value of Grok as manager

Compare:

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

## 12. Expected final architecture

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

## 13. Success criteria

The experiment is successful only if all of these are demonstrated:

- [ ] Pro account can create a Cloud Agent.
- [ ] A specific Cloud Agent model/variant is pinned.
- [ ] On-Demand Usage remains OFF.
- [ ] Windows laptop is registered as a My Machines worker.
- [ ] Cursor can run a Cloud Agent directly on that worker.
- [ ] The worker executes commands/files/tests locally.
- [ ] Grok Bot can create/manage the Cursor Cloud Agent.
- [ ] Grok Bot can cause useful work to be completed through that Cloud Agent on the worker.
- [ ] Grok Bot can observe results and perform at least one follow-up/recovery cycle.
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
- Cloud Agent Settings: https://prod.cursor.com/docs/cloud-agent/settings
- Models & Pricing: https://cursor.com/docs/models-and-pricing
- Usage-based charges: https://prod.cursor.com/help/account-and-billing/overages
- Spend limits: https://prod.cursor.com/help/account-and-billing/spend-limits

**Documentation status:** checked against the current pages on September 14, 2026.
