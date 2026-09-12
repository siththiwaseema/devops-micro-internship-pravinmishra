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

![Week 02 – agentic-ai](screenshots\Assignment 05\1.png)

---

# Task 2 — Compare the Agent Configurations

## Goal

Analyze the configuration differences between the three agents and demonstrate understanding of model and tool selection.

### Written Answers

#### 1. Why does the cost optimizer use Haiku instead of Sonnet?

The cost optimizer uses Haiku instead of Sonnet because cost analysis is straightforward calculation-based work that doesn't require deep reasoning. 
Haiku is 10x cheaper and 2x faster, making it perfect for quick cost checks that deliver immediate feedback. Haiku can easily compare instance sizes, calculate monthly costs, and recommend savings without the overhead and expense of Sonnet's deeper reasoning capabilities.

---

#### 2. Why does the security auditor NOT have Write in its tools list?

The security auditor does NOT have Write in its tools list because its job is to REVIEW and ANALYZE infrastructure, not modify it. By restricting access to only Read and Grep, we prevent the agent from accidentally (or maliciously) modifying or deleting Terraform files. This follows the Principle of Least Privilege — the agent can never corrupt  infrastructure, even if it has a bug. The read-only restriction builds trust and ensures the auditor remains impartial.

---

#### 3. Why does the tf-writer use `inherit` instead of a specific model?

The tf-writer uses "inherit" instead of a specific model because code 
generation requirements depend on the complexity of the task and current session capabilities. Using "inherit" allows the agent to adapt to whatever model is running in your Claude session — if you need complex infrastructure, can run Sonnet; for simpler code, a faster model works fine. This flexibility means one agent works optimally across different scenarios without being locked to a single model choice.
---

### Evidence

#### Screenshot 2 — `security-auditor.md` frontmatter showing model and tools configuration

![Week 02 – agentic-ai](screenshots\Assignment 05\2.png)

---

#### Screenshot 3 — `cost-optimizer.md` frontmatter showing the model and tools configuration

![Week 02 – agentic-ai](screenshots\Assignment 05\3.png)

---

# Task 3 — Run the Security Auditor

## Goal

Trigger the security auditor agent and analyze the generated security report for your Terraform infrastructure.

### Evidence

#### Screenshot 4 — The delegation message showing Claude launched the security-auditor

![Week 02 – agentic-ai](screenshots\Assignment 05\4.png)

---

#### Screenshot 5 — Security audit report output

![Week 02 – agentic-ai](screenshots\Assignment 05\5.png)

---

# Task 4 — Run the Cost Optimizer

## Goal

Trigger the cost optimizer agent and review the generated cost optimization report.

### Evidence

#### Screenshot 6 — The full cost optimization report

![Week 02 – agentic-ai](screenshots\Assignment 05\6.png)

---

# Submission Instructions

- Ensure all agent files are committed in `.claude/agents/`
- Complete all written answers in your GitHub Repo
- Push final changes to your forked GitHub repository

---

## GitHub Repository URL

https://github.com/siththiwaseema/devops-micro-internship-pravinmishra/tree/main



---

# Completion Checklist

- [ ] `.claude/agents/` folder contains all 3 agent files
- [ ] Screenshot 2 shows correct `security-auditor.md` configuration
- [ ] Screenshot 3 shows correct `cost-optimizer.md` configuration
- [ ] All 3 written answers completed 
- [ ] Security auditor executed successfully
- [ ] Cost optimizer executed successfully
- [ ] Security report is visible with findings
- [ ] Cost report is visible with recommendations
- [ ] All required screenshots added
- [ ] GitHub repo updated with agents

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