# Assignment 5 — Bash Script Automation Drill (OPS Checklist)

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will practice Bash scripting by building a series of small automation scripts covering environment setup, variables, arrays, loops, file conditionals, if-else logic, and functions. These scripts form the foundation of real-world Linux automation used in DevOps, cloud, and production support environments.

---

# Task 1 — Bash Environment & Workspace Setup

## Goal

Verify that Bash is available on your system and create a clean workspace for this assignment.

### Evidence

#### Screenshot 1 — Output of `echo $SHELL` and `bash --version`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/1.png)

---

#### Screenshot 2 — Output of `pwd` and `ls -lah` showing the scripts directory

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/2.png)

---

### Notes

Answer the following in your own words:

**1. What is Bash?**

Bash (Bourne Again SHell) is a command-line program that reads the commands I type and asks
the Linux system to run them. It is also a scripting language. I can write many commands in
a `.sh` file, and add variables, conditions (`if`) and loops (`for`, `while`), to automate
tasks like deployments, backups and health checks. It is the default shell on most Linux
systems, including Ubuntu.

---

**2. What is the difference between shell and Bash?**

"Shell" is the general name for any program that lets users talk to the operating system by
typing commands. Bash is one specific shell. Other shells include `sh` (the original Bourne
shell), `zsh` (the default on macOS), `fish` and `dash`.
All Bash is a shell, but not every shell is Bash. Different shells have different features
and syntax, so a script written for Bash might not work correctly in another shell.

---

**3. Why is it important to confirm the Bash version before writing scripts?**

Newer Bash versions add features that older ones don't have. For example, associative arrays
(`declare -A`) need Bash 4 or later, and some string and array features only exist in newer
releases. If I use a newer feature and the script later runs on a server with an older Bash,
it could fail or behave in unexpected ways.
Checking the version first means I know which features I can safely use, so my script works
the same way on every server where it runs.

---

# Task 2 — Your First Bash Script

## Goal

Create your first Bash script, make it executable, and run it from the terminal.

### Evidence

#### Screenshot 1 — Content of `first-script.sh`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/3.png)

---

#### Screenshot 2 — Output of `./first-script.sh`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/4.png)

---

#### Screenshot 3 — Output of `ls -l first-script.sh` showing executable permission

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/5.png)

---

### Notes

Answer the following in your own words:

**1. What is the purpose of `#!/bin/bash`?**

This line is called the **shebang**. It must be the first line of the script, and it tells
the system which program should run the file. Here, it says "run this script with Bash, which
is located at `/bin/bash`."
Without it, the script might be run by a different shell, such as `sh` or `dash`, and any
Bash-specific syntax could fail. It also makes clear to anyone reading the file what kind of
script it is.

---

**2. Why do we use `chmod +x` before running a script?**

In Linux, new files are not executable by default. They can only be read and written. This
is a safety feature that stops random files from being run as programs.
`chmod +x` adds the **execute (x)** permission, which allows the file to be run directly as
a program. Before `chmod +x`, `ls -l` showed `-rw-rw-r--`. Afterwards, it showed
`-rwxrwxr-x`. Without execute permission, `./first-script.sh` fails with "Permission denied".

---

**3. What is the difference between running a script using `./script.sh` and `bash script.sh`?**

- **`./script.sh`** runs the file directly as a program. It needs **execute permission**, and
  the system uses the **shebang line** to decide which interpreter to use. The `./` means
  "the file in the current folder". Without it, Linux only searches the folders listed in
  `$PATH`.
- **`bash script.sh`** starts Bash and passes the file to it to read. It **does not need
  execute permission**, and it **ignores the shebang**, because I have already chosen Bash.

Both run the script in a new Bash process. `./script.sh` is the normal way to run finished
scripts. `bash script.sh` is useful for testing, or when I can't change a file's permissions.

---

# Task 3 — Variables: User Information Script

## Goal

Use variables to store and display user-related information.

### Evidence

#### Screenshot 1 — Content of `user-info.sh`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/6.png)

---

#### Screenshot 2 — Output of `./user-info.sh`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/7.png)

---

### Notes

Answer the following in your own words:

**1. What is a variable in Bash?**

A variable is a named place to store a value, like a labelled box. I give it a name and put
a value inside it, for example `full_name="Waseema"`. Then I can reuse that value anywhere
in the script by using the name.
In my `user-info.sh` script, I stored my name, course, topic and learning goal in variables,
and then printed them with `echo`. Variables make scripts easier to read and update. If a
value changes, I only need to edit it in one place, not everywhere it is used.
By default, Bash treats every variable's value as text (a string).

---

**2. Why should we avoid spaces around the `=` sign when creating variables?**

Bash uses spaces to separate a command from its arguments. With no spaces, Bash reads
`full_name="Waseema"` as a variable assignment. If I write `full_name = "Waseema"`, Bash
thinks `full_name` is a **command** and that `=` and `"Waseema"` are its arguments. It
then fails with an error like `full_name: command not found`.
So the rule is: no spaces on either side of `=` when creating a variable. This is
different from many other programming languages, where spaces around `=` are allowed.

---

**3. How do you access the value stored inside a Bash variable?**

I put a dollar sign `$` in front of the variable's name. For example, `echo "$full_name"`
prints the value stored in `full_name`. I can also write it as `${full_name}`. The curly
braces help when the variable is followed directly by other text, for example
`"${topic_name}s"`.
It is good practice to put variables inside **double quotes** (`"$var"`). Double quotes keep
values with spaces together, like my learning goal sentence. Single quotes
(`'$full_name'`) would print the text `$full_name` literally instead of its value.

---

# Task 4 — Arrays & Loops: Tools Checklist Script

## Goal

Use arrays and loops to print a checklist of tools used in Bash scripting.

### Evidence

#### Screenshot 1 — Content of `tools-checklist.sh`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/8.png)
---

#### Screenshot 2 — Output of `./tools-checklist.sh`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/9.png)

---

### Notes

Answer the following in your own words:

**1. What is an array in Bash?**

An array is a single variable that holds a **list of values** instead of just one. Each value
is stored at a numbered position called an index, which starts at 0.
In my script, `tools=("bash" "nano" "chmod" "echo" "ls" "pwd")` creates an array with 6 items.
`${tools[0]}` is `bash`, `${tools[1]}` is `nano`, and so on. `${#tools[@]}` gives the number
of items, which is 6.

---

**2. Why are arrays useful in scripts?**

Arrays keep related values together under one name, so I don't need a separate variable for
each item (`tool1`, `tool2`, `tool3`...). Combined with a loop, the same action can be run on
every item with just a few lines of code.
They make scripts shorter, cleaner and easier to update. To add a new tool, I just add it to
the array, and the loop handles it automatically. In real DevOps work, arrays are used for
lists of servers, packages to install, services to check or files to back up.

---

**3. What does `"${tools[@]}"` mean?**

It means "all the items in the `tools` array, each one kept as a separate value."
- `tools` is the array name.
- `[@]` means every item, not just one.
- `${...}` is needed to access an array. `$tools` alone would only give the first item.
- The **double quotes** keep each item whole. Without them, an item containing a space, such
  as `"visual studio"`, would be split into two separate items.

---

**4. What is the purpose of the `for` loop in this script?**

The `for` loop goes through the array one item at a time. On each round, the current item is
stored in the variable `tool`, and the commands between `do` and `done` run with that value.
In my script, the loop ran 6 times and printed "Tool available for practice: <tool>" for
each tool. Without a loop, I would need six separate `echo` commands. The loop does the same
work automatically, no matter how many items the array has.

---

# Task 5 — Loops: Number Counter Script

## Goal

Use loops to repeat a task multiple times.

### Evidence

#### Screenshot 1 — Content of `counter.sh`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/10.png)

---

#### Screenshot 2 — Output of `./counter.sh`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/11.png)

---

### Notes

Answer the following in your own words:

**1. What is a loop?**

A loop is a block of code that runs the same commands over and over, once for each item in a
list or until a condition is met. In Bash, the commands to repeat go between `do` and `done`.
In my script, `for number in 1 2 3 4 5` takes each number in turn, stores it in the variable
`number`, and runs `echo "Step $number completed"` for it.
---

**2. Why do we use loops in Bash scripting?**

Loops let me automate repetitive work without writing the same command many times. This
makes scripts shorter, easier to read and easier to change. If the number of items changes,
I only update the list, not the code.
In real DevOps tasks, loops are used to check many servers, restart several services, process
every log file in a folder, or keep retrying a health check until it succeeds.
---

**3. How many times did the loop run in your script?**

**5 times**, once for each number in the list `1 2 3 4 5`. The output showed "Step 1
completed" through "Step 5 completed". Then the script left the loop and printed "Loop
completed successfully" once.
---

**4. What would you change if you wanted the loop to run 10 times?**

I would change the list the loop goes through. The simplest way is to use a range:
`for number in {1..10}`
Other ways to do it:
- List every number: `for number in 1 2 3 4 5 6 7 8 9 10`
- Use a C-style loop: `for ((number=1; number<=10; number++))`
- Use `seq`: `for number in $(seq 1 10)`
The `{1..10}` range is the cleanest, because I just change the end number to control how
many times the loop runs.
---

# Task 6 — Files & Conditionals: File Validation Script

## Goal

Use file checks and conditionals to verify whether files and directories exist.

### Evidence

#### Screenshot 1 — Output of `ls -lah ../test-folder`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/12.png)

---

#### Screenshot 2 — Content of `file-check.sh`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/13.png)

---

#### Screenshot 3 — Output of `./file-check.sh`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/14.png)

---

### Notes

Answer the following in your own words:

**1. What does `-d` check in Bash?**

`-d` checks whether a path exists **and is a directory** (folder). `[ -d "$directory_path" ]`
is true only if `../test-folder` exists and is a folder. If the path doesn't exist, or if it's
a regular file, the test is false and the `else` part runs.
In my script, it printed "Directory exists: ../test-folder" because I created the folder with
`mkdir -p test-folder`.

---

**2. What does `-f` check in Bash?**

`-f` checks whether a path exists **and is a regular file**. It is false for folders and for
paths that don't exist. `[ -f "$file_path" ]` was true for `student-info.txt`, which I created
with `touch`. The file is empty, but `-f` only checks that the file exists, not what it
contains. To check that a file has content, I would use `-s`.

---

**3. Why should file and directory paths be stored in variables?**

- **Easy to change:** if the location changes, I update it in one place at the top of the
  script instead of searching through every line.
- **Fewer mistakes:** typing the same long path many times makes typos more likely. A
  variable is written once and reused.
- **Easier to read:** a name like `$file_path` explains what the value is for.
- **Reusable:** the same script can check different files just by changing the variable, or
  by passing the path in as an argument.
I also put the variables in double quotes (`"$file_path"`), so paths with spaces still work
correctly.

---

**4. What happens if the file does not exist?**

The `-f` test returns false, so Bash skips the `then` part and runs the `else` part. The
script prints "File does not exist: ../test-folder/student-info.txt" and continues running
without crashing. The `if` statement handles the missing file safely.
I could test this by deleting the file with `rm ../test-folder/student-info.txt`, running the
script to see the "does not exist" message, and then recreating the file with `touch`.
Because the paths are relative (`../`), the script must be run from inside the `scripts`
folder. If it's run from another folder, even an existing file would be reported as missing.

---

# Task 7 — Conditionals: Pass or Retry Script

## Goal

Use if-else conditionals to make decisions based on a variable value.

### Evidence

#### Screenshot 1 — Content of `score-check.sh` with `score=85`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/15.png)
---

#### Screenshot 2 — Output showing `Result: Pass`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/16.png)

---

#### Screenshot 3 — Content of `score-check.sh` with `score=55`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/17.png)

---

#### Screenshot 4 — Output showing `Result: Retry`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/18.png)

---

### Notes

Answer the following in your own words:

**1. What is the purpose of if-else in Bash?**

`if-else` lets a script make decisions. It checks a condition. If the condition is true, the
commands after `then` run. If it's false, the commands after `else` run. `fi` marks the end
of the block.
In my script, the condition checked whether `score` was 70 or more. With `score=85`, the
condition was true and the script printed "Result: Pass". With `score=55`, it was false and
the script printed "Result: Retry". It was the same script, with different behavior
depending on the value.

---

**2. What does `-ge` mean?**

`-ge` means **"greater than or equal to"**, and it compares numbers. `[ "$score" -ge 70 ]` is
true when the score is 70 or higher.
Other number comparisons in Bash are `-eq` (equal), `-ne` (not equal), `-gt` (greater than),
`-lt` (less than) and `-le` (less than or equal).
Inside `[ ]`, Bash uses these letter operators for numbers, not `>=`. Using `>` inside
`[ ]` would redirect output to a file instead of comparing the values.

---

**3. Why should conditions be tested with different values?**

Testing only one value proves only one path of the script works. By running with `85` and
then `55`, I confirmed both the `then` part (Pass) and the `else` part (Retry) work
correctly.
It's also good to test **boundary values**, like exactly `70` (should Pass) and `69` (should
Retry). Mistakes often happen at the edge, for example writing `-gt` instead of `-ge`
would make a score of 70 fail. Testing different values catches these bugs before the
script is used for real.

---

**4. How can conditionals help in automation scripts?**

Conditionals let scripts react to what they find, instead of blindly running every command.
For example:
- **Check before acting:** only deploy if `nginx -t` passes, or only copy files if the
  build folder exists.
- **Health checks:** if a website doesn't return 200 OK, restart the service or send an
  alert.
- **Resource checks:** if disk usage is above 80%, clean old logs or warn the team.
- **Safety:** stop the script if a required file or variable is missing, instead of
  continuing and causing damage.
This makes automation safer and more reliable, because the script handles problems on its own
without needing a person to watch it.

---

# Task 8 — Functions: Final Bash Automation Script

## Goal

Create a final Bash script using functions to organize reusable code.

### Evidence

#### Screenshot 1 — Content of `final-automation.sh`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/19.png)

---

#### Screenshot 2 — Output of `./final-automation.sh`

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/20.png)

---

#### Screenshot 3 — Output of `ls -lah` showing all created scripts

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/21.png)

---

### Notes

Answer the following in your own words:

**1. What is a function in Bash?**

A function is a named block of code that I write once and can run whenever I need it, just
by typing its name. It is defined like this:

`function_name() { commands; }`

It doesn't do anything until it is **called**. In my script, I defined four functions first
and then called them at the bottom in the order I wanted. A function must be defined before
it is called, which is why the calls come at the end of the script.

---

**2. Why are functions useful in scripts?**

- **Organization:** each function does one job, so the script is split into clear sections
  instead of one long list of commands.
- **Reusability:** I can call the same function many times without copying the code again.
- **Readability:** names like `check_files` explain what the code does. The bottom of my
  script reads almost like a plan: header, user details, file check, tools.
- **Easier maintenance and debugging:** if the file check has a problem, I only need to
  look inside `check_files`. I can also turn a step off by removing just one function call.

---

**3. Which functions did you create in this script?**

1. **`print_header`**: prints a decorative header with the assignment name.
2. **`print_user_details`**: prints my full name and the assignment name.
3. **`check_files`**: checks whether `../test-folder` and `student-info.txt` exist, and
   prints a passed or failed message for each.
4. **`print_tools`**: loops through the `tools` array and prints each tool.
After calling all four, the script prints "Final Bash automation script completed".

---

**4. How does this final script combine variables, arrays, loops, conditionals, files, and functions?**

- **Variables:** `full_name`, `assignment_name`, `directory_path` and `file_path` store the
  values, and the functions reuse them.
- **Arrays:** `tools=(...)` stores the list of tools in one variable.
- **Loops:** the `for` loop in `print_tools` goes through `"${tools[@]}"` and prints each
  item.
- **Conditionals:** `if-else` in `check_files` decides whether to print a "passed" or
  "failed" message.
- **Files:** the `-d` and `-f` tests check whether the folder and file exist.
- **Functions:** all of the above is organized into four reusable functions, called in
  order.
This script brings together everything I practiced in Tasks 3–7 in one organized
automation. It's the same structure used in real DevOps scripts: set up variables, define
functions for each job, then run them in order.

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

https://www.linkedin.com/posts/siththi-waseema-62a0b0187_dmibypravinmishra-siththiwaseema-devops-share-7509700322177654784-Avh5/?utm_source=share&utm_medium=member_desktop&rcm=ACoAACv5boIBHr4DjAudB4kGRvqYrehYIp1o_Io


---

#### Screenshot — Published LinkedIn post

![Week 02–linux-and-bash-for-devops](screenshots/Assignment%2005/ss.png)

---

# Submission Instructions

- Add all required screenshots in your submission
- Full name must be visible in required screenshots
- All script files must be created and run successfully
- Required notes must be answered clearly for every task
- Do not expose sensitive information (keys, passwords, credentials)

---

# Completion Checklist

- [ ] Task 1: Environment setup verified, workspace created (Screenshots 1–2, Notes answered)
- [ ] Task 2: First script created, executed, permissions verified (Screenshots 1–3, Notes answered)
- [ ] Task 3: Variables script created and run (Screenshots 1–2, Notes answered)
- [ ] Task 4: Arrays and loops script created and run (Screenshots 1–2, Notes answered)
- [ ] Task 5: Counter loop script created and run (Screenshots 1–2, Notes answered)
- [ ] Task 6: File validation script created and run (Screenshots 1–3, Notes answered)
- [ ] Task 7: Pass/Retry conditional script tested with both values (Screenshots 1–4, Notes answered)
- [ ] Task 8: Final automation script created and run (Screenshots 1–3, Notes answered)
- [ ] All scripts run without errors
- [ ] Full Name visible in all required screenshots
- [ ] LinkedIn post published and URL submitted
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