# Assignment 3 — Production Maintenance Drill (OPS Checklist)

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will treat your already deployed React application (on Ubuntu VM with Nginx) as a live production system. You will perform structured operational checks covering network validation, service health, log analysis, resource monitoring, configuration verification, and incident simulation with recovery — mirroring real on-call DevOps responsibilities.

---

# Task 1 — Server Access & Networking Validation

## Goal

Verify that the deployed React application is reachable from the browser and confirm basic network connectivity of the Ubuntu VM.

### Evidence

#### Screenshot 1 — Browser showing the React app with your Full Name visible on the UI

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/1.png)

---

#### Screenshot 2 — Output of `ip a`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/2.png)

---

#### Screenshot 3 — Output of `sudo ss -tulpen`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/3.png)

---

#### Screenshot 4 — Output of `sudo ufw status`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/4.png)

---

### Notes

Answer the following in your own words:

**1. What proves Nginx is listening on 0.0.0.0:80?**

In the `ss -tulpen` output, there is a TCP line in `LISTEN` state with the local address
`0.0.0.0:80`, and the process column shows `nginx`. The address `0.0.0.0` means Nginx accepts
connections on every network interface, not only from inside the server. The fact that the
React app loads from my browser through the public IP also confirms this.

---

**2. What proves SSH is active on port 22?**

`ss -tulpen` shows a TCP line in `LISTEN` state on `0.0.0.0:22` (and `[::]:22` for IPv6),
owned by `sshd` (or `systemd` on Ubuntu 24.04, which starts SSH through socket activation).
My SSH session to the server is also working, which proves port 22 is open and reachable.

---

**3. Did you find any unexpected open ports? Explain briefly.**

No. Only ports 22 (SSH) and 80 (Nginx) listen on all interfaces. The other entries are
normal Ubuntu system services that only listen locally: `systemd-resolved` on
127.0.0.53:53 for DNS, and the DHCP client on UDP port 68. These can't be reached from the
internet.
UFW is inactive, which is the default on EC2. In this setup, the AWS security group works as
the firewall, and it only allows ports 22 and 80 inbound.

---

# Task 2 — Service Health & Systemd Validation (Nginx)

## Goal

Verify that Nginx is properly installed, running, enabled at boot, and safely configured.

### Evidence

#### Screenshot 1 — Output of `systemctl status nginx --no-pager`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/2.1.png)

---

#### Screenshot 2 — Output of `sudo nginx -t`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/2.2.png)

---

#### Screenshot 3 — Output of `sudo ss -lptn '( sport = :80 )'`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/2.2.png)

---

### Notes

Answer the following in your own words:

**1. What happens if Nginx fails to restart in production?**

A restart stops Nginx first and then starts it again. If the start fails, for example
because of a config error or because another program is using port 80, nothing is listening
on port 80. The website goes down, and users see "connection refused" or timeout errors
until someone fixes it. Any other sites or APIs served by the same Nginx also go down.
To avoid this, I would always run `sudo nginx -t` before restarting. For config-only changes,
I would use `sudo systemctl reload nginx` instead. A reload keeps the old workers running if
the new configuration is invalid, so the site stays up.

---

**2. What's your basic rollback plan?**

1. Before changing anything, back up the working config:
   `sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.bak`
   (and back up `/var/www/html` before deploying a new build).
2. If Nginx fails after a change, check why with `sudo nginx -t` and
   `sudo journalctl -u nginx --no-pager | tail -20`.
3. Restore the backup:
   `sudo cp /etc/nginx/sites-available/default.bak /etc/nginx/sites-available/default`
4. Run `sudo nginx -t` again, then `sudo systemctl restart nginx`.
5. Confirm the fix with `systemctl status nginx` and by opening the site in the browser.

---

# Task 3 — Logs & Request Trace

## Goal

Verify real traffic flow and analyze logs to understand system behavior and errors.

### Evidence

#### Screenshot 1 — Output of `sudo tail -n 30 /var/log/nginx/access.log`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/3.1.png)

---

#### Screenshot 2 — Output of `sudo tail -n 30 /var/log/nginx/error.log`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/3.2.png)

---

#### Screenshot 3 — Output of `sudo journalctl -u nginx --no-pager -n 50`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/3.3.png)

---

### Notes

Answer the following in your own words:

**1. Were there any errors in the logs?**

- If yes, mention 1–2 example error lines from the logs and explain what each one means in simple terms.
- If no, explain what it means if the error log is empty or shows no recent errors during your check.

Write your answer here.

No. `/var/log/nginx/error.log` had no errors during my check, and `journalctl -u nginx` only
showed normal start, stop and restart events with no failures.
---

**2. If there were no errors, what does that indicate about the system?**

An empty error log means Nginx did not run into any problems during the time I checked.
Every request was handled without failures, the configuration loaded correctly, and the
server could read the files in `/var/www/html`.
This does not mean the system is permanently perfect. It only covers the period I looked at.
Problems could still happen later, for example under heavy traffic, after a config change,
or if the disk fills up. That's why logs should be checked regularly or monitored with alerts,
not just checked once.

---

**3. Based on the access logs, were your curl requests visible in the log entries? What does that prove about traffic flow?**

Yes. The access log showed my requests coming from `13.201.189.3` with the `curl/...`
user agent: GET requests with status **200** from `curl -s`, and a HEAD request with status
**200** from `curl -I`. The timestamps matched the time I ran the commands.
This proves the full traffic path works. The request went out to the server's public IP,
passed through the AWS security group on port 80, reached Nginx, and Nginx served the React
app successfully. Each request was then recorded in the log.

---

# Task 4 — System Resource Health Check (Capacity Red Flags)

## Goal

Assess server capacity and detect potential performance or failure risks.

### Evidence

#### Screenshot 1 — Output of `uptime`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/4.1.png)

---

#### Screenshot 2 — Output of `free -h`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/4.2.png)

---

#### Screenshot 3 — Output of `df -h`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/4.3.png)

---

#### Screenshot 4 — Output of `sudo du -sh /var/* | sort -h`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/4.4.png)

---

### Notes

Answer the following in your own words:

**1. Which resource looks most critical right now? (CPU/load, memory, or disk) Explain why.**

Memory- CPU load is very low (<0.0x>, compared with <1> core), and the disk is only <XX%>
used. However, the server only has about 1 GB of RAM, and only <XXXMi> is available.
Building the React app (`npm install` / `npm run build`) uses most of that memory.
<If you added swap: "I had to add a swap file to finish the build.">
Serving the finished app with Nginx uses very little memory, but running another build or
adding more services could run the server out of memory.
When Linux runs out of memory, the "OOM killer" force-stops processes to free up RAM. It
could kill Nginx or the build process, which would take the site down with no warning.
Low memory also forces the system to use swap, which is much slower and makes the whole
server sluggish.

---

**2. What happens if disk becomes 100% full in a production server?**

When the disk is full, nothing new can be saved, and many things break at once:
- Logs stop writing - Nginx can't record requests or errors, so problems become invisible
  just when you need to investigate them.
- Applications fail - Anything that needs to save files, such as uploads, database
  writes or temporary files, starts returning errors.
- Deployments and updates fail - `apt`, `npm install` and copying a new build all need
  free disk space.
- The server can become unstable or unresponsive - System services may crash, SSH
  logins can fail, and a reboot might not complete cleanly.

To prevent this, I would monitor disk usage and set an alert (for example, at 80%), rotate and
clean old logs (`logrotate`, `journalctl --vacuum-size`), clear the package cache
(`sudo apt clean`), and increase the EBS volume size if needed.
---

# Task 5 — Configuration & Deployment Verification

## Goal

Ensure the correct React build is deployed and Nginx is serving it properly.

### Evidence

#### Screenshot 1 — Output of `ls -lah /var/www/html | head -n 20`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/5.1.png)

---

#### Screenshot 2 — Output of `grep -R "Deployed by" -n /var/www/html 2>/dev/null | head`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/5.2.png)

---

#### Screenshot 3 — Output of `grep -n "try_files" /etc/nginx/sites-available/default`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/5.3.png)

---

### Notes

Answer the following in your own words:

**1. How do you confirm that the correct version of the application is deployed?**

I verified the deployment in five steps:

1. **Checked the web root files.** `ls -lah /var/www/html` showed that the folder contains the
   React production build. The files are owned by `www-data`, the user Nginx runs as, and have
   755 permissions, so Nginx can read and serve them. The old default Nginx page
   (`index.nginx-debian.html`) is gone.

2. **Confirmed the build files are present.** I could see `index.html`, which is the entry point
   of the app, and the `static/` folder, which contains the bundled JavaScript and CSS. This
   proves the output of `npm run build` was copied, not the raw source code.

3. **Verified my custom change was deployed.** `grep -R "Deployed by"` found the text inside
   the built JavaScript file (`static/js/main.*.js`), together with my name. My change to
   `App.js` went through the build and ended up in the files Nginx is serving, so this is my
   personalized version and not the original template.

4. **Confirmed Nginx serves the correct web root.** The config uses `root /var/www/html;`,
   and `grep try_files` showed the rule `try_files $uri /index.html;`. Nginx serves files from
   the folder I deployed to. For any route that isn't a real file, it returns `index.html` so
   React can handle the routing. Refreshing the page on a React route won't cause a 404 error.

5. **Tested the app in the browser.** I opened `http://13.201.189.3`, and the app loaded
   correctly, showing "Deployed by Siththi Waseema and the deployed date 26/9/2026. This confirms the whole chain works: my source change, the build, the web root, the Nginx config and the public access.
---

# Task 6 — Nginx Configuration Failure Simulation

## Goal

Simulate a real-world Nginx misconfiguration and recover the service safely.

### Evidence

#### Screenshot 1 — Output of `sudo nginx -t` showing the syntax error (broken config)

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/6.1.png)

---

#### Screenshot 2 — Output of `sudo nginx -t` showing syntax ok (fixed config)

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/6.2.png)

---

#### Screenshot 3 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/6.3.png)

---

### Notes

Answer the following in your own words:

**1. What caused the configuration failure?**

A missing semicolon. In Nginx, every directive must end with `;`. Without it, Nginx didn't
know where the `try_files` line ended, so it kept reading onto the next line. When it reached
the closing `}`, it didn't expect it there, so it reported `unexpected "}"` and the syntax
test failed. The error showed the line number just after the actual mistake. Checking the
line above the reported line is a useful debugging habit.

---

**2. How did you fix the issue?**

I opened the config file in nano, went to the `try_files` line, and added the missing `;`
back. I ran `sudo nginx -t` to confirm the syntax was valid before restarting Nginx. After the
restart, I checked with `curl -I` that the site returned 200 OK. I did not restart Nginx
while the config was broken, so the website stayed online during the whole exercise, because
Nginx kept running with the last valid config.

---

**3. How can you avoid this kind of issue in real production systems?**

- **Always run `sudo nginx -t` before restarting or reloading.** It catches syntax errors
  before they cause downtime.
- **Use `sudo systemctl reload nginx` instead of restart for config changes.** If the new
  config is invalid, reload keeps the old one running instead of stopping the site.
- **Back up the config before editing** (`cp default default.bak`), so rolling back takes
  seconds.
- **Keep configs in Git.** Every change is tracked and reviewed, and can be reverted.
- **Test changes on a staging server first**, and automate deployments with a CI/CD pipeline
  that runs `nginx -t` automatically.
- **Monitor the site**, for example with a health check that alerts if it stops returning
  200 OK.

---

# Task 7 — Web Application Failure Simulation

## Goal

Simulate missing deployment content and recover the application safely.

### Evidence

#### Screenshot 1 — Output of `curl -I http://<public-ip>` showing failure (non-200 response)

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/7.1.png)

---

#### Screenshot 2 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/7.2.png)

---

### Notes

Answer the following in your own words:

**1. What caused the application to break in this scenario?**

The web content directory was removed. I moved `/var/www/html`, which held the React build
files, to a backup location and replaced it with an empty folder. Nginx was still running
and still pointing to `/var/www/html`, but there was nothing inside to serve: no
`index.html` and no `static/` files.
Because of the SPA rule `try_files $uri /index.html;`, Nginx tried to fall back to
`/index.html`. That file was also missing, so Nginx kept redirecting to itself until it
stopped and returned **500 Internal Server Error**. The Nginx service was healthy the whole
time. Only the application content was missing.
---

**2. How did you fix the issue and restore the application?**

1. I checked that the backup folder existed with `ls /var/www/`.
2. I deleted the empty folder: `sudo rm -rf /var/www/html`.
3. I restored the original files from the backup:
   `sudo mv /var/www/html_backup /var/www/html`.
4. I restarted Nginx: `sudo systemctl restart nginx`.
5. I checked the recovery with `curl -I http://13.201.189.3`, which returned
   **HTTP/1.1 200 OK**, and confirmed the app loaded in the browser with my name.
Recovery was quick because I had a backup of the working version.

---

**3. What steps would you take to prevent this kind of issue in real production systems?**

- **Always keep a backup of the current release** before deploying or changing anything, so
  rollback takes seconds.
- **Use versioned release folders.** Deploy each build into its own folder (for example,
  `/var/www/releases/2026-09-26/`) and point Nginx to it with a symlink. Rolling back just
  means switching the symlink to the previous release.
- **Automate deployments with a CI/CD pipeline** instead of copying files by hand. This
  reduces mistakes like deleting the wrong folder.
- **Check the deployment after every release.** Use `curl -I` to confirm 200 OK, and check
  that `index.html` exists before switching traffic to the new version.
- **Set up health checks and monitoring** that test the actual HTTP response, not just
  whether the Nginx service is running, and alert the team when the site returns errors.
- **Limit who can change production files**, and be careful with commands like `rm -rf`.
  Check the path twice before running them.
- **Keep the source code in Git**, so the app can always be rebuilt and redeployed even
  if the server files are lost.

---

# Task 8 — Security & Reliability Review

## Goal

Review and reflect on the security and reliability practices applied during this assignment.

### Security & Reliability Notes

Answer the following in your own words:

**1. Why is SSH key-based authentication more secure than sharing passwords?**

A password is a short secret that can be guessed, brute-forced by bots, or leaked when it is
shared. SSH keys work as a pair. The private key stays on my laptop (my `.pem` file), and the
server only holds the public key. The private key is never sent over the network, and it is
far too long to guess, so brute-force attacks are practically impossible.
Each person can also have their own key, so access can be removed for one person without
affecting anyone else. With a shared password, everyone would need a new one.
On my EC2 instance, password login is disabled by default, and access only works with my key.

---

**2. Why should only required ports be open on a production server?**

Every open port is a possible entry point for attackers. Bots constantly scan the internet
for open ports and vulnerable services. I saw this in my Nginx access logs, where unknown IPs
requested paths like `/wp-login.php` within hours.
My server only needs port 80 for the website and port 22 for SSH, so those are the only
ports my security group allows. Closing everything else reduces the attack surface. Even if
a service on another port has a vulnerability, nobody can reach it from outside.
For better security, SSH (22) should be limited to my own IP instead of "anywhere".

---

**3. Why is it important for Nginx to be enabled on boot?**

Servers restart for many reasons, such as updates, maintenance, AWS hardware issues or a
crash. If Nginx is not enabled, it stays stopped after a reboot, and the website remains down
until someone notices and starts it by hand.
`systemctl enable nginx` makes sure Nginx starts automatically every time the server boots,
so the site comes back without anyone having to step in. I confirmed this with
`systemctl is-enabled nginx`, which returned `enabled`.

---

**4. What are the risks of sharing secrets, keys, or credentials publicly?**

Anyone who gets a secret gets the same access as its owner. For example:
- A leaked `.pem` key lets someone log in to my server and change or delete my files, or use
  the server for attacks.
- Leaked AWS access keys let attackers create resources in my account, often expensive
  crypto-mining servers, which can lead to a huge bill within hours.
- Bots scan public GitHub repositories for keys within minutes of a push, and deleting the
  file later doesn't help, because it stays in the Git history.
To prevent this, I never commit keys or `.env` files. I add them to `.gitignore` and blur IPs
or tokens in screenshots. If a key does leak, I would delete or rotate it immediately.

---

**5. Why should cloud resources be stopped or terminated when they are no longer needed?**

- **Cost:** cloud billing is pay-as-you-go. Running instances, attached storage and public
  IPv4 addresses keep costing money, or use up Free Tier credits, even when nobody is using
  them.
- **Security:** forgotten servers stop getting updates and still have open ports, so they
  become easy targets for attackers.
- **Clean management:** unused resources make an account messy and harder to audit.
Stopping an instance pauses compute charges, but the EBS disk is still billed. Terminating it
removes everything. After finishing this assignment, I will terminate my EC2 instance and
check the Billing console to make sure nothing is left running.
---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/siththi-waseema-62a0b0187_cloudcomputing-learningjourney-ugcPost-7509653154427453440-OgbM/?utm_source=share&utm_medium=member_desktop&rcm=ACoAACv5boIBHr4DjAudB4kGRvqYrehYIp1o_Io

---

#### Screenshot — Published LinkedIn post

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2003/8.png)

---

# Submission Instructions

- Add all required screenshots in your submission
- Full name must be visible in required screenshots
- Do not expose sensitive information (keys, passwords, account IDs)

---

# Completion Checklist

- [ ] Task 1: Screenshots (browser, ip a, ss -tulpen, ufw status) + Notes answered
- [ ] Task 2: Screenshots (nginx status, nginx -t, ss port 80) + Notes answered
- [ ] Task 3: Screenshots (access log, error log, journalctl) + Notes answered
- [ ] Task 4: Screenshots (uptime, free -h, df -h, du -sh) + Notes answered
- [ ] Task 5: Screenshots (ls html, grep deployed by, grep try_files) + Notes answered
- [ ] Task 6: Screenshots (nginx -t fail, nginx -t pass, curl recovery) + Notes answered
- [ ] Task 7: Screenshots (curl failure, curl recovery) + Notes answered
- [ ] Task 8: Security & Reliability Notes answered
- [ ] LinkedIn post published and URL submitted
- [ ] Full Name visible in all required screenshots
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