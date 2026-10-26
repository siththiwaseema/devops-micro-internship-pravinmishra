# Assignment 6 — Capstone Assignment — Deploy Book Review App (Three-Tier Architecture) on AWS

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

This is the most important assignment of the course. You will deploy the Book Review App in a fully production-style three-tier architecture on AWS: a Next.js Web Tier behind Nginx and a public ALB, a private Node.js/Express App Tier behind an internal ALB, and a private Multi-AZ MySQL RDS database with a read replica. You are expected to design, deploy, isolate, debug, and document the result independently.

---

# Task 1 — Architecture Diagram

## Goal

Create an architecture diagram showing the custom VPC (10.0.0.0/16), the six subnets across two Availability Zones (two public Web Tier, two private App Tier, two private Database Tier), the public ALB, Web Tier EC2/Nginx, internal ALB, private App Tier EC2, private Multi-AZ RDS with its read replica, and the permitted traffic flow.

### Evidence

#### Diagram image or link

![Week 06–aws-cloud](screenshots/Assignment%206/1.png)

---

# Task 2 — AWS Region & Services Used

## Goal

Record the AWS Region used and list every AWS service used across networking, compute, load balancing, security, and the database.

### Notes

**Region:**

AP SOUTH -1

---

**Services:**

EC2

---

# Task 3 — Public Entry Point

## Goal

Confirm the Book Review App loads through the public ALB DNS name.

### Evidence

#### Public ALB DNS

![Week 06–aws-cloud](screenshots/Assignment%206/2.png)

---

# Task 4 — Evidence Screenshots

## Goal

Capture visual proof of every tier and load balancer.

### Evidence

#### Web EC2

![Week 06–aws-cloud](screenshots/Assignment%206/3.png)

---

#### App EC2

![Week 06–aws-cloud](screenshots/Assignment%206/4.png)

---

#### Public ALB

![Week 06–aws-cloud](screenshots/Assignment%206/5.png)

---

#### Internal ALB

![Week 06–aws-cloud](screenshots/Assignment%206/6.png)

---

#### RDS + Replica

![Week 06–aws-cloud](screenshots/Assignment%206/8.png)

---

#### App UI proof

![Week 06–aws-cloud](screenshots/Assignment%206/9.png)

---

# Task 5 — Summary

## Goal

Summarize what worked in the final deployment, the issues encountered and how each was fixed, and the tools or sources used to research and debug.

### Notes

Public Connectivity: Established a Public Application Load Balancer (ALB) and associated it with a Public Target Group.
Environment Setup: Successfully installed necessary software (Node.js, Nginx, PM2) across the instances.
Internal Routing: Created and configured an Internal ALB to bridge communication between the Web and App tiers.

Issues + Fixes:

Issue: Missing Public Target Group.
Fix: Created capstone-public-tg and registered the Web EC2 instance.
Issue: Internal Target Group showing "Unused" and "None Associated."

Fix: Identified the missing link between the Internal ALB and the Target Group; added a Listener on Port 80 to the Internal ALB.
Issue: Security Group Blockages.
Fix: Updated the App Security Group to allow Port 3001 from the Internal ALB and the Web Security Group to allow Port 80 from the Public ALB.
Issue: Target Group Mismatch (Web and App instances in the same group).
Fix: Cleaned up the targets so that only Web EC2 is in the Public group and only App EC2 is in the Internal group.

Tools/Sources Used:
AWS Management Console: (EC2, VPC, RDS, Target Groups, Load Balancers).
SSH/Terminal: For server-side configuration.
Nginx: Used as a Reverse Proxy on the Web Server.
PM2: Process manager used to keep the Frontend and Backend applications running.
AWS Documentation: For verifying Load Balancer listener and target group health check configurations.

Final Status Note:
Despite troubleshooting the entire 3-tier flow and resolving multiple configuration issues within the Target Groups and Security Groups, the application failed to load in the browser (likely due to a persistent 504 Gateway Timeout or a specific Nginx routing conflict).


---

# LinkedIn Post (Required)

## Goal

Publish a LinkedIn post sharing the capstone deployment, including the public ALB DNS (or a redacted screenshot), three to five lines on what you built and why it is production-style, and one proof screenshot.

## Evidence

#### LinkedIn Post URL

https://www.linkedin.com/feed/update/urn:li:activity:7431972364327825409
---

#### Screenshot of LinkedIn post

![Week 06–aws-cloud](screenshots/Assignment%206/ss.png)

---

# Submission Instructions

- Add all required screenshots and links in your submission
- Do not expose passwords, RDS credentials, connection strings, private keys, or account IDs

---

# Completion Checklist

- [ ] Task 1: Architecture diagram completed
- [ ] Task 2: AWS Region and services documented
- [ ] Task 3: Public ALB DNS confirmed working
- [ ] Task 4: All six evidence screenshots captured (Web Tier, App Tier, both ALBs, RDS + replica, app UI)
- [ ] Task 5: Deployment summary completed (what worked, issues/fixes, tools/sources)
- [ ] LinkedIn post published and URL submitted
- [ ] App Tier and Database Tier confirmed not publicly accessible
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