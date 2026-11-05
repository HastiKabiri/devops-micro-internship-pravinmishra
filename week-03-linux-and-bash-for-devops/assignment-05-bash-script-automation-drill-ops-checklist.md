# Assignment 5 — Bash Script Automation Drill (OPS Checklist)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will practice Bash scripting by building a series of small automation scripts covering environment setup, variables, arrays, loops, file conditionals, if-else logic, and functions. These scripts form the foundation of real-world Linux automation used in DevOps, cloud, and production support environments.

---

# Task 1 — Bash Environment & Workspace Setup

## Goal

Verify that Bash is available on your system and create a clean workspace for this assignment.

### Evidence

#### Screenshot 1 — Output of `echo $SHELL` and `bash --version`

![](screenshots/assign%205/1.png)

---

#### Screenshot 2 — Output of `pwd` and `ls -lah` showing the scripts directory

![](screenshots/assign%205/2.png)

---

### Notes

Answer the following in your own words:

**1. What is Bash?**

Bash (Bourne Again SHell) is a command-line shell and scripting language. It reads the commands I type, or the commands in a script file, and asks the operating system to run them. It is the default shell on most Linux systems. In DevOps we use it to automate repetitive work like setting up servers, deploying apps, checking logs and running backups.

---

**2. What is the difference between shell and Bash?**

A shell is the general name for any program that sits between the user and the operating system and runs commands. There are many shells, such as `sh`, `bash`, `zsh`, `fish` and `ksh`. Bash is one specific shell. It builds on the original Bourne shell (`sh`) and adds features like command history, tab completion, arrays and better scripting syntax. So every Bash is a shell, but not every shell is Bash.

---

**3. Why is it important to confirm the Bash version before writing scripts?**

Different Bash versions support different features. For example, associative arrays (`declare -A`) and some string operations only work in Bash 4 or newer, and macOS still ships Bash 3.2 by default. If I write a script with newer features and it runs on a server with an older version, it can fail or behave unexpectedly. Checking the version first (`bash --version`) means my scripts run the same way on every machine. On my system it is GNU bash 5.3.15.

---

# Task 2 — Your First Bash Script

## Goal

Create your first Bash script, make it executable, and run it from the terminal.

### Evidence

#### Screenshot 1 — Content of `first-script.sh`


![](screenshots/assign%205/2.1.png)
---

#### Screenshot 2 — Output of `./first-script.sh`

![](screenshots/assign%205/2.2.png)

---

#### Screenshot 3 — Output of `ls -l first-script.sh` showing executable permission

![](screenshots/assign%205/2.3.png)
---

### Notes

Answer the following in your own words:

**1. What is the purpose of `#!/bin/bash`?**

`#!/bin/bash` is called the shebang line, and it must be the first line of the script. It tells the operating system which interpreter should run the file, in this case Bash at `/bin/bash`. Without it, the system might run the script with a different shell (like `sh`), and some Bash-specific commands could fail.

---

**2. Why do we use `chmod +x` before running a script?**

A new file is not allowed to run as a program by default. It only has read and write permissions. `chmod +x` adds the execute (`x`) permission, so the file can be run directly with `./first-script.sh`. After running it, `ls -l` shows `-rwxr-xr-x`, and the `x` confirms the script is executable.

---

**3. What is the difference between running a script using `./script.sh` and `bash script.sh`?**

- `./script.sh` runs the file directly. The script needs execute permission (`chmod +x`), and the system uses the shebang line to decide which interpreter runs it. The `./` means "this file in the current directory".
- `bash script.sh` starts Bash and passes the file to it to read. The script does not need execute permission, and the shebang line is ignored because we already chose Bash ourselves.

---

# Task 3 — Variables: User Information Script

## Goal

Use variables to store and display user-related information.

### Evidence

#### Screenshot 1 — Content of `user-info.sh`

![](screenshots/assign%205/3.1.png)

---

#### Screenshot 2 — Output of `./user-info.sh`

![](screenshots/assign%205/3.2.png)

---

### Notes

Answer the following in your own words:

**1. What is a variable in Bash?**

A variable is a name that stores a value, like text or a number, so I can reuse it later in the script. In `user-info.sh`, `full_name="Hasti Kabiri"` stores my name in a variable called `full_name`. If I need to change the value, I only change it in one place instead of everywhere it is used.

---

**2. Why should we avoid spaces around the `=` sign when creating variables?**

Bash uses spaces to separate a command from its arguments. If I write `full_name = "Hasti Kabiri"`, Bash thinks `full_name` is a command and `=` and `"Hasti Kabiri"` are its arguments, so it shows an error like `full_name: command not found`. Writing `full_name="Hasti Kabiri"` with no spaces tells Bash it is a variable assignment.

---

**3. How do you access the value stored inside a Bash variable?**

I put a `$` in front of the variable name, for example `echo "Full Name: $full_name"`. I can also write `${full_name}`, which is useful when the variable is next to other text. It is good practice to put the variable inside double quotes (`"$full_name"`) so values with spaces are kept together.

---

# Task 4 — Arrays & Loops: Tools Checklist Script

## Goal

Use arrays and loops to print a checklist of tools used in Bash scripting.

### Evidence

#### Screenshot 1 — Content of `tools-checklist.sh`

![](screenshots/assign%205/4.1.png)
---

#### Screenshot 2 — Output of `./tools-checklist.sh`

![](screenshots/assign%205/4.2.png)

---

### Notes

Answer the following in your own words:

**1. What is an array in Bash?**

An array is a variable that holds a list of values instead of just one. In my script, `tools=("bash" "nano" "chmod" "echo" "ls" "pwd")` stores six tool names in one variable called `tools`. Each item has a position number (index) starting from 0, so `${tools[0]}` is `bash`.

---

**2. Why are arrays useful in scripts?**

Arrays keep related values together in one place, like a list of tools, servers, users or files. Instead of creating a separate variable for each item, I can store them all in one array and process them with a loop. To add or remove an item, I only change the array, and the rest of the script still works.

---

**3. What does `"${tools[@]}"` mean?**

It means "all the items in the `tools` array". The `@` selects every element. The double quotes make sure each item is treated as a separate value, even if an item contains spaces.

---

**4. What is the purpose of the `for` loop in this script?**

The `for` loop goes through the array one item at a time. On each round, the current item is stored in the `tool` variable and the `echo` line prints it. This prints all six tools without writing six separate `echo` commands.

---

# Task 5 — Loops: Number Counter Script

## Goal

Use loops to repeat a task multiple times.

### Evidence

#### Screenshot 1 — Content of `counter.sh`

![](screenshots/assign%205/5.1.png)

---

#### Screenshot 2 — Output of `./counter.sh`

![](screenshots/assign%205/5.2.png)


---

### Notes

Answer the following in your own words:

**1. What is a loop?**

A loop is a block of code that repeats the same commands several times. In my script, the `for` loop runs the `echo "Step $number completed"` line once for each number in the list, changing the value of `number` each time.

---

**2. Why do we use loops in Bash scripting?**

Loops save us from writing the same command again and again. In DevOps we often need to do the same task many times, like checking several servers, processing many log files or creating multiple users. A loop does this automatically, keeps the script short and makes mistakes less likely.

---

**3. How many times did the loop run in your script?**

The loop ran 5 times, once for each number in `1 2 3 4 5`. The output showed "Step 1 completed" through "Step 5 completed", then "Loop completed successfully".

---

**4. What would you change if you wanted the loop to run 10 times?**

I would change the list of numbers in the `for` line. I could write `for number in 1 2 3 4 5 6 7 8 9 10`, or use the shorter range `for number in {1..10}`, which creates the numbers 1 to 10 for me.

---

# Task 6 — Files & Conditionals: File Validation Script

## Goal

Use file checks and conditionals to verify whether files and directories exist.

### Evidence

#### Screenshot 1 — Output of `ls -lah ../test-folder`

![](screenshots/assign%205/6.1.png)


---

#### Screenshot 2 — Content of `file-check.sh`

![](screenshots/assign%205/6.2.png)

---

#### Screenshot 3 — Output of `./file-check.sh`

![](screenshots/assign%205/6.3.png)

---

### Notes

Answer the following in your own words:

**1. What does `-d` check in Bash?**

`-d` checks whether a path exists and is a directory (folder). In my script, `[ -d "$directory_path" ]` is true because `../test-folder` exists, so it printed "Directory exists: ../test-folder".

---

**2. What does `-f` check in Bash?**

`-f` checks whether a path exists and is a regular file (not a directory). In my script, `[ -f "$file_path" ]` is true because `student-info.txt` was created with `touch`, so it printed "File exists: ../test-folder/student-info.txt".

---

**3. Why should file and directory paths be stored in variables?**

When a path is stored in a variable, I write it only once at the top of the script. If the path changes, I update one line instead of searching through the whole script. It also makes the script easier to read, because `$file_path` is clearer than repeating a long path, and it avoids typing mistakes when the same path is used many times.

---

**4. What happens if the file does not exist?**

The `-f` condition becomes false, so the script skips the `then` part and runs the `else` part. It prints "File does not exist: ../test-folder/student-info.txt" instead of crashing. This lets the script handle missing files safely, for example by showing a warning or creating the file.

---

# Task 7 — Conditionals: Pass or Retry Script

## Goal

Use if-else conditionals to make decisions based on a variable value.

### Evidence

#### Screenshot 1 — Content of `score-check.sh` with `score=85`

![](screenshots/assign%205/7.1.png)

---

#### Screenshot 2 — Output showing `Result: Pass`

![](screenshots/assign%205/7.2.png)

---

#### Screenshot 3 — Content of `score-check.sh` with `score=55`

![](screenshots/assign%205/7.3.png)

---

#### Screenshot 4 — Output showing `Result: Retry`

![](screenshots/assign%205/7.4.png)

---

### Notes

Answer the following in your own words:

**1. What is the purpose of if-else in Bash?**

if-else lets a script make decisions. It checks a condition: if the condition is true, it runs the commands after `then`, and if it is false, it runs the commands after `else`. In my script, it decides whether to print "Result: Pass" or "Result: Retry" based on the score.

---

**2. What does `-ge` mean?**

`-ge` means "greater than or equal to" and is used to compare numbers. `[ "$score" -ge 70 ]` is true when the score is 70 or more. With `score=85` the result was Pass, and with `score=55` the result was Retry.

---

**3. Why should conditions be tested with different values?**

Testing with only one value only proves that one path works. By testing with 85 (Pass) and 55 (Retry), I confirmed that both the `then` and `else` parts work correctly. Testing different values, including edge cases like exactly 70, helps find bugs before the script is used for real.

---

**4. How can conditionals help in automation scripts?**

Conditionals let automation scripts react to the situation instead of always doing the same thing. For example, a script can check if a service is running and restart it if not, check disk usage and send an alert if it is too high, or only deploy if all tests passed. This makes scripts safer and smarter.

---

# Task 8 — Functions: Final Bash Automation Script

## Goal

Create a final Bash script using functions to organize reusable code.

### Evidence

#### Screenshot 1 — Content of `final-automation.sh`

![](screenshots/assign%205/8.1.png)

---

#### Screenshot 2 — Output of `./final-automation.sh`

![](screenshots/assign%205/8.2.png)

---

#### Screenshot 3 — Output of `ls -lah` showing all created scripts

![](screenshots/assign%205/8.3.png)

---

### Notes

Answer the following in your own words:

**1. What is a function in Bash?**

A function is a named block of code that does one specific job. I define it once, for example `print_header() { ... }`, and then run it by writing its name, `print_header`. The code inside a function does not run until the function is called.

---

**2. Why are functions useful in scripts?**

Functions split a long script into small, organised pieces, each with a clear job. This makes the script easier to read, test and fix. A function can also be called many times without copying the same code again, and if I need to change how something works, I only change it inside that one function.

---

**3. Which functions did you create in this script?**

I created four functions:
- `print_header`: prints the assignment title between two separator lines.
- `print_user_details`: prints my full name and the assignment name.
- `check_files`: checks that the test folder and the `student-info.txt` file exist.
- `print_tools`: prints every tool from the `tools` array.

---

**4. How does this final script combine variables, arrays, loops, conditionals, files, and functions?**

- **Variables** store my name, the assignment name and the folder and file paths.
- **An array** (`tools`) stores the list of tools.
- **A loop** (`for tool in "${tools[@]}"`) goes through the array and prints each tool.
- **Conditionals** (`if` / `else`) decide whether to print "passed" or "failed".
- **File checks** (`-d` and `-f`) test if the folder and file really exist.
- **Functions** wrap each of these jobs into a separate block, and they are called in order at the end of the script.

Together they make one organised automation script, like a small version of an ops checklist used in real DevOps work.

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://lnkd.in/p/g4ifPANg


---

#### Screenshot — Published LinkedIn post

![](screenshots/assign%205/linkedin%20post.png)

---

# Submission Instructions

- Add all required screenshots in your submission
- Full name must be visible in required screenshots
- All script files must be created and run successfully
- Required notes must be answered clearly for every task
- Do not expose sensitive information (keys, passwords, credentials)

---

# Completion Checklist

- [✅] Task 1: Environment setup verified, workspace created (Screenshots 1–2, Notes answered)
- [✅] Task 2: First script created, executed, permissions verified (Screenshots 1–3, Notes answered)
- [✅] Task 3: Variables script created and run (Screenshots 1–2, Notes answered)
- [✅] Task 4: Arrays and loops script created and run (Screenshots 1–2, Notes answered)
- [✅] Task 5: Counter loop script created and run (Screenshots 1–2, Notes answered)
- [✅] Task 6: File validation script created and run (Screenshots 1–3, Notes answered)
- [✅] Task 7: Pass/Retry conditional script tested with both values (Screenshots 1–4, Notes answered)
- [✅] Task 8: Final automation script created and run (Screenshots 1–3, Notes answered)
- [✅] All scripts run without errors
- [✅] Full Name visible in all required screenshots
- [✅] LinkedIn post published and URL submitted
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