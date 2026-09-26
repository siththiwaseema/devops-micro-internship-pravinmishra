# Assignment 8 — Week 2 Reflection Blog

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

# Purpose

In this assignment, you will reflect on your Week 2 learning journey and write a short blog capturing your experience working with Agentic AI tools such as Claude Code, Skills, Subagents, MCP, Hooks, Permissions, and Memory.

You will also publish a LinkedIn post summarizing your learning and share both links for evaluation.

---

# Task 1 — Write Your Reflection Blog

## Goal

Write a reflection blog covering your Week 2 learning experience.

### Blog Requirements

Your blog must include:

* Title: **Reflection – Week 2**
* Minimum 300 words
* At least 2–3 topics from Week 2 (Claude Code, Skills, Subagents, MCP, Hooks, Permissions, Memory)
* Honest personal reflection (learning, challenges, mindset)
* One habit/system you plan to implement
* Your full name clearly visible

### Allowed Platforms

You can publish your blog on:

* Hashnode
* Medium
* Dev.to
* LinkedIn Article
* GitHub Markdown file
* Substack

---

### Evidence

#### Screenshot 1 — Blog published and visible

![Week 02 – agentic-ai](screenshots/Assignment%2008/1.png)
---

### Submission Field

Blog Link: 

https://medium.com/@wassimasiththy/why-hooks-and-permissions-are-important-for-safety-771c50230df3?sharedUserId=wassimasiththy
---

# Task 2 — Create LinkedIn Post

## Goal

Share your Week 2 learning publicly on LinkedIn.


---

### Evidence

#### Screenshot 2 — LinkedIn post published

![Week 02 – agentic-ai](screenshots/Assignment%2008/2.png)
---

### Submission Field

LinkedIn Post Content :

Why Hooks and Permissions Are Important for Safety

I learned how Claude works behind the scenes. Claude does not just know what I want. It needs some files to do the job the way I want. Each file has its own purpose.

CLAUDE.md is the rulebook of the project. I write the main rules here, like which tools we use and what Claude should never do. Claude reads this file every time it starts, so it always follows the same rules.

MEMORY.md is where Claude saves important facts. When I tell Claude something once, it keeps it here. In a new session, Claude still remembers it, so I don’t have to repeat myself.

Skills are saved instructions that turn into slash commands. Instead of writing a long prompt again and again, I write it once in a skill file and run it with one short command like /infra-audit.

Hooks are automatic actions. They run at a fixed moment, for example before Claude runs a command or after it changes a file. I don’t need to remember to do them. They happen on their own.

All these files help Claude do the work. But while learning them, one question stayed in my mind: what stops Claude from doing something wrong? That is where hooks and permissions come in.

Claude is not just talking anymore
In a normal chat, Claude only gives answers. If the answer is wrong, nothing breaks. I just ignore it.

Claude Code is different. It can run real commands on my computer. It can create files, delete files, and even change things in my AWS account. That is what makes it powerful. But it also means one wrong command can cause real damage.

For example, a command like rm -rf can delete a whole folder. A command like terraform destroy can remove my live website from AWS. These mistakes are not easy to undo.

So the question is not only “Can Claude do this job?” The question is also “What should Claude be allowed to do?”

Permissions: deciding what Claude can do
Permissions are like giving keys to someone. You don’t give every key to everyone. You give only the keys they need.

In Claude Code, I can decide which actions are allowed, which ones are blocked, and which ones need my approval first.

A simple example from this week: an agent that only checks a Terraform plan does not need Write access. Its job is to read and report. If it cannot write, it cannot change or delete anything, even by mistake.

Think of a bank. The person who checks your balance does not need the key to the safe. Less access means less risk.

Hooks: an automatic safety guard
Permissions set the rules. Hooks help check them at the right moment.

A hook can run before Claude does something. It looks at the command and stops it if it looks dangerous. For example, a hook can block any command that tries to read or change the .env file, where secrets are kept.

A hook can also run after Claude does something. For example, after Claude edits a Terraform file, a hook can automatically format it or check it for errors.

The best part is that hooks don’t depend on anyone remembering. Even if I forget, or Claude forgets, the hook still runs.

It is like a security guard at a building gate. The guard checks everyone, every time, without being told.

Why this matters in DevOps
In DevOps, we work with real systems: servers, websites, databases, cloud accounts. Mistakes here can mean downtime, lost data, or a big AWS bill.

Using AI in this work saves a lot of time. But speed without control is dangerous. Hooks and permissions give that control. They let Claude do the repeated work, while I still decide the limits.

What I learned
CLAUDE.md tells Claude the rules. MEMORY.md helps it remember. Skills save time. But hooks and permissions are what make it safe to use Claude on real projects.

Giving AI power is easy. Giving it the right amount of power, with clear limits, is what real engineering looks like.

P.S. This post is part of the DevOps Micro Internship (DMI) — Self-Paced Engineer Track — by Pravin Mishra. My graded progress is public: https://dmi.pravinmishra.com/s/siththiwaseema.html · Start your DevOps journey: https://dmi.pravinmishra.com/?utm_source=student&utm_medium=ps-blog&utm_campaign=self-paced

#DMIByPravinMishra #AgenticAI #ClaudeCode #DevOps
---

### LinkedIn Post Link:

link: https://www.linkedin.com/posts/siththi-waseema-62a0b0187_1a1a2e-16213e-dmibypravinmishra-ugcPost-7509176782521106432-2Cmj/?utm_source=share&utm_medium=member_desktop&rcm=ACoAACv5boIBHr4DjAudB4kGRvqYrehYIp1o_Io
---

# Submission Instructions

* Blog must be publicly accessible
* LinkedIn post must be visible (public or unlisted where applicable)
* All required fields must be filled
* Screenshot proofs must be added to GitHub repository
* Do not include sensitive information in blog or post

---

# Completion Checklist

* [ ] Blog written with required structure
* [ ] Blog includes at least 2–3 Week 2 topics
* [ ] Blog is publicly accessible
* [ ] LinkedIn post created
* [ ] Required P.S. line included
* [ ] LinkedIn post content copied in submission field
* [ ] Blog link added
* [ ] LinkedIn post link added
* [ ] Screenshots added to GitHub repo

---

# About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory), focused on real-world execution, systems thinking, and agentic AI workflows.

It helps learners build strong DevOps foundations through hands-on experience.

---

# Resources

* 🌐 DMI Official Website: [https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme](https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme)
* 🎓 University: [https://university.pravinmishra.com?utm_source=github&utm_medium=readme](https://university.pravinmishra.com?utm_source=github&utm_medium=readme)
* 💬 Discord Community: [https://discord.pravinmishra.com?utm_source=github&utm_medium=readme](https://discord.pravinmishra.com?utm_source=github&utm_medium=readme)
* 📝 Blog: [https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme](https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme)
* ▶️ YouTube Playlist: [https://www.youtube.com/playlist?list=PLFeSNDtI4Cho](https://www.youtube.com/playlist?list=PLFeSNDtI4Cho)
* 🔗 Pravin Mishra (LinkedIn): [https://www.linkedin.com/in/pravin-mishra-aws-trainer/](https://www.linkedin.com/in/pravin-mishra-aws-trainer/)
* 🏢 CloudAdvisory (LinkedIn): [https://www.linkedin.com/company/thecloudadvisory/](https://www.linkedin.com/company/thecloudadvisory/)
---

*This submission is part of DevOps Micro Internship (DMI) — Agentic AI Track.*
