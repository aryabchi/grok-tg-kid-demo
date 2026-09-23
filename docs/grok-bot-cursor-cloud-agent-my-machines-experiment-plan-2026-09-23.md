# Experiment Setup Plan: Grok Bot → Cursor Cloud Agent → My Machines Worker

*Corrected version — 2026-09-23. Supersedes the 2026-09-15 revision of the 2026-09-14 draft. The earlier file is unchanged.*

**Goal:** Verify, on a controlled repository, the Level-3 agentic pipeline discussed with Grok Bot as the outer/orchestrator agent, Cursor Cloud Agent as the coding agent, and a Cursor **My Machines** worker on the Windows laptop as the execution host.

## Revision notes (2026-09-23)

Checked against current Cursor and xAI docs on 2026-09-23. Changes from the 2026-09-15 revision:

- The worker edits `--worker-dir` in place and can commit or push with the laptop's git credentials. The experiment now requires a dedicated clone, a disposable branch, a clean tree, and an explicit ban on commit, push, and pull requests. Do not register `d:\Data\grok-tg-kid-demo` (remote `git@github.com:user/grok-tg-kid-demo.git`, branch `main`).
- Proof of the chain is a Cursor dashboard run whose environment is `my-windows-laptop`, plus the file on disk in that clone. A file on the laptop, or Grok Bot's own report, is not enough. Grok Bot can also finish work on its own cloud computer or through Grok Bot local execution.
- `worker=` / `machine=` remains documented only for Slack, GitHub, and Linear. Chat targeting from Grok Bot is unverified. Routing matches the worker's registered git remote. This laptop's remotes are SSH (`git@<host>:<path>.git`). Prompts must paste that string from `git remote -v`, not the HTTPS URL from a browser or from Cursor's repo picker.
- The Cloud Agent default model applies only when the spawn does not name a model. Grok Bot may pass its own. Record the pinned setting, the model the spawn requested, and the model on the usage page.
- Model names updated to the current Cursor Models pool: Grok 4.7, Grok 4.6, Grok 4.5, Composer 2.5. Fast variants are a separate, higher price. Pin a non-Fast first-party model.
- Computer use and desktop sharing are macOS and Linux only. This Windows worker cannot produce screenshot proof. Do not pass `--computer-use`.
- Usage is two meters: Cloud Agent draws the monthly Cursor Models pool; Grok Bot has its own allowance, resetting weekly, with no separate spend cap. Snapshot both before the first run and after each run. The first-run Cloud Agent spend limit is a third control, distinct from On-Demand Usage.
- Added `agent worker debug` before any agent task, a privacy-mode and Cloud Agent delegation check, a laptop-awake check, and a TLS-inspection note separate from `HTTPS_PROXY`.
- §7 now specifies the success-path task and a deterministic retry-path task.
- §10 states the controls required before a manager-value comparison means anything.
- §2 now walks through worker-host preparation on this laptop. The dedicated clone is created with the SSH remote you already use (`git@<host>:<path>.git`). That exact string is what the worker registers and what later prompts must name. Do not substitute the HTTPS URL from a browser or from Cursor's repo picker.

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
Dedicated clone on the Windows laptop
```

The key separation is:

- **Grok Bot:** outer manager/orchestrator. It also has its own persistent cloud computer, and optionally a local-execution path through the desktop app. Neither of those is the worker.
- **Cursor Cloud Agent:** coding-agent loop. It can run on a Cursor-hosted VM or on a named worker.
- **My Machines worker:** local execution environment for Cloud Agent tool calls.

Four surfaces can "succeed" at a file-creating task. Only the last one is the experiment:


| Surface                   | Where the work happens                                   |
| ------------------------- | -------------------------------------------------------- |
| Grok Bot cloud computer   | Persistent cloud VM with its own filesystem and terminal |
| Grok Bot local execution  | This machine, through the Grok Bot desktop app           |
| Cursor-hosted Cloud Agent | A Cursor VM                                              |
| My Machines worker        | This laptop, via `agent worker`                          |


---



## 1. Preconditions



### Account / plan

- [ ] Cursor account is on **Pro** (or another paid individual plan that includes Cloud Agents and Grok Bot).
- [ ] Grok Bot is available on the same Cursor account, and the bot's Cursor connection is authorized.
- [ ] Privacy mode is not **Privacy Mode (Legacy)**. That setting blocks Grok Bot entirely.
- [ ] On a team account, Grok Bot → Cloud Agents delegation is allowed. The switch lives on the Grok Bot admin page and is on by default. Personal Pro has no such page.
- [ ] The target repository is accessible to the Cursor account.
- [ ] Source-control integration required by Cloud Agent is configured (GitHub, GitLab, Azure DevOps, or Bitbucket as applicable).
- [ ] Use a **dedicated clone of a test repository, on a disposable branch**. Do not register the `grok-tg-kid-demo` working tree.

Cursor currently lists **Cloud Agents** and **Grok Bot access** on every paid individual plan. Pro includes the Cursor Models pool.

References:

- [https://cursor.com/pricing](https://cursor.com/pricing)
- [https://cursor.com/docs/models-and-pricing](https://cursor.com/docs/models-and-pricing)
- [https://cursor.com/docs/grok-bot](https://cursor.com/docs/grok-bot)
- [https://cursor.com/docs/grok-bot/teams](https://cursor.com/docs/grok-bot/teams)



### Billing safety

**Recommended initial setting:**

- [ ] **On-Demand Usage = OFF**
- [ ] When the first Cloud Agent run asks for a spend limit, set a small one and record the number. That limit is separate from the on-demand switch.

Purpose: when included usage is exhausted, the experiment stops rather than silently producing paid overage.

There are two meters:

- **Cloud Agent** draws the monthly **Cursor Models** pool (or the Other Models pool, if a third-party model is selected).
- **Grok Bot** has its own included usage, resetting weekly. Cursor documents no separate Grok Bot spend cap; account-level on-demand controls apply.

- [ ] Snapshot both usage pages before the first run, and again after each run.
- [ ] Do not enable on-demand merely to make Cloud Agent work.

References:

- [https://cursor.com/help/account-and-billing/overages](https://cursor.com/help/account-and-billing/overages)
- [https://cursor.com/docs/models-and-pricing](https://cursor.com/docs/models-and-pricing)
- [https://cursor.com/docs/grok-bot](https://cursor.com/docs/grok-bot)



### Cloud Agent model

- [ ] Open Cursor Cloud Agent settings.
- [ ] **Pin an explicit non-Fast first-party model** before the first run. Current pool: **Grok 4.7**, **Grok 4.6**, **Grok 4.5**, **Composer 2.5**. Composer 2.5 is the cheapest of these.
- [ ] Do not pin a Fast variant. Fast is a separate price (Composer 2.5 Fast is several times Composer 2.5; Grok Fast is 2×). Grok 4.7 input above 256k tokens is 2×.
- [ ] Record the exact model/variant shown in the UI.
- [ ] Remember the pin applies only when a run does not name a model. Grok Bot may pass its own model when it spawns the agent.
- [ ] **After each run, record three values:** the pinned default, the model the spawn requested (transcript), and the model on the Spending / usage page. Do not assume they match.

Cursor documents a Cloud Agent **Default model** setting: the selected model is used when a run does not specify one.

Known reliability concern from August–September 2026 forum reports: some Cloud Agent runs, including ones spawned by Grok Bot, have not honored the configured default or Fast/effort setting. Unconfirmed, and worth the three-way check above.

Reference:

- [https://cursor.com/docs/cloud-agent/settings](https://cursor.com/docs/cloud-agent/settings)
- [https://cursor.com/docs/models-and-pricing](https://cursor.com/docs/models-and-pricing)



### Context window

- [ ] Leave context-window size at the default initially unless you have a reason to change it.
- [ ] Record the value used.

A larger context window increases token usage. On Grok 4.7, input past 256k is billed at 2×.

Reference:

- [https://cursor.com/docs/cloud-agent](https://cursor.com/docs/cloud-agent)

---



## 2. Prepare the worker host

The host for this experiment is the **Windows laptop**. Do the subsections in order. Stop at the first one that fails.

1. Read the terminology so the worker is not confused with the IDE agent or with Grok Bot.
2. Create the dedicated SSH clone and record its `git@` remote.
3. Install the Cursor CLI and confirm `agent` is on `PATH` in a new PowerShell window.
4. Sign in as the same personal Cursor user that owns the Pro plan.
5. Start the named worker against that clone, from a shell where `ssh` to the git host already works.
6. Leave that window open and keep the laptop awake.

The corporate repo is reached by SSH. Every clone, remote check, and prompt in this experiment uses the `git@<host>:<path>.git` URL. HTTPS is a different remote string, and Cursor matches the worker to the remote stored in the checkout.

### Important terminology

Do **not** confuse these four things:

- the interactive **Cursor IDE Agent** in the editor you have open,
- **Grok Bot's cloud computer**,
- **Grok Bot local execution** (desktop app, its own approval setting),
- the **Cursor My Machines worker** (`agent worker`).

```text
Windows laptop
├── Cursor IDE
│   └── your normal local Agent
├── Grok Bot desktop app
│   └── optional local execution (not the worker)
└── Cursor CLI
    └── My Machines worker
```

Grok Bot's own computer stays in Cursor's cloud and keeps running when the laptop is closed. The My Machines worker does not.

The worker is the execution endpoint for Cloud Agent tool calls. It is not another AI model. The agent loop, inference, and planning stay in Cursor's cloud. The worker performs file edits, terminal commands, and local MCP operations. Browser and computer-use actions require macOS or Linux, so they are out of scope on this laptop.

References:

- [https://cursor.com/docs/cloud-agent/self-hosted](https://cursor.com/docs/cloud-agent/self-hosted)
- [https://cursor.com/docs/cloud-agent/self-hosted/my-machines](https://cursor.com/docs/cloud-agent/self-hosted/my-machines)
- [https://cursor.com/docs/cloud-agent/self-hosted/computer-use](https://cursor.com/docs/cloud-agent/self-hosted/computer-use)



### Dedicated clone

Create a second checkout used only for this experiment. Do not point `--worker-dir` at `d:\Data\grok-tg-kid-demo`. That folder is the IDE workspace on `main`.

The remote must stay in the SSH form you already use to reach the corporate repo:

```text
git@<host>:<path>.git
```

Examples of that shape: `git@github.com:user/grok-tg-kid-demo.git`, or a corporate GitLab host such as `git@gitlab.example.com:group/repo.git`. Copy the URL from an existing working checkout (`git remote get-url origin`) or from the host's **SSH** clone field. Do not use the HTTPS clone URL, and do not run `git remote set-url` to rewrite `git@` into `https://`.

#### Confirm SSH before cloning

In the same PowerShell window you will later use to start the worker:

```powershell
ssh -V
ssh -T git@<host>
```

`ssh -T` should authenticate. A permission or host-key failure here will also break any later `git fetch` the agent runs inside the clone. Fix it before cloning.

On this laptop that means:

- Windows OpenSSH (`ssh.exe`), or the Git-for-Windows SSH you already use for this repo. `git` and `ssh` in this window must be the pair that already works.
- The private key is available to this user, not only to an elevated prompt and not only to Git Credential Manager. If the key has a passphrase, load it into `ssh-agent` in this same window and confirm `ssh -T` works after that. The worker process inherits this window's environment. A key loaded in a different terminal is invisible to the worker.
- If the git host is reachable only on the corporate VPN, connect the VPN first. The worker runs git on the laptop, so it needs the same path to `git@<host>` that you do. Cursor's cloud does not open that SSH session for you.
- Stay in the normal user account. An Administrator PowerShell has a different profile, a different `~\.ssh`, and often a different agent.



#### Clone and branch

Pick a path outside the IDE workspace. Create the parent directory first.

```powershell
New-Item -ItemType Directory -Force -Path C:\cursor-worker | Out-Null
cd C:\cursor-worker

git clone git@<host>:<path>.git experiment-clone
cd .\experiment-clone

git remote get-url origin
```

`git remote get-url origin` must print a `git@` URL. If it prints `https://`, remove the directory and clone again from the SSH URL.

Then make the disposable branch from the current default branch. Do not experiment on `main`.

```powershell
git status
git switch -c experiment/my-machines-2026-09-23
git status
git rev-parse HEAD
git branch --show-current
git remote -v
```

Checklist:

- [ ] Clone path is not `d:\Data\grok-tg-kid-demo`.
- [ ] `git remote get-url origin` starts with `git@`.
- [ ] Branch is `experiment/my-machines-2026-09-23` (or another name you choose). It is not `main`.
- [ ] `git status` is clean.
- [ ] Write down `HEAD`, the branch name, and the full `git@` remote, including the `origin` line from `git remote -v`. Later prompts paste that `git@...` string unchanged. The HTTPS URL shown in a browser or in Cursor's repository picker is a different string and will not match this worker.

The worker uses the laptop's existing SSH key. A commit or push from the agent is a push as you, over that same `git@` remote. The experiment prompts forbid both.

### Install Cursor CLI

Use a normal (non-elevated) PowerShell. Cursor currently documents:

```powershell
irm 'https://cursor.com/install?win32=true' | iex
```

Close that window and open a new one so the installer's `PATH` change is visible. Then:

```powershell
Get-Command agent | Format-List Source,Version
agent --version
```

`Get-Command agent` should resolve to the Cursor CLI, not to some other `agent` executable. If PowerShell says the command is missing, the new window did not pick up `PATH`. Open another window before continuing. Do not start the worker from the install window.

Reference:

- [https://cursor.com/docs/cloud-agent/self-hosted/my-machines](https://cursor.com/docs/cloud-agent/self-hosted/my-machines)



### Sign in

For a personal My Machines worker, in that same new window:

```powershell
agent login
```

A browser opens. Use the same personal Cursor account that owns the Pro subscription. Browser login is the simplest first experiment. Leave the window signed in. The worker started later reuses this login.

Service-account, team Admin, and organization API keys cannot start a My Machines worker. A personal user API key can, but do not put it on the command line for this experiment.

Confirm the CLI account before starting the worker. `agent worker debug` in the next section also reports authentication. If the IDE is signed into a different Cursor account than `agent login`, the machine will not show up for the account you use in the browser.

### Start and name the worker

Start the worker from a PowerShell window where all of the following are already true:

- `agent --version` works
- `agent login` has completed for the Pro account
- `ssh -T git@<host>` succeeds
- the current directory can `cd` to the dedicated clone
- the corporate VPN is connected, if that host requires it

Use an explicit name so the execution environment is unambiguous. Do not pass `--computer-use`. That flag is for macOS and Linux.

Cursor's examples place `--name` and a single `--worker-dir` after `start`, and place repeated `--worker-dir` before `start`. Run the help once and follow the accepted order:

```powershell
agent worker start --help
```

Then start one worker for the one SSH clone:

```powershell
agent worker start `
  --name "my-windows-laptop" `
  --worker-dir "C:\cursor-worker\experiment-clone"
```

Use the real clone path from the previous step. The path must exist. Cursor reads the git remotes inside it and registers those for routing, so the registered repo string will be the `git@<host>:<path>.git` URL, not an HTTPS URL.

What you should see: the process stays in the foreground and keeps the session open. It does not exit back to a prompt. That window is the worker. Closing it, or letting the laptop sleep, drops the session.

While it is running:

- [ ] Do not open this clone in the IDE and edit it.
- [ ] Do not start a second `agent worker` for the same checkout.
- [ ] More than one Cloud Agent can be routed to this same machine. Run one experiment task at a time so the two tasks cannot edit the same files.
- [ ] Before each later task, look at this window and confirm the process is still the one you started. If it exited, start it again from this same prepared shell and re-run `agent worker debug`.

Keep the laptop awake for the whole experiment: plugged in, sleep set to Never, lid close set not to sleep, no idle lock that drops the VPN or Wi-Fi. The worker is held up by the outbound session from this process. Grok Bot's cloud computer keeps running if the laptop sleeps. This worker does not.

### Data that leaves the laptop

The checkout stays on disk. During a run, the worker sends Cursor the content the agent needs: file contents, terminal output, diffs, local MCP results, and routing metadata. Do not register a checkout that contains secrets. After the first direct run, note which MCP servers the session actually had. Stdio MCP servers execute on the laptop; HTTP / SSE MCP servers run on Cursor's backend.

Privacy Mode applies to self-hosted workers. With it enabled, code sent from the worker is not used for training. Privacy Mode (Legacy) is a different setting and blocks Grok Bot.

---



## 3. Verify worker networking before involving Grok Bot

The worker establishes the connection **outbound** to Cursor.

```text
Windows laptop
      │
      │ outbound HTTPS
      ▼
Cursor
```

No inbound port, public IP, or inbound VPN tunnel is required.

Cursor currently lists these worker endpoints:

```text
api2.cursor.sh                                    — agent session
api2direct.cursor.sh                               — agent session
downloads.cursor.com                               — CLI updates
cloud-agent-artifacts.s3.us-east-1.amazonaws.com   — artifact uploads
```

`downloads.cursor.com` is also the host for the first-time Computer Use install, which applies to macOS. On this Windows laptop the host still matters for CLI updates.

If artifact upload is blocked, the agent session continues, but screenshots and log references will be missing in the dashboard. This experiment does not depend on screenshots.

The Grok Bot desktop app uses additional hosts, including the `*.*.cursorvm.com` pattern Cursor documents for clients behind a TLS-inspecting proxy. Those are not a substitute for the four worker hosts. A worker that never appears in the environment dropdown is a worker-network or auth failure, not a Grok Bot failure.

Reference:

- [https://cursor.com/docs/cloud-agent/self-hosted](https://cursor.com/docs/cloud-agent/self-hosted)
- [https://cursor.com/docs/cloud-agent/self-hosted/my-machines](https://cursor.com/docs/cloud-agent/self-hosted/my-machines)



### Corporate-network check

Because this is a corporate Windows laptop/network:

- [ ] Verify outbound HTTPS access to all four worker hosts listed above.
- [ ] Verify whether an HTTP(S) proxy is required.
- [ ] If a proxy is required, configure `HTTPS_PROXY` / `https_proxy` in the worker environment.
- [ ] If a TLS-inspecting gateway (Zscaler or similar) is present, allow the worker hosts and exempt them from inspection. Setting `HTTPS_PROXY` does not fix a rejected certificate.
- [ ] Do not open inbound firewall ports for the worker.
- [ ] Disable sleep and lid-close suspend for the experiment window.

---



## 4. Verify My Machines independently of Grok Bot

Before testing Grok, prove that Cursor Cloud Agent can use the worker **by itself**.

### Preflight

With the worker process running, from the dedicated clone:

```powershell
agent worker debug
```

This checks authentication, privacy routing, repo labels, and whether Cursor can see matching workers. Fix anything it reports before opening the dashboard. To print the same diagnostics at start time, use `agent worker start --debug`.

Then go to **cursor.com/agents**:

1. [ ] Confirm `my-windows-laptop` appears in the environment/run-on selector.
2. [ ] Select `my-windows-laptop`. Do not leave the environment on a Cursor-hosted VM.
3. [ ] Select the test repository and the disposable branch.
4. [ ] Confirm the worker process is still running.
5. [ ] Run a tiny task that forbids git writes.

Suggested task:

> In this checkout, do not commit, do not push, and do not open a pull request. Create a file named `cloud_agent_worker_test.txt` containing the current hostname, the current git branch, the output of `git remote -v`, and a short sentence saying this file was created by a Cursor Cloud Agent running on the My Machines worker. Then show the terminal commands used.

Expected result:

```text
Cursor Cloud Agent
        │
        ▼
My Machines worker
        │
        ▼
File appears in the dedicated clone on Windows
```

Verify on the laptop:

- [x] The file is physically present in the dedicated clone.
- [x] `git status` shows only that file. No commit, no push, HEAD unchanged from the SHA you recorded.
- [x] The dashboard run's environment is `my-windows-laptop`.

This step isolates **Cursor Cloud Agent ↔ worker** from the Grok Bot integration.

---



## 5. Verify that the agent really executed locally

Do not rely only on the final textual answer, and do not treat "a file exists somewhere on the laptop" as proof if you cannot tie it to the dashboard run.

Good proof signals, checked on the dedicated clone after the run:

- [x] The created file is physically present in that clone.
- [x] `hostname` in the file matches this laptop.
- [x] Branch name and `git remote -v` in the file match what you recorded before the run.
- [x] HEAD is unchanged, and `git status` shows only the intended file.
- [x] The dashboard environment is `my-windows-laptop`.

Optional extra signals, still inside the clone:

```powershell
hostname
python --version
git remote -v
git status
git rev-parse HEAD
```

Avoid credentials or confidential files. Anything the agent reads is sent to Cursor.

The point is to establish:

> Cloud Agent reasoning happens in Cursor's cloud, but the tool action actually happens on this Windows worker, in this clone.

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

xAI's engineering guide says Grok Bot can create and manage Cursor Cloud Agents and can start them on a user's own worker machines. It does not document a `worker=` parameter for Grok Bot chat. Cursor documents `worker=` / `machine=` only for Slack, GitHub, and Linear. Treat chat targeting as unverified.

Before the Grok task:

- [x] Worker process is still running. Laptop is awake.
- [x] Dedicated clone is still on the disposable branch, and you have noted the new HEAD if §4 left the test file uncommitted.
- [x] You will judge success from the Cursor dashboard environment field, not from Grok Bot's self-report.

Reference:

- [https://x.ai/bot/guides/grok-bot-for-engineering](https://x.ai/bot/guides/grok-bot-for-engineering)
- [https://cursor.com/docs/grok-bot/teams](https://cursor.com/docs/grok-bot/teams)
- [https://cursor.com/docs/cloud-agent/self-hosted/my-machines](https://cursor.com/docs/cloud-agent/self-hosted/my-machines)



### First Grok task

Keep the task tiny. Name the worker, the `git@` remote copied from `git remote -v`, and the branch. Tell Grok Bot the work must be a Cursor Cloud Agent run on that worker, not work on Grok Bot's own computer and not Grok Bot local execution.

> Using a Cursor Cloud Agent, start a run on my registered My Machines worker named `my-windows-laptop`. Do not use a Cursor-hosted VM. Do not do this task on your own computer or via local execution. Target repository `<git@host:path.git copied from git remote -v>`, branch `<disposable branch>`. The repository string is the SSH remote, not an https URL. In that checkout, do not commit, do not push, and do not open a pull request. Create a file named `grok_cloud_worker_test.txt` containing: (1) the machine hostname, (2) the current git branch, (3) the output of `git remote -v`, (4) a one-line description of what you changed. Then run `<test command>` and report its exact output. In your final report, state the Cursor run id and the environment shown for that run.

Fill in the `git@` remote, branch, and test command from the dedicated clone before sending this. Paste the remote exactly. Do not rewrite it to `https://`.

If the dashboard shows a Cursor-hosted VM, the targeting step failed even if the file contents look right. If no Cloud Agent run exists, Grok Bot did the work on another surface.

---



## 7. Verify the complete chain

Collect evidence at each layer.

### Layer A — Grok Bot

- [x] Grok Bot accepted the engineering task.
- [x] Grok Bot created or managed a Cursor Cloud Agent, visible as a run in Cursor.
- [x] Grok Bot could inspect the Cloud Agent result or transcript.
- [x] Grok Bot could issue a follow-up if needed.
- [x] The task was not completed only on Grok Bot's cloud computer or via Grok Bot local execution.



### Layer B — Cursor Cloud Agent

- [x] Cloud Agent run exists in Cursor. Record the run id.
- [x] Record the pinned default, the model the spawn requested, and the model on the usage page.
- [x] Cloud Agent performed reasoning and issued tool calls.
- [x] Cloud Agent was associated with the intended repository and disposable branch.



### Layer C — My Machines worker

- [x] Dashboard environment for that run id is `my-windows-laptop`. Grok Bot's self-report is recorded and then checked against this.
- [x] The new file is in the dedicated clone on the laptop.
- [x] Shell output in the transcript matches a command you can reproduce in that clone.
- [x] `git status` shows only the intended new file. HEAD is unchanged. Nothing was pushed.



### Layer D — Final feedback loop

Run the success path and the retry path as two separate tasks, on a clean understanding of the tree between them. A system that is eager to declare victory can pass a single naive test.

#### Success path

Use the §6 task as the success path. Its checkable criteria are: the file exists in the dedicated clone with the four required lines, `<test command>` output is reported exactly, the dashboard environment is `my-windows-laptop`, and git HEAD is unchanged.

- [x] Cloud Agent completes on the first attempt, with no errors.
- [x] Grok Bot checks the file contents and the dashboard environment, not only the agent's closing message.
- [x] Grok Bot reports completion with the run id and the command output.
- [x] Grok Bot does not trigger another run after those checks pass.



#### Retry path (deliberate failure)

Add this file to the dedicated clone yourself, on the disposable branch, before the Grok task. Commit it yourself if you want a clean baseline, or leave it uncommitted and record that fact. Do not tell Grok Bot the password.

`check_marker.py`:

```python
from pathlib import Path
import sys

marker = Path("marker.txt")
expected = "MARKER_OK"
if marker.is_file() and marker.read_text(encoding="utf-8").strip() == expected:
    print("ok")
    sys.exit(0)
print("missing marker")
sys.exit(1)
```

Task to Grok Bot:

> Using a Cursor Cloud Agent on My Machines worker `my-windows-laptop`, repository `<git@host:path.git copied from git remote -v>`, branch `<disposable branch>`, make `python check_marker.py` exit 0. The repository string is the SSH remote, not an https URL. Do not commit, do not push, and do not open a pull request. Do not edit `check_marker.py`. The run must be on `my-windows-laptop`, not a Cursor-hosted VM and not your own computer. Verify by running the script. If it fails, read the output, correct the checkout, and run it again. Report the run id, the environment, and the final script output.

The first attempt fails until something writes `marker.txt` containing exactly `MARKER_OK`. That failure is deterministic.

- [ ] First attempt fails with `missing marker`.
- [ ] That output reaches Grok Bot.
- [ ] Grok Bot treats it as a failure.
- [ ] Grok Bot sends a correction without you intervening.
- [x] Retry succeeds, `marker.txt` exists only in the dedicated clone, and `check_marker.py` is unchanged.
- [x] Grok Bot confirms with the script output and the dashboard environment.
- [x] Record: number of retries, whether Grok Bot handed the task back to you, and total time.

Expected pattern:

```text
Grok
  ↓
Cursor Agent
  ↓
run python check_marker.py on the worker
  ↓
failure (missing marker)
  ↓
observe output
  ↓
write marker.txt
  ↓
script exits 0
  ↓
Grok checks output and dashboard environment
```

This is the Level-3 loop: Grok Bot notices failure and recovery, and on the earlier success path it also stops when the proof is already there.

---



## 8. Recommended safety constraints for the first experiment

- [ ] Dedicated clone of a test repository. Not `d:\Data\grok-tg-kid-demo`.
- [ ] Disposable branch. Not `main`.
- [ ] Clean `git status` before the first run. Record HEAD.
- [ ] Prompt forbids commit, push, and pull requests. Confirm HEAD afterward.
- [ ] No production credentials in the clone. Anything the agent reads is sent to Cursor.
- [ ] No destructive shell commands.
- [ ] No second agent run against the same clone while one is in progress.
- [ ] Keep On-Demand Usage **OFF**. Record the Cloud Agent spend limit.
- [ ] Snapshot Cloud Agent usage and Grok Bot usage before and after.
- [ ] Pin a non-Fast first-party model. Record pin, requested model, and billed model.
- [ ] Laptop awake, worker process confirmed running before each task.
- [ ] Record worker name, the `git@` remote, branch, run id, and dashboard environment.

---



## 9. Cost controls and interpretation

Cloud Agent usage is charged at the selected model's API price. Pro's **Cursor Models** pool covers Grok 4.7, Grok 4.6, Grok 4.5, and Composer 2.5. Fast variants of those models draw the same pool at the higher Fast rates. Third-party models draw the **Other Models** pool.

```text
Included Cursor Models usage (monthly)
    ↓
used by Cloud Agent
    ↓
included pool decreases

Grok Bot included usage (weekly)
    ↓
used by the outer bot, separate from the Cloud Agent pool

When included usage is exhausted:
    │
    ├── On-Demand OFF → stop
    │
    └── On-Demand ON → additional paid usage
```

A low Cloud Agent spend limit can stop a run even while on-demand is off. Record that number so a stopped run is interpretable.

References:

- [https://cursor.com/docs/models-and-pricing](https://cursor.com/docs/models-and-pricing)
- [https://cursor.com/help/account-and-billing/overages](https://cursor.com/help/account-and-billing/overages)
- [https://cursor.com/docs/cloud-agent](https://cursor.com/docs/cloud-agent)
- [https://cursor.com/docs/grok-bot](https://cursor.com/docs/grok-bot)

---



## 10. Optional phase: test the value of Grok as manager

Run this only after §4–§7 pass. Otherwise the comparison mixes "the pipeline does not work" with "the manager added nothing."

### A. Direct Cloud Agent

```text
You → Cursor Cloud Agent → My Machines
```



### B. Level 3

```text
You → Grok Bot → Cursor Cloud Agent → My Machines
```

Use the same task, the same dedicated clone reset to the same SHA, the same worker, and the same pinned model. Confirm the billed model matched on both sides. Do not treat Grok Bot memory or extra skills as part of the result unless that context is also given to the direct Cloud Agent.

Measure whether Grok adds value through:

- better task decomposition
- better context gathering
- better verification
- useful follow-ups
- recovery from failures
- reduced human supervision

This is the experiment that tests whether the agent-managing-an-agent architecture is useful, rather than merely technically possible.

---



## 11. Expected final architecture

After successful setup:

```text
                         YOU
                          │
                          ▼
                    GROK BOT
                    outer agent
                          │
                          │ creates/manages
                          ▼
                 CURSOR CLOUD AGENT
                    inner coding agent
                          │
             reasoning / planning / tool calls
                          │
                          ▼
                MY MACHINES WORKER
                   "my-windows-laptop"
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
       dedicated clone   terminal     tests
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

The worker is a **Cursor CLI process on the laptop** that executes Cloud Agent tool calls inside a dedicated clone.

It is not the interactive Cursor IDE Agent, not Grok Bot's cloud computer, and not Grok Bot local execution. The IDE session on `grok-tg-kid-demo` can stay as it is, as long as the worker is not pointed at that folder.

---



## 12. Success criteria

The experiment is successful only if all of these are demonstrated:

- [ ] Pro account can create a Cloud Agent.
- [ ] A non-Fast first-party model is pinned, and the pin, the spawn's requested model, and the usage-page model are all recorded.
- [ ] On-Demand Usage remains OFF. The spend limit is recorded. Both usage meters are snapshotted.
- [ ] A dedicated clone, not `grok-tg-kid-demo`, is registered as `my-windows-laptop`.
- [ ] `agent worker debug` is clean, and the machine appears in the dashboard selector.
- [ ] Cursor can run a Cloud Agent directly on that worker, and the dashboard environment says so.
- [ ] The worker executes commands and file writes in that clone. HEAD is unchanged unless you committed the fixture yourself.
- [ ] Grok Bot can create and manage a Cursor Cloud Agent that the dashboard shows on `my-windows-laptop`.
- [ ] A Grok Bot self-report of the environment was checked against that dashboard field.
- [ ] Grok Bot observes results, distinguishes the §7 success path from the §7 retry path, and completes one recovery cycle without you writing the fix.
- [ ] No inbound connection to the laptop is required.
- [ ] No commit, push, or pull request was created by an agent.
- [ ] No unexpected paid usage occurs.

---



## Current documentation to verify independently

**xAI**

- Grok Bot for Engineering: [https://x.ai/bot/guides/grok-bot-for-engineering](https://x.ai/bot/guides/grok-bot-for-engineering)

**Cursor**

- Cloud Agents: [https://cursor.com/docs/cloud-agent](https://cursor.com/docs/cloud-agent)
- My Machines: [https://cursor.com/docs/cloud-agent/self-hosted/my-machines](https://cursor.com/docs/cloud-agent/self-hosted/my-machines)
- Self-Hosted Machines: [https://cursor.com/docs/cloud-agent/self-hosted](https://cursor.com/docs/cloud-agent/self-hosted)
- Computer use: [https://cursor.com/docs/cloud-agent/self-hosted/computer-use](https://cursor.com/docs/cloud-agent/self-hosted/computer-use)
- Cloud Agent Settings: [https://cursor.com/docs/cloud-agent/settings](https://cursor.com/docs/cloud-agent/settings)
- Models & Pricing: [https://cursor.com/docs/models-and-pricing](https://cursor.com/docs/models-and-pricing)
- Grok Bot overview: [https://cursor.com/docs/grok-bot](https://cursor.com/docs/grok-bot)
- Grok Bot for Teams: [https://cursor.com/docs/grok-bot/teams](https://cursor.com/docs/grok-bot/teams)
- Usage-based charges: [https://cursor.com/help/account-and-billing/overages](https://cursor.com/help/account-and-billing/overages)
- Spend limits: [https://cursor.com/help/account-and-billing/spend-limits](https://cursor.com/help/account-and-billing/spend-limits)

**Documentation status:** 2026-09-15 revision checked on September 15, 2026. This revision checked against current Cursor and xAI pages on September 23, 2026.