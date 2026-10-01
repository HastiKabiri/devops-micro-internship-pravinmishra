# Assignment 4 — Building Your AI Team

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will build and configure a set of specialized AI subagents inside your project. You will learn how different models and tool permissions define agent behavior, and you will trigger two real agent delegations to analyze security and cost aspects of your Terraform infrastructure.

---

# Task 1 — Create the Agents Folder and Add Files

## Goal

Create the `.claude/agents/` directory and add all required agent files.

### Evidence

#### Screenshot 1 — VS Code sidebar showing `.claude/agents/` with all 3 files

![](screenshots/Assing%204%20pics/scre%201.png)
---

# Task 2 — Compare the Agent Configurations

## Goal

Analyze the configuration differences between the three agents and demonstrate understanding of model and tool selection.

### Written Answers

#### 1. Why does the cost optimizer use Haiku instead of Sonnet?

Haiku is utilized by cost optimizers in place of Sonnet because it helps to save costs, gives response time that is way quicker and possesses adequate power for simple tasks, thereby making Sonnet unnecessary.

---

#### 2. Why does the security auditor NOT have Write in its tools list?

The tools used by the security auditor do not allow writing in order to ensure security and to avoid any changes that may be deliberate or unintended. The auditor works only in read-only mode, so that it can analyze any code, configure or infrastructure without the risk of making any changes to the code, corrupt the data, or cause instability.

---

#### 3. Why does the tf-writer use `inherit` instead of a specific model?

In the `tf-writer`, there is an architectural preference for using the word `inherit` over just tying it to one particular model. The following are the main reasons:

- The Aspect of Flexibility:** The usage of `inherit` enables the writing tool to automatically select the model used in either the parent session, the CLI or a more general configuration.
- The Aspect of Configuration:** When it comes to selecting a particular model (like Sonnet or Haiku), the change can be made only at the parent level or that of the orchestrator.
- The Aspect of Consistency in Workflow:** The utilization of `inherit` ensures that the creation of text is carried out under the same model levels, costs, and context as the overall workflow.
---

### Evidence

#### Screenshot 2 — `security-auditor.md` frontmatter showing model and tools configuration

![](screenshots/Assing%204%20pics/scre%202.png)

---

#### Screenshot 3 — `cost-optimizer.md` frontmatter showing the model and tools configuration

![](screenshots/Assing%204%20pics/scre%203.png)

---

# Task 3 — Run the Security Auditor

## Goal

Trigger the security auditor agent and analyze the generated security report for your Terraform infrastructure.

### Evidence

#### Screenshot 4 — The delegation message showing Claude launched the security-auditor

![](screenshots/Assing%204%20pics/scre%204.png)

---

#### Screenshot 5 — Security audit report output

![](screenshots/Assing%204%20pics/scre%205.png)

---

# Task 4 — Run the Cost Optimizer

## Goal

Trigger the cost optimizer agent and review the generated cost optimization report.

### Evidence

#### Screenshot 6 — The full cost optimization report

![](screenshots/Assing%204%20pics/scre%206.png)


---

# Submission Instructions

- Ensure all agent files are committed in `.claude/agents/`
- Complete all written answers in your GitHub Repo
- Push final changes to your forked GitHub repository

---

## GitHub Repository URL

Paste your forked repository URL here:

https://github.com/HastiKabiri/Ultimate-Agentic-DevOps-with-Claude-Code.git 

---

# Completion Checklist

- [✅] `.claude/agents/` folder contains all 3 agent files
- [✅] Screenshot 2 shows correct `security-auditor.md` configuration
- [✅] Screenshot 3 shows correct `cost-optimizer.md` configuration
- [✅] All 3 written answers completed 
- [✅] Security auditor executed successfully
- [✅] Cost optimizer executed successfully
- [✅] Security report is visible with findings
- [✅] Cost report is visible with recommendations
- [✅] All required screenshots added
- [✅] GitHub repo updated with agents

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