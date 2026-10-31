# Assignment 6 — Building an AI-Assisted Git Safety Net (PR Ready Check)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

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

![](screenshots/Assign6/1.png)

---

### Notes

**1. Why create a dedicated branch instead of doing this work on main?**

A dedicated branch keeps this work isolated from `main`, so `main` always stays clean and in sync with upstream. This assignment deliberately stages risky changes (a fake secret and a debug statement), so if anything goes wrong I can simply delete the branch without affecting `main` or my other open work such as `feature-readme-update`. It also lets me open a focused Pull Request where reviewers see only the changes for this task, which is the standard team workflow: develop on a feature branch, review through a PR, and only then merge into `main`.

---

# Task 1 — Stage a Change With Realistic Risk

## Goal

On your own fork of this repository (the one you've been submitting your DMI work in since onboarding), create a new branch and stage a change that a real reviewer should catch: a hardcoded-looking secret and a leftover debug statement.

### Evidence

#### Screenshot 1 — Output of  `git status` showing the staged file on feature/ai-pr-ready

![](screenshots/Assign6/1.png)
![](screenshots/Assign6/2.png)

---

### Notes

**1. Why does this assignment use an obviously fake key instead of a real one?**

Because a real key should never be put into a Git repository, even for practice. Once a secret is committed and pushed, it stays in the Git history and can be copied by anyone who sees the repo, and bots constantly scan public GitHub repositories for leaked credentials and abuse them within minutes. Deleting the file later does not remove it from history, so the only safe fix would be to revoke and rotate the key. A fake key looks realistic enough to test whether the pre-commit hook and the AI review actually catch it, without risking a real account, data, or cloud bill.

---

# Task 2 — Write a Real Git Pre-Commit Hook

## Goal

Create a tracked, shareable pre-commit hook that blocks a commit containing secret-like patterns or files over 1MB.

### Evidence

#### Screenshot 2 — `hooks/pre-commit` open in VS Code showing the full script
![](screenshots/Assign6/3-4.png)

---

#### Screenshot 3 — Output of `git config core.hooksPath` confirming it points to `hooks`

![](screenshots/Assign6/3-4.png)

---

### Notes

**1. Why is `hooks/pre-commit` tracked in the repo instead of living only in `.git/hooks/`?**

Because the `.git/` folder is never committed or pushed, so a hook placed only in `.git/hooks/` exists on my machine alone — teammates who clone the repo would not get it, and it could be lost if I re-clone. Keeping the hook in a tracked `hooks/` folder means it is version-controlled like any other code: everyone gets the same script, changes to it are reviewed in Pull Requests, and its history is visible. Each developer then enables it once with `git config core.hooksPath hooks`, so the whole team shares one consistent safety check instead of relying on individual local setups.

---

**2. Compare this to `PreToolUse` from Week 2 Assignment 6. What does each one intercept, and what do they have in common?**

The Git `pre-commit` hook intercepts a **commit**: it runs when I type `git commit`, inspects the staged changes, and blocks the commit if it finds a secret-like pattern (such as an `AKIA...` key) or a file over 1MB. The Claude Code `PreToolUse` hook intercepts an **AI agent's tool call**: it runs before Claude executes a tool such as a Bash command, and in Week 2 it blocked dangerous commands like `terraform destroy`.

What they have in common: both are automatic safety gates that run **before** an action happens, not after. Both are simple scripts that inspect what is about to happen and use the exit code to decide — exit `0` lets it through, while a blocking exit code stops it (any non-zero exit for the Git hook, exit `2` for `PreToolUse`). Both enforce a fixed rule consistently, without relying on a human (or an AI) to remember to be careful, and both catch mistakes at the cheapest point — before a secret reaches Git history or a destructive command reaches real infrastructure.

---

# Task 3 — Prove the Hook Blocks the Risky Commit

## Goal

Attempt to commit the staged file from Task 1 and show the hook rejecting it.

### Evidence

#### Screenshot 4 — Terminal showing `git commit` rejected with the hook's "BLOCKED" message naming the exact file

![](screenshots/Assign6/5.png)


---

### Notes

**1. Which line in `hooks/pre-commit` matched your fake key, and why did it match?**

Line 7 matched it:

```bash
if git diff --cached -- "$file" | grep -qE 'AKIA[0-9A-Z]{16}|-----BEGIN (RSA|OPENSSH|PRIVATE) KEY-----'; then
```

This line takes the staged diff of each file and searches it with the regular expression `AKIA[0-9A-Z]{16}`. My fake key `AKIAABCDEFGHIJKLMNOP` is the text `AKIA` followed by exactly 16 uppercase letters (`ABCDEFGHIJKLMNOP`), which is the same shape as a real AWS Access Key ID. The pattern matched, so the hook printed `BLOCKED: possible secret in scripts/notify.sh`, set `blocked=1`, and exited with code `1`, which made Git reject the commit.

---

**2. Could this hook have caught a poorly-named variable that stores a secret without the `AKIA` prefix? What does that tell you about the limits of a fixed rule like this?**

No. The hook only looks for two exact text patterns: an `AKIA` key or a `-----BEGIN ... KEY-----` header. A secret such as `x=9f8a7b6c5d4e3f2a1b0c`, a database password, a GitHub token, or an AWS *Secret* Access Key (which has no `AKIA` prefix) stored in a vaguely named variable would pass straight through, because the hook does not understand what the value means — it only matches text.

This shows that a fixed rule is fast, cheap, and reliable for the patterns it knows, but it can only catch what someone thought to write a pattern for. It misses anything new or unusually formatted, and it can also give false positives. That is why it should be one layer, not the only one: it should be combined with broader secret scanners (such as `gitleaks` or GitHub secret scanning), AI-assisted review that can reason about context, and human code review.

---

# Task 4 — Build the `/pr-ready` Skill

## Goal

Create a manually invoked Claude Code skill that reads your staged changes and produces a PR-readiness report and a draft PR description — without writing, committing, or pushing anything itself.

### Evidence

#### Screenshot 5 — `SKILL.md` frontmatter showing `allowed-tools: Bash, Read, Grep` (no `Write`) and `disable-model-invocation: true`

![](screenshots/Assign6/6-7.png)

---

#### Screenshot 6 — `/pr-ready` output while the risky file is still staged, showing it flagged the secret and/or debug statement

![](screenshots/Assign6/6-7.png)

---

### Notes

**1. Why does `/pr-ready` have `Bash` and `Read` but not `Write`?**

Because `/pr-ready` is meant to be a reviewer, not an author. It needs `Bash` to run read-only Git commands like `git diff --cached` and `git status`, and `Read` (plus `Grep`) to open and search files for context. Leaving out `Write` (and `Edit`) means the skill physically cannot change my files — it can only look and report. This follows the principle of least privilege: give the AI only the permissions the job requires. The output is a draft that a human reviews and decides on, so the person stays in control of what actually gets changed, committed, and pushed. It is the same "look but don't touch" idea as the restricted skill in Week 2.

---

**2. The pre-commit hook and `/pr-ready` both looked at the same staged diff. Did they flag the same things? What did one catch that the other didn't?**

They overlapped on one thing: both flagged the fake AWS key `AKIAABCDEFGHIJKLMNOP` in `scripts/notify.sh`. The pre-commit hook caught it with its `AKIA[0-9A-Z]{16}` regex and blocked the commit, and `/pr-ready` reported it as a credential-shaped string.

But `/pr-ready` caught several things the hook could not:
- the leftover `echo "DEBUG: ..."` statement — and that it actually **prints the secret** into terminal and CI logs
- that `notify.sh` has no real notification logic, so its purpose is unclear
- that the change has no notes or documentation
- that the `hooks/` folder was still untracked and should go in a separate commit

The hook, on the other hand, gives a hard guarantee: it runs automatically on every commit and actually **blocks** it, while `/pr-ready` only runs when I invoke it and produces advice that a human must act on.

So the hook is a fast, strict, but narrow rule that only knows patterns someone wrote in advance, while the AI review understands context and intent and finds a broader set of problems, but is not an enforced gate. Used together, they cover each other's weaknesses.

---

# Task 5 — Fix the Issues and Re-Verify

## Goal

Remove the secret and debug statement, then prove both gates now pass clean.

### Evidence

#### Screenshot 7 — `git commit` succeeding after the fix (no BLOCKED message)

![](screenshots/Assign6/8.png)

---

#### Screenshot 8 — Second `/pr-ready` run showing a clean risk report and a drafted PR title + description

![](screenshots/Assign6/9.png)
---

### Notes

**1. What exactly did you change to satisfy the pre-commit hook?**

I edited `scripts/notify.sh` and made two changes:

1. **Removed the hardcoded key** `AWS_ACCESS_KEY_ID=AKIAABCDEFGHIJKLMNOP`. This was the line the hook's `AKIA[0-9A-Z]{16}` pattern matched. I replaced it with `: "${AWS_ACCESS_KEY_ID:?AWS_ACCESS_KEY_ID must be set in the environment}"`, so the script now reads the key from the environment and stops with a clear error if it is missing — the credential is never stored in the repository.
2. **Removed the debug statement** `echo "DEBUG: token is $AWS_ACCESS_KEY_ID"`, which would have printed the secret into terminal and CI logs. I replaced it with a harmless status message, `echo "Sending notification..."`.

The final file:

```bash
#!/bin/bash
# Sends a notification. Credentials come from the environment, never hardcoded.
: "${AWS_ACCESS_KEY_ID:?AWS_ACCESS_KEY_ID must be set in the environment}"
echo "Sending notification..."
```

Strictly, only the first change was needed to pass the hook, because the hook checks for secret patterns, not debug statements. I removed the debug echo as well because `/pr-ready` flagged it. After re-staging, `git commit -m "add notification script"` succeeded with no `BLOCKED` message (commit `17a16ae`).

---

# Task 6 — Push and Open a Pull Request Using the AI Draft

## Goal

Push your branch and open a real Pull Request, using `/pr-ready`'s drafted title and description as your starting point — read it critically and edit before you use it.

**Important:** Open this Pull Request with base repository set to **your own fork** — not the shared upstream `pravinmishraaws/devops-micro-internship-pravinmishra` repository. This assignment's hook and skill files are your own practice work, not a change meant for the shared class repo.

### Evidence

#### Screenshot 9 — Your Pull Request showing the base repository is your own fork, plus the title and description, with the `/pr-ready` draft visible for comparison (paste it in the PR conversation or your notes below)

![](screenshots/Assign6/10.png)

---

#### PR Link

https://github.com/HastiKabiri/devops-micro-internship-interviews/pull/1

---

### Notes

**1. What, if anything, did you edit in the AI's drafted PR description before using it? Why?**

I kept the title (`feat: add notification script with environment-based credentials`) because it was accurate, but I edited the description in three ways:

- **Added a "Verified" line** stating that the pre-commit hook passes and the second `/pr-ready` run reported no risks, so the reviewer knows what was actually tested instead of having to assume it.
- **Added a note that the `hooks/` folder is not part of this PR** and will be committed separately. The AI draft did not mention it, and a reviewer could otherwise wonder where the hook is or expect it in this PR.
- **Made the wording more precise and split it into paragraphs**: the original said both issues were "caught by the pre-commit hook and the /pr-ready review", but the hook only blocks secret patterns — it was `/pr-ready` that flagged the debug echo. I corrected that and separated "what changed" from "how it was verified" so it is easier to read.

I made these edits because the PR description is my statement to the reviewer, so it must be accurate and complete, not just plausible-sounding.

---

**2. If you had blindly copy-pasted the AI's draft without reading it, what could go wrong?**

The PR would contain claims I had not checked. In this case the draft slightly overstated what the pre-commit hook does (implying it caught the debug echo, which it cannot) and left out that `hooks/` was excluded — small things, but they would mislead the reviewer. More generally, an AI draft can describe changes that are not in the diff, miss changes that are, invent test results, or even repeat sensitive content such as a key or internal detail from the code into a public PR description. Reviewers rely on the description to understand the change, so wrong information can lead to a bad merge, wasted review time, and a loss of trust in my PRs. Since my name is on the PR, I am responsible for every line of it — the AI is a drafting assistant, not the author.

---

**3. Why does this PR need to target your own fork instead of the shared upstream repository?**

Because this branch is a practice exercise, not a real contribution. It contains a demo script, and its history includes an assignment about fake secrets — that does not belong in the shared `pravinmishraaws/devops-micro-internship-interviews` repository. Opening it against upstream would send noise to the maintainer and to every other student watching the repo, and many students doing the same would flood it with PRs that are never meant to be merged. Targeting my own fork lets me practise the full workflow — push, open a PR, review, edit the description, and merge — safely in a space I own and control, without affecting anyone else.

---

# Task 7 — Map the Workflow to the Agentic Loop

## Goal

Explain this assignment's workflow using the same Gather → Analyze → Human Act → Verify structure from Week 3.

### Notes

**1. Which step(s) represent Gather?**

Collecting the facts about what is about to be committed: staging the change with `git add scripts/notify.sh` and inspecting it with `git status` (Task 1). Inside `/pr-ready`, Gather is its first step — running `git diff --cached` and `git status` to see exactly what is staged. The pre-commit hook gathers too, by reading the staged file list with `git diff --cached --name-only` and each file's diff and size.

---

**2. Which step(s) represent Analyze?**

Two analysers looked at the gathered diff:

- **The pre-commit hook** (Tasks 2–3) analysed it with fixed rules — the `AKIA[0-9A-Z]{16}` / private-key regex and the 1MB size limit — and blocked the commit when the fake key matched.
- **The `/pr-ready` skill** (Task 4) analysed it with AI reasoning — it reported the credential-shaped string, the debug echo that printed the secret, the missing notes, and the untracked `hooks/` folder, then drafted a PR title and description.

Both are read-only: they report and recommend, but do not change anything themselves.

---

**3. Which step is Human Act, and why must a human — not Claude — run `git commit`, `git push`, and open the PR?**

Human Act is Tasks 5–6: I decided how to fix `scripts/notify.sh`, ran `git commit`, ran `git push origin feature/ai-pr-ready`, and opened the Pull Request after reading and editing the AI's draft.

A human must do these because they are the points where a change becomes permanent and shared. A commit records history, and a push sends it to GitHub, where it may be public and very hard to fully take back — a leaked secret, for example, has to be rotated, not just deleted. Opening a PR is a statement to reviewers made under my name. Those decisions need human judgement and accountability, so the AI is limited to drafting (`/pr-ready` has no `Write` access and is told never to commit, push, or open PRs) and the human stays in control of anything irreversible.

---

**4. Which step is Verify?**

Task 5's re-check after the fix: re-staging the corrected file, the pre-commit hook letting `git commit -m "add notification script"` succeed with no `BLOCKED` message, and the second `/pr-ready` run returning a clean risk report. Task 6 adds a final check — confirming on GitHub that the PR's base repository is my own fork and that the title and description are accurate.

---

**5. In one or two sentences: why do you need *both* the fixed-rule pre-commit hook and the AI skill? Isn't one enough?**

No — the pre-commit hook is an automatic, enforced gate that reliably blocks known patterns on every commit but is blind to anything it has no rule for, while the AI skill understands context and catches broader issues like the debug echo or missing notes but only gives advice when someone runs it. Together they give a hard guarantee for known risks plus intelligent review for everything else, with a human making the final call.

---

# Task 8 — LinkedIn Post

## Goal

Publish a LinkedIn post summarizing what you built and what you learned about combining fixed-rule safety checks with AI-assisted review.

### Evidence

#### LinkedIn Post URL

https://lnkd.in/p/gMn_EsbD

---

## Key Learnings

Add 3-5 bullet points on what you learned this week.

- **Git workflow fundamentals:** tracking, staging, and committing with small, atomic, well-named commits (`feat:`, `docs:`, `chore:`), and using feature branches so `main` stays clean and every change is isolated and easy to review or throw away.
- **Collaborating on GitHub:** forking, cloning, keeping a fork in sync with `upstream` (fetch, merge, rebase), pushing a branch to `origin`, and opening a Pull Request with the correct base and head — plus setting up HTTPS authentication safely with Git Credential Manager and a Personal Access Token.
- **Secrets must never reach Git:** once a key is committed and pushed it lives in the history and must be rotated, not just deleted. Credentials belong in the environment or a secrets manager, never hardcoded, and debug statements that print them are just as dangerous.
- **Git hooks are automatic safety gates:** a tracked `pre-commit` hook (shared via `core.hooksPath`) blocks risky commits before they happen — the same "intercept before the action" idea as Claude Code's `PreToolUse` hook — but a fixed rule only catches the patterns it was written for.
- **AI review helps, humans decide:** the read-only `/pr-ready` skill found issues the hook could not and drafted a PR description, but the draft still needed reading and correcting. Layering fixed rules, AI review, and human judgement works far better than any one of them alone, and a human must always own `git commit`, `git push`, and the PR.

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

https://github.com/HastiKabiri/devops-micro-internship-interviews

---

# Completion Checklist

- [✅] Branch `feature/ai-pr-ready` created with a staged file containing a fake secret and a debug statement
- [✅] `hooks/pre-commit` created and tracked in the repo (not only in `.git/hooks/`)
- [✅] `core.hooksPath` configured to point at `hooks/`
- [✅] Pre-commit hook shown blocking the risky commit
- [✅] `.claude/skills/pr-ready/SKILL.md` created with correct `allowed-tools` (no `Write`) and `disable-model-invocation: true`
- [✅] `/pr-ready` run against the risky diff and shown flagging issues
- [✅] Risky file fixed; `git commit` succeeds cleanly
- [✅] `/pr-ready` re-run showing a clean report and drafted PR title/description
- [✅] Pull Request opened using the AI draft as a starting point, with your own fork as the base repository (not upstream), PR link included
- [✅] Agentic Loop mapping (Task 7) completed in your own words
- [✅] LinkedIn post published and URL submitted
- [✅] All required screenshots added
- [✅] GitHub repository URL provided

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

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
