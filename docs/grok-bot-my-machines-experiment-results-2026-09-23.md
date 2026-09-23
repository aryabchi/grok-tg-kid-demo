# Experiment results: Grok Bot → Cursor Cloud Agent → My Machines

*23 September 2026. What was proved, and how to stand the same pipeline up again. Outcome of `grok-bot-cursor-cloud-agent-my-machines-experiment-plan-2026-09-23.md`.*

## What was proved

On a disposable GitLab repo, cloned over SSH onto a Windows laptop:

1. **Cursor Cloud Agent executes on a My Machines worker.** With the worker selected in the agents UI, it wrote a file in that checkout, ran `hostname`, `git`, and `python` there, then committed and pushed with the laptop's SSH key. The file's hostname and `git remote -v` matched the laptop.

2. **Grok Bot can start that worker from chat.** Told to use a Cursor Cloud Agent on `<worker-name>` and not a Cursor-hosted VM, it created a run whose environment was that worker and that checkout, and reported the run id, status, and environment. It passed no model. The run used the account's saved Cloud Agent default.

3. **A later message in the same chat follows up on that run.** It committed the test file, pushed, and opened a merge request, still on `<worker-name>`.

4. **Grok Bot re-checks before it ships.** Asked to make a local script exit 0, the agent failed once, wrote the missing file, and reran the script successfully. Asked to commit only if the script still exited 0, it ran the script again, saw a failure, and did not commit. Told to fix and continue, it made the script pass, then committed and pushed.

5. **A self-hosted GitLab that is not registered with Cursor can still receive a merge request from the worker.** Cursor's built-in pull-request tool failed. The Cloud Agent opened the merge request through a Git integration already available on the worker. The branch had to be named `cursor/<description>-<suffix>`.

6. **The two products bill separately.** Grok Bot's own turns appear as `grok-bot-default` on its weekly allowance. The Cloud Agent run is a different row on the Cursor plan. The run transcript Grok Bot could read did not include a model id.

Not checked: the billed Cloud Agent model against the pinned default, and whether Grok-as-manager beats talking to Cloud Agent directly.

## Architecture

```text
You
  │  chat
  ▼
Grok Bot                    cloud computer of its own; billed as grok-bot-default
  │  creates / follows up
  ▼
Cursor Cloud Agent          reasoning in Cursor's cloud; model = account default
  │  tool calls
  ▼
My Machines worker          process on the Windows laptop
  │
  ▼
Dedicated SSH clone         git push uses the laptop's SSH key
```

Proof is a Cloud Agent run whose environment is `<worker-name>`, plus a file in that clone that matches the laptop's git remote and a local command. Grok Bot's own computer and Grok Bot local execution are other surfaces. The prompts forbade both.

## How to reproduce the setup

### 1. Account

- A paid Cursor plan with Cloud Agents and Grok Bot. The same user for the IDE, `agent login`, and Grok Bot.
- Privacy Mode (Legacy) off. It blocks Grok Bot. Cloud Agent delegation left on.
- On-demand usage off. Snapshot the Grok Bot meter and the Cloud Agent meter before the first run. Set and record the spend limit when the first Cloud Agent run asks for one.
- Pin a non-Fast Cloud Agent default model. Grok Bot does not pass a model, so this default is what runs. Compare it afterward with the Cloud Agent row on the usage page.

### 2. Disposable repository

Use a repo created for the experiment, not the daily IDE workspace. Remote stays SSH. Cursor registers `git remote get-url origin`. An `https://` URL does not match.

In a non-elevated PowerShell, as the Windows account that will start the worker:

```powershell
ssh -T git@<gitlab-host>
git clone git@<gitlab-host>:<group>/<repo>.git <path-to-dedicated-clone>
cd <path-to-dedicated-clone>
git remote get-url origin
git switch -c <worker-branch>
git status
```

`ssh -T` must authenticate, `git remote get-url origin` must print a `git@` URL, and `git status` must be clean. Write down `<worker-branch>` and that remote. Work on `<worker-branch>`, not `<default-branch>`.

If `ssh -T` succeeds only while a VPN is connected, connect it in this window and leave it up for the whole experiment. The worker runs git locally. If the key has a passphrase, load it into `ssh-agent` in this same window. The worker inherits this window only.

This GitLab was not added as a Cursor source-control integration. Cursor's pull-request tool then fails. A merge request still works when the worker already has a Git integration for that host, and the branch is named `cursor/<description>-<suffix>`. Enable that integration on this Cursor account before starting the worker.

### 3. Cursor CLI

Install in a non-elevated PowerShell, then open a new window so `PATH` updates:

```powershell
irm 'https://cursor.com/install?win32=true' | iex
```

```powershell
agent --version
agent login
```

`agent login` uses the same Cursor user as the IDE. This experiment ran CLI `2026.09.18-9a7762b`. Use whatever `agent --version` prints as `<cli-version>`.

On that Windows build, `agent worker start` authenticates and then exits: `better_sqlite3.node` is built for Node ABI 127, and the bundled Node is ABI 137. The package is `better-sqlite3` 12.11.1. Reinstalling the CLI reproduces the crash. Replace only that file with the official `better-sqlite3-v12.11.1-node-v137-win32-x64` prebuild. Confirm the version on disk is 12.11.1 before using this URL. `agent update` installs a new directory and needs the same replacement.

```powershell
$dir = "$env:LOCALAPPDATA\cursor-agent\versions\<cli-version>\node_modules\better-sqlite3\build\Release"
Copy-Item "$dir\better_sqlite3.node" "$dir\better_sqlite3.node.bak"
$tmp = Join-Path $env:TEMP "better-sqlite3-v137"
New-Item -ItemType Directory -Force -Path $tmp | Out-Null
$url = "https://github.com/WiseLibs/better-sqlite3/releases/download/v12.11.1/better-sqlite3-v12.11.1-node-v137-win32-x64.tar.gz"
Invoke-WebRequest -Uri $url -OutFile "$tmp\bsql.tar.gz"
tar -xzf "$tmp\bsql.tar.gz" -C $tmp
Copy-Item "$tmp\build\Release\better_sqlite3.node" "$dir\better_sqlite3.node" -Force
```

### 4. Start the worker

Same window: `agent login` done, `ssh -T git@<gitlab-host>` still succeeds, `python --version` prints a version. If SSH needs a VPN, it stays connected. If the network requires an HTTP proxy, set `HTTPS_PROXY` here. A proxy variable does not fix a TLS inspector that rejects the worker hosts. Allow:

```text
api2.cursor.sh
api2direct.cursor.sh
downloads.cursor.com
cloud-agent-artifacts.s3.us-east-1.amazonaws.com
```

The worker dials out. No inbound port.

```powershell
agent worker start --help
agent worker start `
  --name "<worker-name>" `
  --worker-dir "<path-to-dedicated-clone>"
```

Follow the flag order `--help` prints. Do not pass `--computer-use`. The process stays in the foreground. Sleep or closing the window drops it. Grok Bot's cloud computer keeps running if the laptop sleeps. Keep the machine awake for the experiment.

`agent worker debug` should show that Cursor can see this worker and repo. `<worker-name>` then appears at cursor.com/agents. One run at a time. The worker edits this checkout in place, and a push is a push as the laptop's git user.

### 5. Prove the worker, then add Grok Bot

At cursor.com/agents, select `<worker-name>` and `<worker-branch>`. Run a task that writes one file and runs `hostname`, `git remote -v`, and `python --version`, and that forbids commit, push, and a merge request. The file must be in the clone, and the run page must show `<worker-name>`. One follow-up on that run can commit, push, and open a merge request into `<default-branch>`, which confirms the Git integration.

Create a Grok Bot whose job is code and repositories. Confirm the worker process is still running, then send the prompt in the next section. `worker=` / `machine=` is documented for Slack, GitHub, and Linear, not for this chat. Naming `<worker-name>` and the checkout path was enough. A later message in the same chat follows up on that Cloud Agent run. Repeat the worker name in the follow-up. Check the run page. Grok Bot's report of the environment is a claim until that page agrees.

## Best practices

Grok Bot can finish a task on its own cloud computer, through local execution in the desktop app, on a Cursor-hosted VM, or on the My Machines worker. Only the last one is this pipeline. Say that in the first message and in every follow-up. When the built-in pull-request tool failed, a short "open the merge request" was not enough: Grok Bot started to do the job on the laptop directly, and stopped only after it was told to stay on the Cloud Agent on `<worker-name>`.

```text
Using a Cursor Cloud Agent, start a run on my registered My Machines
worker named <worker-name>. Workspace: <path-to-dedicated-clone>.
Do not use a Cursor-hosted VM. Do not do this on your own computer
or via local execution. Do not commit, push, or open a merge request.
In your final report, state the run id, its status, and the environment
shown for that run.
```

Ask for a check you can repeat in the clone (`python --version`, `git status`, the file). Say what to do on failure ("read the output, fix the checkout, run it again") and on success ("stop"). One Cloud Agent at a time against that clone.

Shell commands on the worker need no introduction. `git`, `python`, and `hostname` ran because they are on the laptop.

Other tools do. Cursor's pull-request tool only knows Git hosts registered with Cursor, so it failed here. The merge request went through a Git integration already on the worker. Grok Bot used it after a prompt told it to, not because it searched for one. When the task needs such a tool, name it in the same message ("the Git integration already available on this worker"), forbid the built-in substitute, forbid leaving the worker, and give the branch shape (`cursor/<description>-<suffix>`).

Do not ask this Windows worker for screenshots or computer use. That exists on macOS and Linux. Proof here is files, git state, and command output.
