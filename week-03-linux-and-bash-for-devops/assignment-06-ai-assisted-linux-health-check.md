# Assignment 6 — Build an AI-Assisted Linux Health Check (AI-Assisted Linux Incident Triage)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will build a read-only Bash triage script that checks the health of your Ubuntu server and Nginx application, connect it to Claude Code as a reusable `/linux-triage` skill, simulate a controlled Nginx incident, use the skill to gather and analyze evidence, recover the service manually, and verify recovery. The workflow follows the Agentic Loop: Gather → Analyze → Human Act → Verify.

---

# Task 1 — Confirm the Healthy Baseline and Create the Workspace

## Goal

Confirm that Nginx and the React application are healthy before building the automation.

### Evidence

#### Screenshot 1 — Output of `systemctl is-active nginx`, `ss -ltn | grep ':80'`, and `curl -I http://localhost`

![](screenshots/assign%206/1.png)

---

#### Screenshot 2 — Output of `pwd` and `find . -maxdepth 4 -type d | sort` showing the workspace folder structure

![](screenshots/assign%206/2.png)

---

### Notes

Answer the following in your own words:

**1. What proves that Nginx is running?**

`systemctl is-active nginx` returns `active`, which means systemd reports the Nginx service as running. If it were stopped, the command would return `inactive` or `failed` instead.

---

**2. What proves that the server is listening for HTTP traffic?**

`ss -ltn | grep ':80'` shows a line in the `LISTEN` state on port 80 (`0.0.0.0:80`), which proves a process is waiting for HTTP connections. `curl -I http://localhost` then returns an `HTTP/1.1 200 OK` response from Nginx, which proves the server is not only listening but actually answering requests.

---

**3. Why must you capture a healthy baseline before simulating an incident?**

A healthy baseline shows what "normal" looks like on this server. Without it, I could not tell whether a result during the incident is a real problem or just how the server always behaves. Comparing the baseline with the failed state makes the cause clear, and it also proves my lab setup was working before I broke anything on purpose, so the failure is caused by the simulation and not by a broken setup.

---

# Task 2 — Create Project Context and Safety Rules in CLAUDE.md

## Goal

Tell Claude exactly what this project does and what it is not allowed to do.

### Evidence

#### Screenshot 3 — CLAUDE.md open in VS Code showing all four sections (Project Overview, Incident Workflow, Safety Rules, Output Rules)

![](screenshots/assign%206/3.png)

---

### Notes

Answer the following in your own words:

**1. Why should Claude receive project-specific operational rules?**

Claude does not know my server, my project or my limits unless I tell it. `CLAUDE.md` gives it the context (what the project does), the workflow it must follow (Gather, Analyze, Human Act, Verify) and the safety rules (what it must never do). This makes its behaviour predictable and safe, and it stops it from guessing or taking risky actions on a live server.

---

**2. Why is the human required to execute the recovery command?**

Recovery actions change the system, and a wrong action can make an incident worse or hide the real problem. The human operator is responsible for the server, can understand the wider situation and can decide whether the action is safe right now. Keeping the human in control means the AI can help diagnose, but a person always approves and runs anything that changes the system.

---

**3. Which rule prevents Claude from making an unsupported diagnosis?**

The rule "Do not claim a root cause unless the report contains supporting evidence." Together with "Use only the Bash report as the primary source of incident evidence", it forces Claude to base its diagnosis on real data from the report instead of guessing.

---

# Task 3 — Use Agentic AI to Plan Before Writing the Script

## Goal

Use Claude Code to inspect the environment and produce a read-only plan before creating any Bash code.

### Evidence

#### Screenshot 4 — Claude Code showing the five-check plan and read-only inspection results

![](screenshots/assign%206/4.png)

---

### Notes

Answer the following in your own words:

**1. Which part of this task represents the Gather phase?**

The Gather phase is when Claude reads `CLAUDE.md` and inspects the server with read-only commands such as `systemctl is-active nginx`, `ss -ltn`, `curl -I http://localhost`, `df -h /` and `free -m`. These commands only collect information about the current state and do not change anything.

---

**2. Did Claude follow the instruction not to create files? How did you verify this?**

Yes. Claude only ran read-only commands (such as `systemctl`, `ss`, `curl`, `df` and `free`) and ended with "As you asked, I didn't create or edit any files." I verified this myself by saving the list of files in the workspace (`find . -type f | sort`) before and after the Claude session and comparing them with `diff`. There was no difference, so no files were created, changed or removed. Write and Edit were also blocked for the session.

---

**3. Why is planning before coding useful in DevOps automation?**

Planning first makes sure we check the right things before writing any code. It lets us agree on what "healthy" and "failed" mean for each check, catch mistakes early, and avoid writing scripts that are wrong or unsafe. On real servers, a wrong automation can cause outages, so it is safer and faster to review a plan than to fix a broken script later.

---

# Task 4 — Build the Linux Triage Bash Script

## Goal

Create one Bash script that gathers consistent Linux and Nginx health evidence.

### Evidence

#### Screenshot 5 — Top section of `linux-triage.sh` showing variables, thresholds, and the checks array

![](screenshots/assign%206/5.png)

---

#### Screenshot 6 — Middle section showing check functions and conditionals

![](screenshots/assign%206/6.png)

---

#### Screenshot 7 — Bottom section showing the loop, summary function, and exit behavior

![](screenshots/assign%206/7.png)

---

#### Screenshot 8 — Output of `bash -n scripts/linux-triage.sh` (no syntax errors) and `ls -l scripts/linux-triage.sh` showing executable permission

![](screenshots/assign%206/8.png)

---

### Notes

Answer the following in your own words:

**1. What is stored in the checks array?**

The `checks` array stores the names of the five check functions: `check_service`, `check_port`, `check_http`, `check_disk` and `check_memory`.

---

**2. How does the `for` loop use that array?**

The loop `for check_function in "${checks[@]}"` goes through the array one item at a time. On each round, the function name is stored in `check_function`, and the line `"$check_function"` calls that function. This runs all five checks in order without calling each one by hand.

---

**3. Why are the health checks separated into functions?**

Each function does one job, so the script is easier to read, test and fix. If one check needs to change, I only edit that function. New checks are easy to add: write a new function and add its name to the array. It also keeps the output consistent, because every check reports through the same `mark_pass`, `mark_warning` and `mark_failure` functions.

---

**4. What is the purpose of `$(...)` in this script?**

`$(...)` is command substitution. It runs a command and stores its output so the script can use it. For example, `$(hostname)` puts the server name in the report, `$(date -u ...)` adds the timestamp, and `http_code=$(curl ...)` saves the HTTP status code so the script can compare it with `200`.

---

**5. Why does the script use different exit codes for HEALTHY, WARN, and FAIL?**

Exit codes let other programs and people know the result without reading the whole report. `0` means HEALTHY, `1` means WARN and `2` means FAIL. Other tools, like Claude Code, a CI/CD pipeline, cron or a monitoring system, can check the exit code and react automatically, for example by sending an alert only when the code is not 0.

---

# Task 5 — Run and Understand the Healthy-State Report

## Goal

Run the Bash script against the healthy server and verify that it creates a report.

### Evidence

#### Screenshot 9 — Output of `./scripts/linux-triage.sh` showing your Full Name and all five check results

![](screenshots/assign%206/9.png)

---

#### Screenshot 10 — Output showing the captured exit code and final summary

![](screenshots/assign%206/10.png)

---

### Notes

Answer the following in your own words:

**1. What is the overall status of your healthy baseline?**

The overall status of my baseline is HEALTHY. All five checks passed: Nginx is active, port 80 is listening, the local HTTP check returned 200, and root disk usage and available memory are both well within the thresholds. There are no WARN or FAIL results.

---

**2. Which exact Linux evidence proves the application is serving traffic?**

The line `[PASS] Local HTTP check returned status 200` in the report. It comes from `curl` making a real request to `http://localhost` and Nginx answering with HTTP 200. It is supported by `[PASS] Nginx service is active` and `[PASS] Port 80 is listening`.

---

**3. Did your script return exit code 0 or 1? Explain why.**

The script returned exit code 0, because there were no failed checks and no warnings. In the `print_summary` function, the exit code is 0 only when both `failure_count` and `warning_count` are 0, and the captured exit code matched the `Script Exit Code: 0` line in the report.

---

**4. What is the difference between a warning and a failure in this script?**

A warning means something needs attention but the service still works, for example disk usage between 80% and 89% or available memory below 100 MB. A failure means something is broken or critical, for example Nginx is not running, port 80 is not listening, the HTTP check does not return 200, or disk usage is 90% or more. Warnings give exit code 1 and failures give exit code 2.

---

# Task 6 — Create and Run the /linux-triage Skill

## Goal

Turn the Bash script into a reusable, manually invoked Agentic AI workflow.

### Evidence

#### Screenshot 11 — `SKILL.md` showing the frontmatter, allowed tool restrictions, and safety rules

![](screenshots/assign%206/11.png)

---

#### Screenshot 12 — `/linux-triage` output for the healthy server

![](screenshots/assign%206/12.png)

---

### Notes

Answer the following in your own words:

**1. Why does this skill have Bash, Read, and Grep, but not Write?**

The skill only needs to gather and read evidence. Bash runs the read-only triage script, Read opens the report, and Grep searches inside it. Without Write, Claude cannot create or change files, which follows the safety rule that triage must be read-only and makes sure the skill cannot accidentally edit configuration or delete evidence.

---

**2. Why is `disable-model-invocation: true` useful for this skill?**

It means Claude will never run the skill on its own. The skill only runs when I type `/linux-triage`. This keeps the human in control of when the server is inspected and avoids Claude starting triage automatically during unrelated conversations.

---

**3. What part is performed by Bash, and what part is performed by Claude?**

Bash does the Gather part: it runs the five checks on the server, collects the real evidence and writes it to `reports/linux-health-report.txt` with an exit code. Claude does the Analyze part: it reads the report, explains the overall status and any WARN or FAIL results, suggests the most likely cause, and recommends one recovery command and one verification command for the human to review.

---

**4. Why is this better than asking Claude "Is my server healthy?" without giving it evidence?**

If I just ask "Is my server healthy?", Claude has no real data, so it can only guess or give generic advice. With the skill, Claude analyses real, current evidence from my server collected the same way every time. The answer is based on facts, it can be checked against the report, and it is much less likely to be wrong.

---

# Task 7 — Simulate an Nginx Incident and Let the Skill Diagnose It

## Goal

Create a controlled service failure, gather evidence through Bash, and let Claude analyze the evidence without taking recovery action.

### Evidence

#### Screenshot 13 — Output showing Nginx is inactive and the HTTP request fails

![](screenshots/assign%206/13.png)

---

#### Screenshot 14 — `/linux-triage` output showing failed evidence, most likely cause, and a suggested recovery command

![](screenshots/assign%206/14.png)

---

#### Screenshot 15 — `incident-failure-report.txt` showing the failed checks and your Full Name

![](screenshots/assign%206/15.png)

---

### Notes

Answer the following in your own words:

**1. Which three checks failed?**

1. `[FAIL] Nginx service is not active`
2. `[FAIL] Port 80 is not listening`
3. `[FAIL] Local HTTP check returned status 000`

---

**2. What evidence supports the conclusion that Nginx is unavailable?**

`systemctl is-active nginx` returned `inactive`, port 80 no longer appeared in the `ss -ltn` output, and `curl -I http://localhost` could not connect, which the script reported as status `000` (no HTTP response at all). Disk and memory still passed, so the problem is only with the Nginx service, not with the server's resources.

---

**3. Did Claude execute the recovery command? Why is that important?**

No. Claude only recommended `sudo systemctl start nginx` and asked me to review and run it myself. This is important because the skill has no Write access, is told never to use sudo or start services, and the human must stay in control of any action that changes the server. It also proves the safety rules in `CLAUDE.md` and `SKILL.md` actually work.

---

**4. Which phase of the Agentic Loop is represented by the Bash report?**

The Bash report represents the Gather phase. The script collects the evidence from the server and saves it in the report.

---

**5. Which phase is represented by Claude's explanation?**

Claude's explanation represents the Analyze phase. It reads the evidence, explains what failed, finds the most likely cause and recommends a safe next step.

---

# Task 8 — Recover Manually, Verify Again, and Write the Incident Summary

## Goal

Recover the service as the human operator and prove that the system is healthy again.

### Evidence

#### Screenshot 16 — Output showing Nginx is active and `curl -I http://localhost` returns 200 OK

![](screenshots/assign%206/16.png)

---

#### Screenshot 17 — Second `/linux-triage` output showing successful recovery with no FAIL results

![](screenshots/assign%206/17.png)

---

#### Screenshot 18 — Output of `ls -lah reports` showing both `incident-failure-report.txt` and `recovery-report.txt`

![](screenshots/assign%206/18.png)

---

#### Screenshot 19 — `incident-summary.md` showing all required sections and your Full Name

![](screenshots/assign%206/19.png)

---

### Notes

Answer the following in your own words:

**1. What action did you execute manually?**

I reviewed Claude's recommendation and manually ran `sudo systemctl start nginx` in a regular terminal to start the Nginx service again.

---

**2. What evidence proves that the service recovered?**

`systemctl is-active nginx` returned `active`, and `curl -I http://localhost` returned `HTTP/1.1 200 OK`. The second `/linux-triage` run showed all five checks as PASS, Overall Status HEALTHY and exit code 0, with no FAIL results.

---

**3. Why is the second triage run necessary?**

Starting the service does not prove the problem is fixed. The second triage run is the Verify phase: it checks every part again in the same consistent way and proves that the service, the port and the application are all healthy. It also produces `recovery-report.txt`, which can be compared with the failure report as evidence that the recovery worked.

---

**4. What could go wrong if an AI agent automatically restarted every failed service?**

Restarting can hide the real cause, so the same problem keeps happening. If a service fails because of a bad configuration, a full disk or an attack, restarting it again and again can cause restart loops, data loss or a bigger outage. Some services are stopped on purpose for maintenance, and an agent could restart them at the wrong time. Without a human in the loop, nobody reviews the decision or takes responsibility for it.

---

**5. In one sentence, explain the difference between using AI as a chatbot and using AI in this agentic workflow.**

A chatbot only answers questions from general knowledge, while in this agentic workflow the AI follows project rules, gathers real evidence from the server with tools, analyses it and recommends a safe action that a human approves and verifies.

---

# Incident Summary

Fill in all seven sections below in your own words.

**Full Name:** Hasti Kabiri

**Date:** 05/11/2026

---

**1. Reported Symptom**

The React application stopped responding on `http://localhost`. A request to the site could not connect, and the web page was no longer available.

---

**2. Evidence Collected**

The Bash triage report (`reports/incident-failure-report.txt`) showed three failed checks:

- `[FAIL] Nginx service is not active`: systemd reported Nginx as `inactive`.
- `[FAIL] Port 80 is not listening`: no process was listening on port 80.
- `[FAIL] Local HTTP check returned status 000`: `curl` could not connect, so there was no HTTP response.

The root disk usage and available memory checks still passed, and the overall status was FAIL with exit code 2.

---

**3. Most Likely Cause**

The Nginx service was stopped. Because Nginx was not running, nothing was listening on port 80, so the application could not respond to HTTP requests. Disk and memory were healthy, so resource problems are not supported by the evidence. In this lab, the service was stopped on purpose with `sudo systemctl stop nginx` to simulate the incident.

---

**4. Human-Approved Recovery Action**

`sudo systemctl start nginx`. Claude recommended this command but did not run it. I reviewed it and ran it myself in a regular terminal.

---

**5. Verification**

- `systemctl is-active nginx` returned `active`.
- `curl -I http://localhost` returned `HTTP/1.1 200 OK`.
- The second `/linux-triage` run showed all five checks as PASS, Overall Status HEALTHY and exit code 0, saved in `reports/recovery-report.txt`.

---

**6. Safety Decision**

The AI skill was allowed to gather and analyse evidence because those actions are read-only and cannot harm the server. It was not allowed to restart the service because changing a production system is a risky decision that needs human judgement and responsibility. The skill had no Write tool, was told never to use sudo, and the safety rules said to recommend a recovery command but never execute it.

---

**7. Agentic Loop Mapping**

- **Gather:** the `linux-triage.sh` Bash script collected evidence from the server and saved it to the report.
- **Analyze:** Claude read the report through the `/linux-triage` skill, identified the three failed checks and explained that Nginx was stopped.
- **Human Act:** I reviewed the recommended command and ran `sudo systemctl start nginx` myself.
- **Verify:** I checked `systemctl` and `curl`, then ran `/linux-triage` again, which showed no FAIL results.

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://lnkd.in/p/grw2M3Zj

---

#### Screenshot — Published LinkedIn post

![](screenshots/assign%206/linkedin.png)

---

# GitHub Repository URL

Paste the URL of your GitHub folder or repository containing the assignment files here:

https://github.com/HastiKabiri/devops-micro-internship-pravinmishra
---

# Submission Instructions

- Add all required screenshots in your submission
- Full Name must be visible in required screenshots and the Bash report
- All written answers must be in your own words
- Do not expose sensitive information (keys, passwords, AWS account IDs, tokens)
- GitHub URL must be included in this document

---

# Completion Checklist

- [✅] Task 1: Healthy baseline confirmed, workspace created (Screenshots 1–2, Notes answered)
- [✅] Task 2: CLAUDE.md created with all four sections (Screenshot 3, Notes answered)
- [✅] Task 3: Five-check plan produced by Claude using read-only tools (Screenshot 4, Notes answered)
- [✅] Task 4: `linux-triage.sh` created, syntax validated, executable permission set (Screenshots 5–8, Notes answered)
- [✅] Task 5: Healthy-state report generated with no FAIL result (Screenshots 9–10, Notes answered)
- [✅] Task 6: `/linux-triage` skill created and run successfully on healthy server (Screenshots 11–12, Notes answered)
- [✅] Task 7: Nginx incident simulated, failed evidence captured, Claude did not execute recovery (Screenshots 13–15, Notes answered)
- [✅] Task 8: Nginx recovered manually, recovery verified, reports saved, incident summary complete (Screenshots 16–19, Notes answered)
- [✅] Incident summary contains all seven required sections
- [✅] LinkedIn post published and URL submitted
- [✅] Full Name visible in all required screenshots and the Bash report
- [✅] Skill does not have Write permission
- [✅] Skill did not execute any recovery commands
- [✅] No sensitive data exposed

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