# Assignment 6 — Building an AI-Assisted Git Safety Net (PR Ready Check)

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In Week 2 you built Claude Code hooks that block a dangerous action *before* it happens (`PreToolUse`), and a restricted skill that could look but not touch (`allowed-tools` without `Write`). In this assignment you will discover that Git has the exact same idea, decades older: a **pre-commit hook** that blocks a commit before it's created.

You will build both halves of a real "PR Ready" workflow:

1. A **Git hook that follows fixed rules** — scans staged changes for hardcoded secrets and oversized files and refuses the commit. No AI involved, no guessing, just a rule that gives the same answer every time.
2. A **restricted Claude Code skill** (`/pr-ready`) that reads your staged diff and drafts a Pull Request title, description, and a short list of things worth a second look — the kind of judgment a fixed rule can't make (mixed changes, missing context, unclear intent). The skill never commits, pushes, or opens the PR. You do that yourself, using its draft as a starting point.

This mirrors the Agentic Loop from Week 3's Linux triage assignment: **Gather → Analyze → Human Act → Verify**. The hook and the skill both gather and analyze; only you act.

---

# Task 0 — Confirm Your Fork and Create a Feature Branch

## Goal

Confirm you are working in your own fork, then create a dedicated branch for this assignment.

### Evidence

#### Screenshot 1 — Output of git remote -v and git branch showing the new branch

![Week 04–git-and-github](screenshots/Assignment%2006/1.png)

---

### Notes

**1. Why create a dedicated branch instead of doing this work on main?**

A dedicated branch keeps this work separate from main, which should always hold clean, working code. In this assignment, I deliberately stage a file containing a fake secret and a debug statement, so I definitely don't want that anywhere near main. If something goes wrong on the feature branch, I can fix it or delete the branch without affecting anyone else.

It also makes reviewing easier. A branch groups all the changes for one task, so when I open a Pull Request, the reviewer sees only what's related to this work. Nothing gets merged into main until it has been checked and approved.

---

# Task 1 — Stage a Change With Realistic Risk

## Goal

On your own fork of this repository (the one you've been submitting your DMI work in since onboarding), create a new branch and stage a change that a real reviewer should catch: a hardcoded-looking secret and a leftover debug statement.

### Evidence

#### Screenshot 1 — Output of  `git status` showing the staged file on feature/ai-pr-ready

![Week 04–git-and-github](screenshots/Assignment%2006/2.png)

---

### Notes

**1. Why does this assignment use an obviously fake key instead of a real one?**

The point of the exercise is to test whether the hook can catch something that looks like a secret, not to handle a real one. A fake key follows the same pattern as a real AWS key (it starts with AKIA and has the right length), so it triggers the check without any actual risk.

Using a real key would be dangerous. Even if the commit gets blocked, the key has still been typed into a file, staged, and possibly shown in screenshots or sent to an AI tool for review. If it ever reached GitHub, bots scan public repositories for keys within minutes and could misuse it. With a fake key, nothing bad can happen, even if something goes wrong.

---

# Task 2 — Write a Real Git Pre-Commit Hook

## Goal

Create a tracked, shareable pre-commit hook that blocks a commit containing secret-like patterns or files over 1MB.

### Evidence

#### Screenshot 2 — `hooks/pre-commit` open in VS Code showing the full script

![Week 04–git-and-github](screenshots/Assignment%2006/3.png)

---

#### Screenshot 3 — Output of `git config core.hooksPath` confirming it points to `hooks`

![Week 04–git-and-github](screenshots/Assignment%2006/4.png)

---

### Notes

**1. Why is `hooks/pre-commit` tracked in the repo instead of living only in `.git/hooks/`?**

Git never tracks anything inside the .git folder, so a hook saved in .git/hooks/ only exists on my own computer. If a teammate clones the repo, they don't get it, and their commits aren't checked at all.

By keeping the hook in a normal hooks/ folder, it's committed and shared like any other file, so everyone on the team gets the same check. Setting core.hooksPath to hooks then tells Git to run the hooks from that folder. It also means the hook can be reviewed and improved through Pull Requests, and every change to it is recorded in the history, just like the rest of the code.

---

**2. Compare this to `PreToolUse` from Week 2 Assignment 6. What does each one intercept, and what do they have in common?**

The pre-commit hook intercepts a Git commit. It runs just before the commit is created, checks the staged changes, and stops the commit if it finds a secret or a file over 1MB.

The PreToolUse hook intercepts a Claude Code action. It runs just before Claude uses a tool, such as running a Bash command or editing a file, and can block that action if it breaks a rule.

What they have in common is that both act as a gatekeeper at the moment before something happens, not after. Both follow fixed rules rather than making judgement calls, and both can block the action by exiting with an error. The idea is the same: it's much easier to stop a mistake before it happens than to clean it up afterwards.

---

# Task 3 — Prove the Hook Blocks the Risky Commit

## Goal

Attempt to commit the staged file from Task 1 and show the hook rejecting it.

### Evidence

#### Screenshot 4 — Terminal showing `git commit` rejected with the hook's "BLOCKED" message naming the exact file

![Week 04–git-and-github](screenshots/Assignment%2006/4.png)

---

### Notes

**1. Which line in `hooks/pre-commit` matched your fake key, and why did it match?**

if git diff --cached -- "$file" | grep -qE 'AKIA[0-9A-Z]{16}|-----BEGIN (RSA|OPENSSH|PRIVATE) KEY-----'; then


It matched because of the first part of the pattern, AKIA[0-9A-Z]{16}. This looks for the letters AKIA followed by exactly 16 uppercase letters or numbers, which is the format AWS uses for access key IDs.

My fake key was AKIAABCDEFGHIJKLMNOP. It starts with AKIA, and the rest, ABCDEFGHIJKLMNOP, is exactly 16 uppercase letters, so it fits the pattern perfectly. The hook doesn't know or care that the key is fake. It only checks the shape of the text, so anything that looks like an AWS key gets blocked.
---

**2. Could this hook have caught a poorly-named variable that stores a secret without the `AKIA` prefix? What does that tell you about the limits of a fixed rule like this?**

No. The hook only looks for two exact patterns: text starting with AKIA followed by 16 characters, and private key headers like -----BEGIN RSA KEY-----. A password, API token or database connection string stored in a variable like x = "s3cr3tP@ss" doesn't match either pattern, so it would pass straight through without any warning.

This shows that a fixed rule can only catch what it has been told to look for. It's fast and reliable, and it gives the same answer every time, but it has no understanding of what the code actually means. It can also be fooled easily, for example by splitting a key across two lines. That's why it works best alongside something that can use judgement, like the /pr-ready skill or a human reviewer, which can notice that a value looks sensitive even when it doesn't match a known pattern.

---

# Task 4 — Build the `/pr-ready` Skill

## Goal

Create a manually invoked Claude Code skill that reads your staged changes and produces a PR-readiness report and a draft PR description — without writing, committing, or pushing anything itself.

### Evidence

#### Screenshot 5 — `SKILL.md` frontmatter showing `allowed-tools: Bash, Read, Grep` (no `Write`) and `disable-model-invocation: true`

![Week 04–git-and-github](screenshots/Assignment%2006/5.png)

---

#### Screenshot 6 — `/pr-ready` output while the risky file is still staged, showing it flagged the secret and/or debug statement

![Week 04–git-and-github](screenshots/Assignment%2006/6.png)

---

### Notes

**1. Why does `/pr-ready` have `Bash` and `Read` but not `Write`?**

The skill's job is only to review changes and give advice, not to change anything. It needs Bash to run git diff --cached and git status and see what's staged, and Read to open files for more context. That's all it needs to do the review.

Leaving out Write means it can't edit or create files, so it can't quietly "fix" my code or change something I didn't ask it to. This keeps me in control, the AI points out problems and drafts the PR description, but I decide what to change and make the changes myself.

It's worth noting that Bash is still powerful. In theory, it could run commands that change files or run git commit. That's why the skill's instructions also say never to commit, push or edit files, and why I should still check any tool request before approving it, rather than relying on the tool list alone.

---

**2. The pre-commit hook and `/pr-ready` both looked at the same staged diff. Did they flag the same things? What did one catch that the other didn't?**

They overlapped on one thing: both flagged the hardcoded AWS key in scripts/notify.sh.

The hook stopped there. It found the AKIA pattern, printed BLOCKED: possible secret in scripts/notify.sh, and rejected the commit. It said nothing about the debug line, because it has no rule for that.

/pr-ready caught more. As well as the key, it flagged the echo "DEBUG: token is $AWS_ACCESS_KEY_ID" line as leftover debug code. It also pointed out that this line would print the secret into logs or terminal output, which is a second way for the key to leak. It could explain why each issue mattered and suggest how to fix it, which the hook can't do.

On the other hand, the hook has something the skill doesn't: it actually blocked the commit. /pr-ready only gives advice, so if I ignored it, nothing would stop me committing the file. The hook is a hard gate that runs every time, in milliseconds, and gives the same result. The AI gives a broader, more thoughtful review, but it can't enforce anything, and it might not flag the same things every time.

---

# Task 5 — Fix the Issues and Re-Verify

## Goal

Remove the secret and debug statement, then prove both gates now pass clean.

### Evidence

#### Screenshot 7 — `git commit` succeeding after the fix (no BLOCKED message)

![Week 04–git-and-github](screenshots/Assignment%2006/7.png)

---

#### Screenshot 8 — Second `/pr-ready` run showing a clean risk report and a drafted PR title + description

![Week 04–git-and-github](screenshots/Assignment%2006/8.png)

---

### Notes

**1. What exactly did you change to satisfy the pre-commit hook?**


I removed the two risky lines from scripts/notify.sh:

The hardcoded key (AWS_ACCESS_KEY_ID=AKIAABCDEFGHIJKLMNOP). This was what the hook blocked. Instead of storing the key in the code, the script now expects it to come from an environment variable, and stops with a clear error if it isn't set.
The debug line (echo "DEBUG: token is $AWS_ACCESS_KEY_ID"). The hook didn't block this, but /pr-ready flagged it because it would print the secret to the screen or logs. I replaced it with a plain message that doesn't show any sensitive data.
---

# Task 6 — Push and Open a Pull Request Using the AI Draft

## Goal

Push your branch and open a real Pull Request, using `/pr-ready`'s drafted title and description as your starting point — read it critically and edit before you use it.

**Important:** Open this Pull Request with base repository set to **your own fork** — not the shared upstream `pravinmishraaws/devops-micro-internship-pravinmishra` repository. This assignment's hook and skill files are your own practice work, not a change meant for the shared class repo.

### Evidence

#### Screenshot 9 — Your Pull Request showing the base repository is your own fork, plus the title and description, with the `/pr-ready` draft visible for comparison (paste it in the PR conversation or your notes below)

![Week 04–git-and-github](screenshots/Assignment%2006/9.png)

---

#### PR Link

https://github.com/siththiwaseema/devops-micro-internship-interviews/pulls

---

### Notes

**1. What, if anything, did you edit in the AI's drafted PR description before using it? Why?**

I used the AI's draft as a starting point, but I made a few changes before using it:

I added context the AI couldn't know. The draft described the changes correctly, but it didn't explain why they existed. I added that this PR is part of DMI Week 4 Assignment 6, and that the aim is to show a pre-commit hook and an AI review skill working together.
I made sure all the files were mentioned. The draft focused mainly on scripts/notify.sh, but the PR also adds hooks/pre-commit and the /pr-ready skill. I listed those clearly, so a reviewer knows exactly what's included.
I clarified that the key was never real. The history of this branch includes a blocked attempt with a fake AWS key, so I noted that no real credentials were used at any point.
I adjusted the wording to sound like me. Some phrases were more formal than I'd normally write, so I made them simpler and more direct.

I made these edits because the AI only sees the staged diff. It doesn't know the purpose of the work, what happened before, or who will read the PR. As the person opening the PR, I'm responsible for making sure the description is accurate and complete.
---

**2. If you had blindly copy-pasted the AI's draft without reading it, what could go wrong?**

The description might not match what the PR actually changes, and that's exactly the problem from the scenario, where a PR described as a "minor copy fix" quietly changed a database connection string. The AI could leave out a file, describe a change wrongly, or make something risky sound harmless. A reviewer who trusts the description might then approve something they didn't really understand.

The draft could also include things that shouldn't be there, such as details copied from the diff, like a secret or an internal file path, or confident claims that aren't true, such as saying something was tested when it wasn't.

Finally, my name goes on the PR, not the AI's. If the description is wrong, I'm the one responsible. Reading and editing it first is how I make sure I actually understand my own change and can stand behind what it says.
---

**3. Why does this PR need to target your own fork instead of the shared upstream repository?**

This PR is a practice exercise, not a real contribution. It adds a demo script, a hook and a skill that only make sense for my assignment. If I opened it against Pravin's upstream repository, it would add noise to a repo that many students share, and the maintainer would have to close it.

The upstream repo is also used by hundreds of students, so if everyone sent their practice PRs there, it would quickly become cluttered with unrelated changes. It could also accidentally introduce things like core.hooksPath changes or test scripts that other people don't want.

Targeting my own fork lets me practise the full workflow — push, open a PR, review the description — in a safe space that only I control. It's like testing in a staging environment before going to production: I can make mistakes without affecting anyone else.

---

# Task 7 — Map the Workflow to the Agentic Loop

## Goal

Explain this assignment's workflow using the same Gather → Analyze → Human Act → Verify structure from Week 3.

### Notes

**1. Which step(s) represent Gather?**

Gather is where facts about the change are collected, without making any judgement yet. In this assignment, that happens in two places:

Staging the change (Task 1). Running git add scripts/notify.sh and git status defines exactly what is going to be checked. The staged diff is the "evidence" that both tools work from.
The tools reading the staged diff. The pre-commit hook runs git diff --cached --name-only to list the staged files, then git diff --cached and git cat-file -s to get each file's content and size. The /pr-ready skill does the same thing by running git diff --cached and git status before it writes anything.

---

**2. Which step(s) represent Analyze?**

The pre-commit hook (Task 3). It compares each staged file against its fixed rules: does the content match the AKIA or private key pattern, and is the file over 1MB? The result is a simple yes or no. In my case, it found the fake key and decided to block the commit.
The /pr-ready skill (Task 4). It reads the same diff, but uses judgement rather than fixed patterns. It spotted the hardcoded key and the debug line that printed it, explained why each one was risky, and drafted a PR title and description based on what the change actually does.

---

**3. Which step is Human Act, and why must a human — not Claude — run `git commit`, `git push`, and open the PR?**

Human Act is where I take the findings and do something about them. In this assignment, that's Task 5 and Task 6: I edited scripts/notify.sh to remove the key and the debug line, re-staged it, ran git commit, pushed the branch with git push, and opened the Pull Request using an edited version of the AI's draft.

These actions need to be done by a human because they change things that other people rely on. Once a commit is pushed, it's in the shared history, and removing a leaked secret from it is difficult. A Pull Request asks the team to trust and review a change. If Claude did these steps by itself, a mistake in its judgement, such as missing a secret or misdescribing a change, would go straight through with nobody checking it.

AI can also be wrong while sounding completely confident. Keeping the final action with a human means someone who understands the purpose of the change, and who is accountable for it, always makes the decision. The AI advises, and I decide and act.

---

**4. Which step is Verify?**

Verify is where I check again, after making the fix, that the problem is actually gone. I don't just assume my changes worked. In this assignment, that happens in Task 5:

The pre-commit hook runs again when I retry git commit. This time, it finds no AKIA pattern and no oversized files, so the commit goes through with no BLOCKED message. That's proof the secret has been removed from the staged changes.
I run /pr-ready a second time on the fixed version. It comes back with a clean risk report, with no secret and no debug statement, and drafts a PR title and description.

---

**5. In one or two sentences: why do you need *both* the fixed-rule pre-commit hook and the AI skill? Isn't one enough?**

No, one isn't enough. The hook is a reliable gate that blocks known dangers like AWS keys every time, but it can only catch patterns it has been told about. The AI can spot problems that no rule covers, such as debug code, mixed changes or a misleading description, but it can only give advice and might miss things, so together they cover each other's weaknesses.

---

# Task 8 — LinkedIn Post

## Goal

Publish a LinkedIn post summarizing what you built and what you learned about combining fixed-rule safety checks with AI-assisted review.

### Evidence

#### LinkedIn Post URL

https://www.linkedin.com/posts/siththi-waseema-62a0b0187_dmibypravinmishra-git-github-share-7510669498052431872-KG6U/?utm_source=share&utm_medium=member_desktop&rcm=ACoAACv5boIBHr4DjAudB4kGRvqYrehYIp1o_Io

---

## Key Learnings


Git pre-commit hook in plain Bash, plus a read-only Claude Code skill called /pr-ready. The hook blocked my commit the moment it spotted a hardcoded AWS-style key. /pr-ready flagged the same key, and also caught a debug line that would have printed it into the logs. 

Together, showed me how a fixed-rule check and an AI-assisted review work side by side, with me approving every Git action. 

---

# Submission Instructions

- Ensure `hooks/pre-commit` and `.claude/skills/pr-ready/SKILL.md` are committed to your GitHub repository
- Add all required screenshots to your submission
- All written answers must be in your own words
- Do not use a real secret or credential anywhere in your submission — the fake key in Task 1 is intentional and must stay clearly fake
- Open your Pull Request against your own fork, not the shared upstream repository
- Push your final changes to your forked repository
- Include your PR link and LinkedIn post URL

---

## GitHub Repository URL

Paste your forked repository URL here:

https://github.com/siththiwaseema/devops-micro-internship-interviews.git

---

# Completion Checklist

- [ ] Branch `feature/ai-pr-ready` created with a staged file containing a fake secret and a debug statement
- [ ] `hooks/pre-commit` created and tracked in the repo (not only in `.git/hooks/`)
- [ ] `core.hooksPath` configured to point at `hooks/`
- [ ] Pre-commit hook shown blocking the risky commit
- [ ] `.claude/skills/pr-ready/SKILL.md` created with correct `allowed-tools` (no `Write`) and `disable-model-invocation: true`
- [ ] `/pr-ready` run against the risky diff and shown flagging issues
- [ ] Risky file fixed; `git commit` succeeds cleanly
- [ ] `/pr-ready` re-run showing a clean report and drafted PR title/description
- [ ] Pull Request opened using the AI draft as a starting point, with your own fork as the base repository (not upstream), PR link included
- [ ] Agentic Loop mapping (Task 7) completed in your own words
- [ ] LinkedIn post published and URL submitted
- [ ] All required screenshots added
- [ ] GitHub repository URL provided

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
