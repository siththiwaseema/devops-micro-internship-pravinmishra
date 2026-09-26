# Assignment 6 — Build an AI-Assisted Linux Health Check (AI-Assisted Linux Incident Triage)

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will build a read-only Bash triage script that checks the health of your Ubuntu server and Nginx application, connect it to Claude Code as a reusable `/linux-triage` skill, simulate a controlled Nginx incident, use the skill to gather and analyze evidence, recover the service manually, and verify recovery. The workflow follows the Agentic Loop: Gather → Analyze → Human Act → Verify.

---

# Task 1 — Confirm the Healthy Baseline and Create the Workspace

## Goal

Confirm that Nginx and the React application are healthy before building the automation.

### Evidence

#### Screenshot 1 — Output of `systemctl is-active nginx`, `ss -ltn | grep ':80'`, and `curl -I http://localhost`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/1.png)

---

#### Screenshot 2 — Output of `pwd` and `find . -maxdepth 4 -type d | sort` showing the workspace folder structure

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/2.png)
---

### Notes

Answer the following in your own words:

**1. What proves that Nginx is running?**

`systemctl is-active nginx` returned **`active`**. That comes directly from systemd, the
Linux service manager, and it means the Nginx service is currently running. If Nginx had
stopped or crashed, it would show `inactive` or `failed` instead.
The `curl -I http://localhost` response also included a `Server: nginx` header, which shows
Nginx was the program that answered the request.
---

**2. What proves that the server is listening for HTTP traffic?**

`ss -ltn | grep ':80'` showed a line in **`LISTEN`** state on `0.0.0.0:80` (and `[::]:80`
for IPv6). Port 80 is the standard HTTP port, and `0.0.0.0` means the server accepts
connections on all network interfaces.
Listening alone doesn't prove the app works, so I also ran `curl -I http://localhost`, which
returned **`HTTP/1.1 200 OK`**. This proves the server is listening and actually answering
HTTP requests successfully.
---

**3. Why must you capture a healthy baseline before simulating an incident?**

A baseline is a record of what "normal" looks like: the service is active, port 80 is
listening and the app returns 200 OK. It matters for three reasons:
- **To compare against during the incident.** When something breaks, I can see exactly what
  changed, for example `active` became `inactive`, or `200` became `502`/`connection refused`.
- **To prove the problem came from the simulation.** If I skip this check and the server was
  already broken, I could blame the wrong cause and waste time on the wrong fix.
- **To know when recovery is complete.** "Fixed" means getting back to the baseline, with
  the same status, port and response as before, not just "it looks OK now".
In real operations, engineers always confirm a known-good state before making changes, so
that any difference afterwards can be clearly traced and verified.
---

# Task 2 — Create Project Context and Safety Rules in CLAUDE.md

## Goal

Tell Claude exactly what this project does and what it is not allowed to do.

### Evidence

#### Screenshot 3 — CLAUDE.md open in VS Code showing all four sections (Project Overview, Incident Workflow, Safety Rules, Output Rules)

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/3.png)

---

### Notes

Answer the following in your own words:

**1. Why should Claude receive project-specific operational rules?**

Without rules, an AI assistant only has general knowledge. It doesn't know what this project
is for, which actions are allowed, or how this team wants incidents handled. Left on its
own, it might suggest restarting services, editing configs or deleting files, which are
risky actions on a real server.
`CLAUDE.md` is loaded automatically as project context, so every session starts with the
same instructions. It defines:
- **The role:** the Bash script collects evidence, and Claude only analyzes it.
- **The process:** gather evidence, analyze, have a human approve and run the fix, then
  verify.
- **The limits:** no restarts, no package changes, no config edits, no deletions.
- **The output format:** the same six-part structure for every analysis.
This makes Claude's behavior predictable and safe, and it keeps the triage process
consistent every time.

---

**2. Why is the human required to execute the recovery command?**

- **Safety:** a wrong command on a production server can cause a bigger outage than the
  original problem, for example restarting a service with a broken config. The human checks
  the command before it runs.
- **Accountability:** a person owns the decision and its result. In real teams, changes to
  production need a human's approval.
- **The AI can be wrong:** Claude only sees the report. It could misread the evidence or be
  missing information that the human has.
- **Context:** the human knows things the AI doesn't, like planned maintenance, recent
  deployments, or whether now is a safe time to restart.
This is the "human-in-the-loop" approach. The AI speeds up analysis, but the human keeps
control of every action that changes the system.

---

**3. Which rule prevents Claude from making an unsupported diagnosis?**

**"Do not claim a root cause unless the report contains supporting evidence."**
This rule stops Claude from guessing or making up a cause that sounds believable but isn't
proven. Two other rules support it:
- **"Use only the Bash report as the primary source of incident evidence"**: the analysis
  must be based on real collected data, not assumptions.
- **Output Rule 3, "Exact evidence from the report"**: Claude must show the actual lines
  that support its conclusion, so the human can check them.
Together, these rules make every diagnosis evidence-based and easy to verify.
---

# Task 3 — Use Agentic AI to Plan Before Writing the Script

## Goal

Use Claude Code to inspect the environment and produce a read-only plan before creating any Bash code.

### Evidence

#### Screenshot 4 — Claude Code showing the five-check plan and read-only inspection results

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/4.png)

---

### Notes

Answer the following in your own words:

**1. Which part of this task represents the Gather phase?**

The Gather phase is when Claude Code **read `CLAUDE.md` and inspected the server using
read-only commands** before proposing anything. It used tools like `Read` for the project file
and Bash commands such as `systemctl is-active nginx`, `ss -ltn`, `curl -I http://localhost`,
`df -h /` and `free -h` to see the current state of the system.
This is the "look before you act" step. Claude collected facts about the real environment,
such as which services run and which ports listen, so the plan is based on evidence and not
assumptions. The five-check plan it proposed afterwards is the start of the Analyze and Plan
phase.

---

**2. Did Claude follow the instruction not to create files? How did you verify this?**

Yes. Claude only read files and ran read-only commands. It didn't create or edit anything.
I verified this in several ways:
- **Watching the tool calls:** Claude Code shows every tool it uses. I only saw read-type
  actions (Read, and Bash commands like `systemctl`, `ss`, `curl`, `df`, `free`). There
  were no Write or Edit actions.
- **Approval prompts:** Claude Code asks for permission before changing files or running
  risky commands. It didn't request any file changes, and I would have rejected any request
  to create or edit a file.
- **Comparing the folder before and after:** I ran `find . -maxdepth 4 | sort` before
  starting Claude and again after exiting. The output was the same, with only `CLAUDE.md`
  and the empty `scripts`, `reports` and `.claude/skills/linux-triage` folders.
- **Checking timestamps:** `ls -la` showed that `CLAUDE.md` still had its original
  modification time, so it wasn't edited.

---

**3. Why is planning before coding useful in DevOps automation?**

- **Catches mistakes early:** reviewing the plan (commands, healthy results, failure
  meanings) is much cheaper than debugging a script that is already running on a server.
- **Grounds the script in the real environment:** inspecting first confirms things like the
  Nginx service name, the port, and which tools are installed, so the script doesn't rely on
  wrong assumptions.
- **Defines what "healthy" and "failed" mean up front:** each check gets clear criteria
  before any code exists, which makes the script's logic and output reliable.
- **Keeps the human in control:** I can approve, change or reject the plan before any code
  is written, especially important when an AI agent is doing the work.
- **Safer automation:** automation runs quickly and repeatedly, so a bad script can cause
  damage at scale. Planning reduces that risk, and a read-only plan means nothing can break
  while it's being designed.

---

# Task 4 — Build the Linux Triage Bash Script

## Goal

Create one Bash script that gathers consistent Linux and Nginx health evidence.

### Evidence

#### Screenshot 5 — Top section of `linux-triage.sh` showing variables, thresholds, and the checks array

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/5.png)

---

#### Screenshot 6 — Middle section showing check functions and conditionals

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/6.png)

---

#### Screenshot 7 — Bottom section showing the loop, summary function, and exit behavior

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/7.png)

---

#### Screenshot 8 — Output of `bash -n scripts/linux-triage.sh` (no syntax errors) and `ls -l scripts/linux-triage.sh` showing executable permission

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/8.png)

---

### Notes

Answer the following in your own words:

**1. What is stored in the checks array?**

The `checks` array stores the **names of the five health-check functions**:
`check_service`, `check_port`, `check_http`, `check_disk` and `check_memory`.
It doesn't store results or data, just the list of checks the script should run, in order.
It works like a checklist of jobs. To add a new check, for example SSL expiry, I would write
a new function and add its name to the array. The rest of the script doesn't need to change.

---

**2. How does the `for` loop use that array?**

The loop `for check_function in "${checks[@]}"` goes through the array one item at a time.
On each round, the current function name is stored in `check_function`, and the line
`"$check_function"` **runs that function**. Bash replaces the variable with the name, for
example `check_service`, and then calls it like any other command.
So the loop runs all five checks automatically, in the order they are listed in the array,
and each one records its PASS, WARN or FAIL result in the report.

---

**3. Why are the health checks separated into functions?**

- **One job per function:** each function tests one thing, so the code is easy to read,
  understand and fix.
- **Easy to debug:** if the HTTP check behaves oddly, I only need to look at `check_http`.
- **Easy to extend:** new checks can be added without touching the existing ones, just by
  writing a new function and adding it to the array.
- **Consistent results:** every check reports through the same helpers (`mark_pass`,
  `mark_warning`, `mark_failure`), so the report format and the counters stay consistent.
- **Loop-friendly:** because each check is a function, the array and loop can run them all
  automatically.
- **Local variables:** `local` keeps values like `http_code` and `disk_usage` inside their own
  function, so the checks can't accidentally overwrite each other's data.

---

**4. What is the purpose of `$(...)` in this script?**

`$(...)` is **command substitution**. It runs a command and puts its output into the script,
usually into a variable, so the script can make decisions based on live system data.
Examples from the script:
- `http_code=$(curl -s -o /dev/null -w '%{http_code}' ...)` saves the HTTP status code,
  for example `200`.
- `disk_usage=$(df -P / | awk ...)` saves the root disk usage as a number, for example `34`.
- `available_memory=$(free -m | awk ...)` saves the available memory in MB.
- `$(date -u ...)` and `$(hostname)` add the timestamp and server name to the report header.
- `base_dir="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"` finds the project folder
  from the script's own location, so the report is always saved in `reports/`, no matter
  which folder the script is run from.
Without `$(...)`, the script could run these commands but couldn't use their results in
`if` conditions or in the report.

---

**5. Why does the script use different exit codes for HEALTHY, WARN, and FAIL?**

An exit code is a number a script returns when it finishes. Other programs can read it
without having to read the text output. This script uses:
- **0 = HEALTHY:** all checks passed. In Linux, 0 always means success.
- **1 = WARN:** nothing is broken, but something needs attention, for example disk above
  80% or memory running low.
- **2 = FAIL:** at least one critical check failed, for example Nginx down, port 80 not
  listening, or HTTP not returning 200.
Different codes let automation react to how serious the problem is. For example, a cron job,
CI/CD pipeline or monitoring tool could do nothing on 0, send a warning on 1, and page the
on-call engineer or stop a deployment on 2. I can check the code after running the script
with `echo $?`. If the script only returned "success" or "failure", a small warning and a
full outage would look the same.

---

# Task 5 — Run and Understand the Healthy-State Report

## Goal

Run the Bash script against the healthy server and verify that it creates a report.

### Evidence

#### Screenshot 9 — Output of `./scripts/linux-triage.sh` showing your Full Name and all five check results

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/9.png)

---

#### Screenshot 10 — Output showing the captured exit code and final summary

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/10.png)

---

### Notes

Answer the following in your own words:

**1. What is the overall status of your healthy baseline?**

The overall status was **HEALTHY**. All five checks passed:
- [PASS] Nginx service is active
- [PASS] Port 80 is listening
- [PASS] Local HTTP check returned status 200
- [PASS] Root disk usage is <XX>%
- [PASS] Available memory is <XXX> MB
The summary showed PASS: 5, WARN: 0, FAIL: 0. Because there were no failures, the server is
in a known-good state, and it's safe to continue to the incident simulation.

---

**2. Which exact Linux evidence proves the application is serving traffic?**

The line **`[PASS] Local HTTP check returned status 200`** is the strongest proof. The
script ran `curl` against `http://localhost`, and Nginx answered with HTTP status **200 OK**,
which means a real request received a successful response.
Two other checks support it:
- `[PASS] Nginx service is active`: the web server process is running, according to
  `systemctl`.
- `[PASS] Port 80 is listening`: the server is accepting connections on the HTTP port,
  according to `ss -ltn`.
Together, these show the full chain works: the service is running, the port is open, and
requests get successful responses. A running service alone wouldn't be enough, because it
could still return errors.

---

**3. Did your script return exit code 0 or 1? Explain why.**

It returned **0**. The script decides the exit code in `print_summary`:
- If `failure_count` is above 0, it returns **2**.
- Otherwise, if `warning_count` is above 0, it returns **1**.
- Otherwise, it returns **0**.
My run had 0 failures and 0 warnings, so the status was HEALTHY and the exit code was 0.
The value I captured with `script_exit_code=$?` matched the "Script Exit Code: 0" line in the
report. I had to capture `$?` immediately after running the script, because any other
command would overwrite it.
---

**4. What is the difference between a warning and a failure in this script?**

- **Failure (FAIL, exit code 2):** something is **broken right now**, and users are
  probably affected. For example, Nginx isn't running, port 80 isn't listening, HTTP
  doesn't return 200, or the disk is 90% or more full. This needs immediate action.
- **Warning (WARN, exit code 1):** the service still works, but a resource is getting close
  to a problem. For example, disk usage is 80–89%, or available memory is below 100 MB.
  Nothing is down yet, but if it's ignored, it could turn into a failure.
In short, a warning is an early signal to act soon, and a failure means the system is
already unhealthy and needs attention now. The thresholds are set in variables at the top
of the script (`disk_warning_threshold=80`, `disk_failure_threshold=90`,
`memory_warning_mb=100`), so they are easy to adjust.
---

# Task 6 — Create and Run the /linux-triage Skill

## Goal

Turn the Bash script into a reusable, manually invoked Agentic AI workflow.

### Evidence

#### Screenshot 11 — `SKILL.md` showing the frontmatter, allowed tool restrictions, and safety rules

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/11.png)


---

#### Screenshot 12 — `/linux-triage` output for the healthy server

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/12.png)

---

### Notes

Answer the following in your own words:

**1. Why does this skill have Bash, Read, and Grep, but not Write?**

The skill only needs to **collect and read evidence**, never change anything:
- **Bash:** to run `scripts/linux-triage.sh`, which gathers the health data.
- **Read:** to open `CLAUDE.md` and `reports/linux-health-report.txt`.
- **Grep:** to search the report for specific lines, like `[FAIL]` or `[WARN]`.
**Write** (and Edit) are left out on purpose, following the principle of **least
privilege**: give a tool only the permissions it needs for its job. `allowed-tools` lists the
tools the skill can use without asking me each time. Because Write isn't on the list, Claude
can't quietly create or change files during triage. Any attempt would need my approval, and
I would reject it.
The script itself does write the report file, but that happens inside the script I
reviewed. Claude doesn't write it directly.

---

**2. Why is `disable-model-invocation: true` useful for this skill?**

It means the skill **only runs when I type `/linux-triage` myself**. Claude can't decide on
its own to run it in the middle of another conversation just because the topic seems related.
This matters because the skill runs commands on the server. With manual invocation:
- I choose **when** triage happens, for example after a change or during an incident, not at
  random moments.
- Nothing runs on the server unexpectedly.
- The workflow is predictable and repeatable: same command, same steps, same report format.
It keeps a human in control of when the automation starts, which fits the project's safety
rules.

---

**3. What part is performed by Bash, and what part is performed by Claude?**

- **Bash (the script): Gather.** It runs the five checks (service, port 80, HTTP response,
  disk, memory), captures recent Nginx logs, marks each result PASS, WARN or FAIL, counts
  them, and writes everything to `reports/linux-health-report.txt` with an exit code. It is
  fast, reliable, and gives the same result every time.
- **Claude: Analyze and explain.** It reads the rules in `CLAUDE.md` and the report, points
  out any WARN or FAIL lines with the exact evidence, explains the most likely cause,
  suggests **one** safe recovery command and **one** verification command, and asks me to
  review and run them myself.
- **Me (the human): Approve, execute and verify.** I decide whether to run the recovery
  command, and I confirm the fix.
Bash is good at running exact checks the same way every time. Claude is good at reading the
results and explaining them clearly. Each does the part it is best at.

---

**4. Why is this better than asking Claude "Is my server healthy?" without giving it evidence?**

Claude can only **guess**. It might give general advice, or an answer that
sounds confident but has no connection to the actual server. That is risky during an
incident, when a wrong diagnosis leads to the wrong fix.
With this skill:
- **The answer is based on real data** collected from the server at that moment.
- **Every claim can be checked,** because Claude must quote the exact lines from the report.
- **It follows the rule "no root cause without supporting evidence",** so it doesn't make
  things up.
- **It is consistent:** the same checks run and the same report format comes out every
  time, so results can be compared over time.
- **It is safer:** the process is read-only, the recovery command is only a recommendation,
  and I run it myself.
In short, it turns a guess into an **evidence-based diagnosis** that I can verify.

---

# Task 7 — Simulate an Nginx Incident and Let the Skill Diagnose It

## Goal

Create a controlled service failure, gather evidence through Bash, and let Claude analyze the evidence without taking recovery action.

### Evidence

#### Screenshot 13 — Output showing Nginx is inactive and the HTTP request fails

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/13.png)

---

#### Screenshot 14 — `/linux-triage` output showing failed evidence, most likely cause, and a suggested recovery command

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/14.png)

---

#### Screenshot 15 — `incident-failure-report.txt` showing the failed checks and your Full Name

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/15.png)

---

### Notes

Answer the following in your own words:

**1. Which three checks failed?**

- **[FAIL] Nginx service is not active**: `systemctl is-active` reported that the service
  was not running.
- **[FAIL] Port 80 is not listening**: `ss -ltn` found no process listening on port 80.
- **[FAIL] Local HTTP check returned status 000**: `curl` couldn't connect to
  `http://localhost` at all, so no HTTP status came back. `000` means the connection
  failed, not that the server sent an error page.
The disk and memory checks still passed. That shows the server itself was healthy, and only
the web service was down. The overall status was **FAIL**, with exit code **2**.

---

**2. What evidence supports the conclusion that Nginx is unavailable?**

The evidence came from several independent sources, and they all agree:
- `systemctl is-active nginx` returned **`inactive`**.
- `curl -I --max-time 5 http://localhost` failed with **"Failed to connect to localhost
  port 80: Connection refused"**. Nothing was accepting connections.
- The report showed all three FAIL lines: service not active, port 80 not listening, and
  HTTP status 000.
- The **recent Nginx logs** in the report showed **"Stopping nginx.service"** and
  **"Stopped nginx.service"**. This proves the service was shut down cleanly, not crashed,
  which matches my manual `systemctl stop`.
- Disk and memory were PASS, which rules out resource exhaustion as the cause.
Together, this points to one clear cause: **the Nginx service was stopped**.

---

**3. Did Claude execute the recovery command? Why is that important?**

**No.** Claude recommended `sudo systemctl start nginx` and a verification command
(`systemctl is-active nginx` or `curl -I http://localhost`), and then asked me to review and
run them myself. It didn't use sudo or change the server.
This is important because:
- **Humans stay in control of production changes.** An AI shouldn't change a live system
  without approval.
- **Claude only sees the report.** A human may know things it doesn't. In this case, I
  stopped Nginx on purpose. In real life, it might have been stopped for maintenance, so
  starting it automatically could interfere with that work.
- **It follows the rules in CLAUDE.md and SKILL.md:** "Recommend a recovery command, but do
  not execute it" and "Never execute the recovery command."
- **It proves the safety design works.** The AI gave useful help without having the power
  to make things worse.

---

**4. Which phase of the Agentic Loop is represented by the Bash report?**

The **Gather** phase, meaning evidence collection. The script ran the five read-only checks,
captured the Nginx logs, and saved the facts in `linux-health-report.txt` (copied to
`incident-failure-report.txt`) without changing anything on the server. It records **what is
happening**, but doesn't try to interpret it.

---

**5. Which phase is represented by Claude's explanation?**

The **Analyze** phase. Claude read the evidence, pointed out the three FAIL checks, quoted
the exact report lines, explained the most likely cause (Nginx stopped), and proposed a safe
recovery command and a verification command. That proposal leads into the next phase, where
**I** approve and execute the fix and then **verify** the system again, completing the loop
set out in CLAUDE.md: gather, analyze, human approves and executes, verify.

---

# Task 8 — Recover Manually, Verify Again, and Write the Incident Summary

## Goal

Recover the service as the human operator and prove that the system is healthy again.

### Evidence

#### Screenshot 16 — Output showing Nginx is active and `curl -I http://localhost` returns 200 OK

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/16.png)

---

#### Screenshot 17 — Second `/linux-triage` output showing successful recovery with no FAIL results

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/17.png)
---

#### Screenshot 18 — Output of `ls -lah reports` showing both `incident-failure-report.txt` and `recovery-report.txt`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/18.png)

---

#### Screenshot 19 — `incident-summary.md` showing all required sections and your Full Name

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/19.png)

---

### Notes

Answer the following in your own words:

**1. What action did you execute manually?**

After reviewing Claude's recommendation, I ran:

`sudo systemctl start nginx`

I chose this command myself, in a separate regular terminal. Claude only suggested it and
didn't run it. I used `start`, not `restart`, because the evidence showed Nginx had been
cleanly stopped, not crashed or misconfigured, so starting it again was the smallest safe fix.

---

**2. What evidence proves that the service recovered?**

- `systemctl is-active nginx` returned **`active`**.
- `curl -I http://localhost` returned **`HTTP/1.1 200 OK`**, with the `Server: nginx`
  header.
- The second `/linux-triage` run showed all checks back to **PASS**:
  - [PASS] Nginx service is active
  - [PASS] Port 80 is listening
  - [PASS] Local HTTP check returned status 200
  - Disk and memory were PASS (or WARN), with **FAIL: 0**
- The overall status went from **FAIL (exit code 2)** back to **HEALTHY (exit code 0)**,
  matching the baseline from Task 5.
- The recent logs in the report now showed **"Started nginx.service"**.
Comparing `incident-failure-report.txt` with `recovery-report.txt` shows the change from
failed to healthy clearly.

---

**3. Why is the second triage run necessary?**

Running a fix command doesn't prove the problem is solved. The command might fail silently,
the service might start and then crash, or only part of the system might come back. The
second run **verifies** the recovery, using the same five checks as before, so the result
before and after can be compared directly.
It also:
- confirms **all** checks passed, not just the one I was focused on
- produces a saved `recovery-report.txt` as a record that the incident was resolved
- completes the Verify step of the agentic loop. An incident is only closed when the system
  is back to its known-good baseline.

---

**4. What could go wrong if an AI agent automatically restarted every failed service?**

- **It could hide the real problem.** A restart can make symptoms go away for a while, while
  the root cause (a bad config, memory leak or full disk) stays, and the service keeps
  failing again.
- **Restart loops:** a service with a broken config would fail, restart and fail again,
  over and over, filling the logs and wasting resources.
- **It could interfere with intentional stops.** A service might have been stopped on
  purpose for maintenance, a deployment, or to stop an active security attack. Restarting it
  could undo important work or reopen a security hole.
- **It could make things worse.** Restarting some services during an outage could corrupt
  data, drop active users, or put more load on systems that are already struggling.
- **No accountability.** Changes would happen with no human decision behind them, making it
  hard to know who changed what and why.
- **Wrong diagnosis at scale.** If the AI misreads the evidence, it would apply the wrong
  fix instantly, and possibly on many servers at once.

---

**5. In one sentence, explain the difference between using AI as a chatbot and using AI in this agentic workflow.**

As a chatbot, AI answers from general knowledge and guesses about a server it can't see,
while in this agentic workflow it follows defined rules, collects real evidence from the
server with a script, analyzes that evidence, and recommends a fix that a human approves,
runs and verifies.

---

# Incident Summary

Fill in all seven sections below in your own words.

**Full Name:** Siththi Waseema

**Date:** 27/9/2026

---
## 1. Reported Symptom
Explain what appeared to be broken.

## 2. Evidence Collected
List the failed Bash checks and the exact evidence they showed.

## 3. Most Likely Cause
Explain the cause using only the collected evidence.

## 4. Human-Approved Recovery Action
Write the command you reviewed and executed manually.

## 5. Verification
Explain which outputs proved that Nginx and the application recovered.

## 6. Safety Decision
Explain why the AI skill was allowed to gather and analyze evidence but was not allowed to restart the service.

## 7. Agentic Loop Mapping
Explain how this incident followed:

Gather -> Analyze -> Human Act -> Verify
---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/siththi-waseema-62a0b0187_dmibypravinmishra-siththiwaseema-devops-share-7509722011431223296-Td46/?utm_source=share&utm_medium=member_desktop&rcm=ACoAACv5boIBHr4DjAudB4kGRvqYrehYIp1o_Io


---

#### Screenshot — Published LinkedIn post

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2006/ss.png)

---

# GitHub Repository URL

Paste the URL of your GitHub folder or repository containing the assignment files here:

https://github.com/siththiwaseema/devops-micro-internship-pravinmishra.git

---

# Submission Instructions

- Add all required screenshots in your submission
- Full Name must be visible in required screenshots and the Bash report
- All written answers must be in your own words
- Do not expose sensitive information (keys, passwords, AWS account IDs, tokens)
- GitHub URL must be included in this document

---

# Completion Checklist

- [ ] Task 1: Healthy baseline confirmed, workspace created (Screenshots 1–2, Notes answered)
- [ ] Task 2: CLAUDE.md created with all four sections (Screenshot 3, Notes answered)
- [ ] Task 3: Five-check plan produced by Claude using read-only tools (Screenshot 4, Notes answered)
- [ ] Task 4: `linux-triage.sh` created, syntax validated, executable permission set (Screenshots 5–8, Notes answered)
- [ ] Task 5: Healthy-state report generated with no FAIL result (Screenshots 9–10, Notes answered)
- [ ] Task 6: `/linux-triage` skill created and run successfully on healthy server (Screenshots 11–12, Notes answered)
- [ ] Task 7: Nginx incident simulated, failed evidence captured, Claude did not execute recovery (Screenshots 13–15, Notes answered)
- [ ] Task 8: Nginx recovered manually, recovery verified, reports saved, incident summary complete (Screenshots 16–19, Notes answered)
- [ ] Incident summary contains all seven required sections
- [ ] LinkedIn post published and URL submitted
- [ ] Full Name visible in all required screenshots and the Bash report
- [ ] Skill does not have Write permission
- [ ] Skill did not execute any recovery commands
- [ ] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) — Agentic AI Track.*