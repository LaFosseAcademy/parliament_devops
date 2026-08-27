# Session 2 — Bash Scripting & Automation for DevOps — Trainer Script

A full day taking students from "I can move around the filesystem" to "I can write, name, and run real scripts that scaffold an application and automate a container build". This is written as a script — read it out, adapt where it feels natural, but the intent is that you could deliver this cold from the page.

---

## 📦 STARTER CODE — put this in the repo before training

Everything here goes into **`/bash-scripting-and-automation/starter-code`** before the session.

**Deliberately no working scripts in the main folder.** The entire point of today is that students *write* the scripts. What the repo provides is a **reference sheet**, a **dictionary template** they fill in themselves, and a **completed-code folder for catch-up only**.

<br>

**`README.md`**
```markdown
# Bash Scripting & Automation — Starter Code

## What's here

- **commands.md** — every command we use today, with a one-line
  explanation. Use it as a reference while you work.

- **my-command-dictionary.md** — you fill this in. By the end of the
  day you should have your own reference you'd actually use again.

- **completed-code/** — the finished versions of the scripts we build.
  Only open this if you've fallen behind and need to catch up.
  Reading the answer before trying is the fastest way to learn nothing.

## Setup

You need a real terminal:
- Mac: Terminal or iTerm
- Windows: WSL2 (Ubuntu), or Git Bash
- Or the Azure Cloud Shell in a browser

Check Docker works, we need it this afternoon:

    docker --version
```

<br>

**`commands.md`**
```markdown
# Bash Command Reference — Session 2

## Moving around (you already know these)

    pwd                 Print working directory — where am I?
    ls                  List files here
    ls -l               Long format — permissions, size, date
    ls -la              ...including hidden files (those starting with .)
    cd <folder>         Change directory
    cd ~                Go to your home folder
    cd ..               Go up one level

## Looking at files

    cat <file>          Print the whole file
    head -5 <file>      First 5 lines
    tail -5 <file>      Last 5 lines
    less <file>         Scroll through a file (q to quit)
    wc -l <file>        Count the lines

## Finding things

    grep "text" <file>  Show only lines containing "text"
    find . -name "*.js" Find files by name, from here downwards
    which <command>     Where does this command live?
    type <command>      What IS this command? (binary, alias, function)
    file <path>         What kind of file is this?

## Making things

    touch <file>        Create an empty file
    mkdir <folder>      Create a folder
    mkdir -p a/b/c      Create nested folders, no error if they exist
    cp <src> <dest>     Copy
    mv <src> <dest>     Move (or rename)
    rm <file>           Delete a file
    rm -r <folder>      Delete a folder and its contents

## Chaining commands

    a ; b               Run a, THEN b — regardless of whether a worked
    a && b              Run b ONLY IF a succeeded
    a || b              Run b ONLY IF a failed
    a | b               Send a's OUTPUT into b as INPUT
    a > file            Send a's output into a file (OVERWRITES)
    a >> file           Send a's output into a file (APPENDS)
    $(a)                Run a, and use its output right here

## Permissions

    chmod +x <file>     Allow this file to be run as a program
    chmod -x <file>     Remove that permission
    ls -l               Check what permissions a file has

## Script variables

    $1  $2  $3          First, second, third argument
    $@                  All arguments
    $#                  How many arguments
    $0                  The script's own name
    $?                  Exit code of the last command (0 = success)
    $USER  $HOME  $PATH Built-in environment variables

## Where programs live

    /bin                Essential system commands (ls, cp, cat)
    /usr/bin            Most normal user commands (git, grep, python)
    /usr/local/bin      Software you installed yourself
    ~/bin               YOUR personal scripts (we create this today)
    /etc                System configuration files
    /var                Variable data — logs, caches, web files
    /tmp                Temporary files, wiped on reboot
```

<br>

**`my-command-dictionary.md`** *(students fill this in — it's one of today's activities)*
```markdown
# My Command Dictionary

Fill this in as you go. The rule: **write the explanation in your own
words**, not copied from the notes. If you can't explain it simply,
you haven't understood it yet.

| Command | What it does (in MY words) | An example I actually ran |
|---|---|---|
| `pwd` | | |
| `ls -la` | | |
| `grep` | | |
| `wc -l` | | |
| `chmod +x` | | |
| `head` / `tail` | | |
| `which` | | |
| `echo $PATH` | | |
| `>` vs `>>` | | |
| `\|` (pipe) | | |
| `&&` vs `\|\|` | | |
| `$(...)` | | |
| `$?` | | |
| `set -e` | | |

## Three commands I found myself that weren't in the session

| Command | What it does | Where I found it |
|---|---|---|
| | | |
| | | |
| | | |
```

<br>

**`completed-code/`** — put the finished `scaffold`, `new-app` and `containerise` scripts here (the full versions appear in the Solution blocks later in this document). Mark the folder clearly as catch-up only.

<br>

---

## 💬 SLACK SNIPPETS — queue these before you start

Have these in a draft message. **Post reactively**, at the point in the session where they're needed, so students copy rather than mistype. Long heredoc-heavy blocks especially — transcribing those teaches nothing except typo tolerance.

| # | When | What to post |
|---|---|---|
| 1 | 09:45 | The chaining & pipelines exercise commands |
| 2 | 10:30 | The `hello.sh` file contents |
| 3 | 11:00 | The `~/bin` + PATH setup lines |
| 4 | 11:30 | The `if` / `for` / function shapes |
| 5 | 12:15 | **Scaffold stages 2–5** — the heredoc blocks. Post these; typing them is pure transcription |
| 6 | 14:15 | The PowerShell exercise commands |
| 7 | 15:15 | The `Dockerfile` and `.dockerignore` contents for the capstone |

Each is marked **💬 SLACK** in the body below.

---

## Organisation

### Audience

Students who have covered **Session 1 (Azure fundamentals)** and **Docker**, and who are comfortable with `cd`, `ls` and `mkdir` but have **not written a shell script before**.

They are experienced developers: **Node, Express, REST, MVC, SQL, Git, GitHub, Docker, frontend, unit and integration testing**. So the *contents* of the files we generate today will be completely familiar — an Express app with routers, controllers and models is their daily bread.

What is genuinely new:
- **The shell as a programming environment** rather than a place to type `cd`
- **Bash logic** — `if`, loops, functions. Conceptually familiar from JavaScript, but the syntax is alien and needs slow, careful explanation
- **The Linux filesystem** — where things live and why
- **Automation as a discipline**, not just convenience

**NOTE FOR TRAINERS** <br>
The trap with this room is assuming that because they can write JavaScript, bash will come easily. It doesn't. Bash's syntax is genuinely strange — `fi` to close an `if`, spaces mattering inside `[ ]`, `$` on read but not on assignment — and being a good developer doesn't help. Explain each construct as if it were a new language, because it is one. Where a JavaScript comparison helps, make it explicitly; where the analogy breaks, say so. <br>
**END OF NOTE**

### How this document is laid out

**Every command block has a *Run from* line above it**, so you always know which folder you should be in. Where a command changes directory, that's called out:

- Run: `mkdir server` → **cd inside**

The folders we use today:

| Folder | What it's for |
|---|---|
| `~/bash-training` | Scratch folder for all morning practice |
| `~/bin` | Where finished scripts get installed so they run by name |
| `~/countries-app`, `~/books-app` | The scaffolded applications we generate |

**Every activity has a `**Solution**` block** immediately after it.

### Duration & schedule

Full day, **09:00–17:00**, with a one-hour lunch and two 15-minute breaks.

**Hands-on time today: ~3 hours 45 minutes** across nine activities, every one with a solution in this document.

| Time | Section | Activity time |
|---|---|---|
| 09:00–09:15 | Welcome & recap | |
| 09:15–09:45 | The shell, and chaining commands | |
| 09:45–10:15 | Hands-on: chaining & pipelines | **30 min** |
| **10:15–10:30** | **Break** | |
| 10:30–11:00 | Your first script: shebang, permissions | **10 min** |
| 11:00–11:40 | The filesystem, PATH, and naming a script | **20 min** |
| 11:40–12:00 | Logic: conditionals, loops and functions | **10 min + challenge** |
| **12:00–12:15** | **Break** | |
| 12:15–13:00 | Building an app scaffold: the `countries` script | **45 min** |
| **13:00–14:00** | **Lunch** | |
| 14:00–14:15 | From scripting to automation | |
| 14:15–15:00 | PowerShell on the Azure Cloud Shell | **25 min** |
| **15:00–15:15** | **Break** | |
| 15:15–16:45 | Capstone: scaffold an app & automate a container build | **90 min** |
| 16:45–17:00 | Wrap-up & Q&A | |

### Set-up

#### For Trainers

**For the slidee presentation** <br>
**Open** the [Slidee](https://github.com/mklilley/slidee) presentation:

- Run `npx slidee` from within the `curriculum` folder to see the available slide decks (those with extension `.slides.md`)
- Click on `See your presentations`
- Open `Weeks X Y > CDO > Bash Scripting and Automation`
- Hit `s` to open the presenter notes
- Resource to be kept open on a tab throughout the day to refer to

#### For Students
- **Direct** students to the starter code for this module
  - [Student Facing Repo](https://github.com/LaFosseAcademy/cloud-devops-student-repos/tree/main)
- **Use**
  - **/bash-scripting-and-automation/starter-code**
- **Make sure**
  - Students have a **real terminal**: WSL2 on Windows, macOS Terminal, or Git Bash
  - **Docker** installed and working (`docker --version`) — needed for the afternoon capstone
  - Students can sign in to the **Azure Portal** — needed for the PowerShell section
  - A text editor they're comfortable with — VS Code ideally, `nano` is fine

**NOTE FOR TRAINERS — Windows** <br>
Windows students on **Git Bash** will hit two limits today: no `apt`, and no `tree`. Both are cosmetic — `apt` examples are illustrative and `ls -R` substitutes for `tree`. **WSL2 is strongly preferred** and worth requiring in the pre-session email, because the afternoon capstone needs Docker talking to a Linux filesystem. If someone is stuck on Git Bash, pair them with a WSL2 or Mac user for the capstone. <br>
**END OF NOTE**

## Learning objectives

- **Chain** commands with `;`, `&&`, `||`, pipes and redirection, and understand exit codes
- **Explain** the Linux filesystem layout, and what a "binary" actually is
- **Write** a bash script with a shebang, make it executable, and run it
- **Give** a script a name on the `PATH` so it runs like any built-in command
- **Use** variables, positional arguments, conditionals, loops and functions
- **Scaffold** a complete Express MVC application from a single script
- **Understand** what makes automation a DevOps *discipline*, not just a convenience
- **Navigate** the Azure Cloud Shell and run basic PowerShell, including the `Az` module
- **Build** their own command reference, in their own words

<br>

## Sequence

### 09:00–09:15 — Welcome & Recap: Where We Are

Morning. Quick recap, because today builds directly on two things you've done.

**Session 1** gave you Azure — resource groups, the Portal, the CLI, and one idea that matters more than the rest: **everything clickable has a scriptable equivalent**, because the Portal and the CLI are both just clients of the same API.

And you've done **Docker** — building images, running containers, pushing to a registry.

Today is the glue. It's about the **shell** and **scripting** — the single most-used skill a DevOps engineer has, and the thing that turns "I did a task once, by hand" into "this task now runs the same way every time, for everyone, forever."

Here's the shape of the day. This morning is pure bash: from chaining two commands together, up to a named script that scaffolds an entire Express application in one command. After lunch we talk about what automation actually *means*, look at PowerShell in the Azure Cloud Shell, and then spend the back half on a capstone (our practical conclusion) pulling bash, Docker and Azure together.

**One thing to say up front.** You are all experienced developers. You write JavaScript daily. **Bash will not feel like JavaScript.** It's a genuinely odd language — you close an `if` with `fi`, spaces inside brackets change the meaning, and variables have a `$` when you read them but not when you set them. None of that is you being slow. It's just strange. I'll go slowly through the strange bits.

You already know `cd`, `ls` and `mkdir`, so we're not at zero — but we'll definitely be seeing a lot of new commands today. 

**Everyone set up a scratch folder now:**

*(Run from `~/`)*
- Run: `mkdir -p ~/bash-training` → **cd inside** with `cd ~/bash-training`
- Confirm with `pwd`

And check Docker, since we need it after lunch:

*(Run from `~/bash-training`)*
```bash
docker --version
```

<br>
<br>

### 09:15–09:45 — The Shell, and Chaining Commands Together

Let's reframe what the shell actually *is*, because it changes how you think about everything else today.

When you type `ls` and hit enter, you're not talking to a magic box — you're talking to a program called a **shell**, and on almost every Linux server and container that shell is **bash**. Bash reads what you type, works out which command you meant, finds the program, runs it, shows you the output, and waits for the next line. That's the whole loop.

**ASK** <br>
When you type `ls`, what *is* `ls`? Where does it come from? <br>
**ANSWER** <br>
It's a **program** — an actual file sitting on disk, usually at `/bin/ls`. It isn't built into bash; bash goes and finds it. You can prove it: `which ls` prints the path. That's a genuinely useful mental shift — **commands are just files**, which is exactly why you'll be able to add your own to the list before lunch.

Before we chain anything, one idea to hold onto: **every command has an input, an output, and a place it sends errors.** By default output and errors both land on your screen. Almost everything clever today is redirecting where that output goes — into another command, or into a file.

The powerful bit is that bash lets you **combine** commands. You know the commands individually. Today is mostly about the operators *between* them.

**Running things in sequence with `;`**

A semicolon means "run this, **then** run that, regardless of what happened".

*(Run from `~/bash-training`)*
```bash
mkdir project ; cd project ; ls
```

Read the `;` as the word "then". The catch: it doesn't care if a command *failed*. If `mkdir project` fails because the folder already exists, bash still barrels on and tries to `cd` and `ls`.

*You are now inside `~/bash-training/project` — hop back out:*

*(Run from `~/bash-training/project`)*
```bash
cd ~/bash-training
```

**Running the next thing only if the last one worked, with `&&`**

The one you'll use constantly.

*(Run from `~/bash-training`)*
```bash
mkdir project2 && cd project2
```

`&&` means "only run the right-hand command **if** the left-hand one succeeded". Read it as "and-then-only-if-that-worked".

*(Run from `~/bash-training/project2`)*
```bash
cd ~/bash-training
```

Another everyday example — only install if the package list refreshed successfully:

*(Run from `~/bash-training` — WSL/Linux only, illustrative on Mac)*
```bash
sudo apt update && sudo apt install -y git
```

**Running something only if the last thing *failed*, with `||`**

The mirror image. Read `||` as "or-else".

*(Run from `~/bash-training`)*
```bash
cd project-that-does-not-exist || echo "Couldn't enter that folder"
```

You'll often see `&&` and `||` together as a tiny inline if/else:

*(Run from `~/bash-training`)*
```bash
cd project && echo "It worked" || echo "Couldn't enter project folder"
```

*(That one leaves you inside `project` — `cd ~/bash-training` again.)*

**ASK** <br>
What's the practical difference between `mkdir app ; cd app` and `mkdir app && cd app`? Why might the second save you from a nasty mistake? <br>
**ANSWER** <br>
With `;`, if `mkdir app` fails — folder already exists, or no permission — bash still runs `cd app`. With `&&`, a failed `mkdir` stops the chain, so you don't carry on operating in the **wrong directory**. In a script that then deletes or overwrites files, that's the difference between a clean run and wiping the wrong folder. This is not hypothetical; it's one of the classic ways people destroy things.

**Piping output from one command into another with `|`**

This is the idea that makes the shell feel like a superpower. A **pipe** (`|`) takes the *output* of the command on its left and feeds it as the *input* of the command on its right.

Meet the four small commands we'll pipe *into*, because they're new:

- `SLIDE ACROSS`

| Command | Does |
|---|---|
| `sort` | Puts lines in alphabetical order. `sort -r` reverses it |
| `head -5` | Keeps only the **first** 5 lines. (`tail -5` keeps the last 5) |
| `wc -l` | "word count, lines" — tells you **how many** lines there were |
| `grep "text"` | The filter. Keeps **only lines containing** "text". Think "search" |

`grep` is the single most-used command in a DevOps engineer's day, mostly for hunting through logs.

*Build the pipeline up one stage at a time on screen, so students watch the list shrink:*

*(Run from `~/bash-training`)*
```bash
ls /etc/
```
A long list of the system's configuration files. Now sort it:

*(Run from `~/bash-training`)*
```bash
ls /etc/ | sort
```
Now keep only the first five:

*(Run from `~/bash-training`)*
```bash
ls /etc/ | sort | head -5
```

Each command does one small job; the pipe stitches them into a **pipeline**, output flowing left to right. That's the Unix philosophy in a line: lots of small tools that do one thing well, joined together.

Now `grep` filtering rather than sorting:

*(Run from `~/bash-training`)*
```bash
ls /etc/ | grep conf
# Lists everything in /etc/keeps only lines containing 'conf'
ls /etc/ | grep conf | wc -l
# 'wc -l' counts the number of lines
```

Imagine we did a deployment and it didn't work, with a similar commands we could use 'grep' to search for errors, how many errors there are. 

**ASK** <br>
You've all chained array methods — `.filter().map().length`. How is a pipeline different, and how is it the same? <br>
**ANSWER** <br>
**Same idea:** each stage takes a collection, transforms it, and hands it on, so you compose small operations into a bigger one. **Different in one important way:** array methods pass *structured data* — objects with properties. A bash pipe passes **plain text, line by line**. That's why you need `grep` and `wc` rather than `.filter()` and `.length` — you're pattern-matching strings, not reading properties. Hold onto that, because this afternoon PowerShell does it the *other* way and passes real objects.

**ASK** <br>
Given what you now know about `grep` and `wc -l`, what does this do? <br>
```bash
cat access.log | grep "404" | wc -l
```
**ANSWER** <br>
"Read the log file, keep only the lines containing 404, then count them." You've counted your 404 errors without opening the file. That exact pipeline — `cat` a log, `grep` the error, `wc -l` to count — is something you'll type in real jobs constantly.

**Redirecting output into a file with `>` and `>>`**

By default a command prints to your screen. You can send that output into a file instead.

*(Run from `~/bash-training`)*
```bash
ls -la > listing.txt     # List hidden items in long format
cat listing.txt
```

Nothing appeared on screen for the first command — the output went into the file.

The crucial distinction:
- `>` **overwrites** — wipes the file and writes fresh
- `>>` **appends** — adds to the end, keeping what was there

*(Run from `~/bash-training`)*
```bash
echo "deployed" >> log.txt
echo "deployed again" >> log.txt
cat log.txt
```

Get those the wrong way round and you'll destroy a file you meant to add to. This bites everyone once.

*(Run from `~/bash-training`)*
```bash
echo "mistake" > log.txt
cat log.txt
```



**Capturing a command's output into text with `$( )`**

Anything inside the dollar symbol and parenthesies `$( )` runs first, and its output replaces the `$( )`.

*(Run from `~/bash-training`)*
```bash
echo "This backup ran at $(date)"
```

We'll use this constantly to timestamp things.

*(Run from `~/bash-training`)*
```bash
echo "Earlier I made a $(cat log.txt)"
```

So the toolkit: `;`, `&&`, `||`, `|`, `>`, `>>`, `$( )` — plus `sort`, `head`, `wc -l` and `grep`. Everything we write today is your existing commands glued together with these.

<br>
<br>

### 09:45–10:15 — Hands-On: Chaining & Pipelines
*(Activity: 30 min)*

Your turn. Type these — don't paste — and **predict what each will do before you hit enter**.

**HANDS ON (30 min)** <br>

- `SLIDE ACROSS`

*(Run everything from `~/bash-training`)*

Part A — sequencing and safety.<br>
1. Run `mkdir demo && cd demo`, confirm with `pwd` → **you are now in `~/bash-training/demo`** <br>
2. From `~/bash-training`, run `mkdir demo && cd demo` **again**. What happens, and why did `cd` not run? Try the same with `;` and note the difference<br>
3. Create three empty files in one line: `touch a.txt b.txt c.txt`, confirm with `ls`<br>

Part B — pipes.<br>
4. `ls -la /etc | wc -l` — count the entries in `/etc`<br>
5. `ls /etc | sort | head -10`, then `ls /etc | sort -r | head -10`. What did `-r` change?<br>
6. `history | grep cd` — search your own command history<br>
7. `cat /etc/passwd | grep "$USER"` — find your own user's line in the system's user file<br>

Part C — redirection and substitution.<br>
8. `ls -la > my-listing.txt`, then `cat my-listing.txt`<br>
9. `echo "First line" > notes.txt`, then `echo "Second line" >> notes.txt`, then `cat notes.<br>txt`. Now run the **first** command again — what happened to "Second line", and why?<br>
10. `echo "Report generated on $(date) by $(whoami)" > report.txt` and read it back<br>

Part D — start your dictionary.<br>
11. Open `02-my-command-dictionary.md` from the student facing repo and fill in every row you've met so far, **in your own words**. Not copied from the notes — if you can't explain it simply, you haven't got it yet<br>

Part E (stretch).<br>
12. In one line, count how many files in `/usr/bin` have names containing "python"<br>
13. Build a pipeline that lists your files, sorts them by size, and shows only the largest few. (Hint: `ls -lS`)<br>
**END OF NOTE**

**💬 SLACK — snippet 1**, post at the start:
```bash
# Part A
mkdir demo && cd demo
pwd
mkdir demo && cd demo
mkdir demo ; cd demo
touch a.txt b.txt c.txt

# Part B
ls -la /etc | wc -l
ls /etc | sort | head -10
ls /etc | sort -r | head -10
history | grep cd
cat /etc/passwd | grep "$USER"

# Part C
ls -la > my-listing.txt
cat my-listing.txt
echo "First line" > notes.txt
echo "Second line" >> notes.txt
cat notes.txt
echo "Report generated on $(date) by $(whoami)" > report.txt
```

**Solution**

*(Run from `~/bash-training`)*
```bash
# --- Part A ---
mkdir demo && cd demo
pwd                          # -> /home/<you>/bash-training/demo

mkdir demo && cd demo        # cd does NOT run: mkdir failed, && stopped the chain
mkdir demo ; cd demo         # cd DOES run: ';' ignores the failure and carries on
cd ~/bash-training/demo      # get back to a known place

touch a.txt b.txt c.txt
ls                           # -> a.txt  b.txt  c.txt

# --- Part B ---
ls -la /etc | wc -l
ls /etc | sort | head -10
ls /etc | sort -r | head -10 # -r reverses the sort: Z to A
history | grep cd
cat /etc/passwd | grep "$USER"

# --- Part C ---
ls -la > my-listing.txt
cat my-listing.txt

echo "First line" > notes.txt
echo "Second line" >> notes.txt
cat notes.txt                # -> First line / Second line

echo "First line" > notes.txt   # '>' rebuilds the file from scratch...
cat notes.txt                   # -> First line only. "Second line" is gone.

echo "Report generated on $(date) by $(whoami)" > report.txt
cat report.txt

# --- Part E ---
ls /usr/bin | grep python | wc -l
ls -lS | head -5
```

**Part D — a good dictionary entry looks like this.** Show one on screen as a model, because "in your own words" needs demonstrating:

| Command | What it does (in MY words) | An example I actually ran |
|---|---|---|
| `wc -l` | Counts lines. Usually I pipe something into it rather than giving it a file, so it's really "how many results did that produce" | `ls /etc \| grep conf \| wc -l` |
| `>` vs `>>` | `>` replaces the whole file, `>>` adds to the bottom. Same as opening a file in write vs append mode in Node | `echo "hi" >> log.txt` |

**ASK** *(bring the room back)* <br>
In step 9, running the first `echo` again wiped "Second line". If that had been a log file you'd appended to all week, what one-character change would have saved you? <br>
**ANSWER** <br>
`>>` instead of `>`. One of the most common ways people accidentally destroy data in a shell — worth burning into muscle memory now.

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>
### 10:30–11:00 — Your First Script: Shebang, Permissions, Running It
*(Activity: 10 min)*

Everything so far, we've typed one line at a time. Fine for two or three commands. But real setup work is dozens of steps, run repeatedly, that need to behave *identically* every time. Typing them by hand invites typos, skipped steps, and "well, it worked on my machine".

A **script** is the fix. Just a plain text file containing the commands you'd otherwise type, run top to bottom. Because it's a file, you can save it, put it in Git, review it in a pull request, share it, and re-run it identically a thousand times.

**ASK** <br>
That list — saved, versioned, reviewed, re-run — should sound familiar. What have you been doing that with for years? <br>
**ANSWER** <br>
Application code. And that's the whole point: the moment a manual process becomes a script, **every good habit you already have for code applies to it**. Code review, version history, blame, rollback. This is "everything as code" from Session 1, arriving for the first time in something you can actually run.

**Create the file**

*(Run from `~/bash-training`)*
- Run: `touch hello.sh`
- Then open it: `code hello.sh` (or `nano hello.sh`)

```bash
#!/bin/bash
# hello.sh — my first script

echo "Hello, $USER!"
echo "Today is $(date)"
echo "You are currently in $(pwd)"
```

Two things to explain slowly, because they trip everyone up.

**The shebang.** That first line, `#!/bin/bash`, is a **shebang** (from "hash" `#` plus "bang" `!`). It tells the operating system **which program should interpret the rest of this file**. Here: "run this using the bash program, which lives at `/bin/bash`". For Python you'd write `#!/usr/bin/env python3`.

Without it, the system has to guess, and you can't reliably run the file as a standalone program. It **must** be the very first line — no blank lines, no spaces above it.

**Comments.** Any line starting with `#` (other than the shebang, special because of the `!`) is a **comment** — bash reads and ignores it. Notes for humans, explaining *why*.

**Try to run it**

*(Run from `~/bash-training`)*
```bash
./hello.sh
```

You'll get **Permission denied**. That's expected, and it's a good thing.

**ASK** <br>
Why does Linux refuse to run a text file you just created, by default? <br>
**ANSWER** <br>
Safety. If every text file were automatically executable, anything you downloaded — an email attachment, a file off the web — could run as a program the moment you touched it. Linux makes you **explicitly** mark a file as "yes, this is allowed to run".

**Make it executable**

`chmod` is short for "**ch**ange **mod**e" — it changes a file's permissions. `+x` means "**add** the e**x**ecute permission".

*(Run from `~/bash-training`)*
```bash
ls -l hello.sh        # look at the permissions BEFORE
chmod +x hello.sh
ls -l hello.sh        # and AFTER — spot the difference
./hello.sh
```

Do the before-and-after `ls -l` on screen. They'll see `-rw-r--r--` become `-rwxr-xr-x` — three new `x` characters. Worth decoding briefly:

```
-rwxr-xr-x
 |└┬┘└┬┘└┬┘
 | |  |  └── everyone else:  r-x  (read, execute)
 | |  └───── the group:      r-x  (read, execute)
 | └──────── the owner:      rwx  (read, write, execute)
 └────────── file type: '-' is a normal file, 'd' is a directory
```

We'll meet the numeric form (`chmod 400`, `chmod 755`) in a later session when we handle SSH keys.

**Why the `./`?** When you type `hello.sh` on its own, bash searches a specific list of folders for a command by that name — and your **current folder is not on that list**. The `./` means "the file is right **here**" (`.` is shorthand for the current directory — the same `.` you've typed in `docker build .`).

That "specific list of folders" is what we're doing next, and it's worth understanding properly.

**HANDS ON (10 min)** <br>
*(Run from `~/bash-training`)*
1. Create `hello.sh` as above, make it executable, and run it
2. Run `ls -l hello.sh` before and after the `chmod` and note exactly what changed
3. Add two more lines: one printing how many files are in the current directory, one printing your machine's hostname
4. Remove the execute permission with `chmod -x hello.sh`, try to run it to see the error, then restore it
**END OF NOTE**

**💬 SLACK — snippet 2**:
```bash
#!/bin/bash
# hello.sh — my first script

echo "Hello, $USER!"
echo "Today is $(date)"
echo "You are currently in $(pwd)"
echo "There are $(ls | wc -l) files here"
echo "Running on $(hostname)"
```

**Solution**

`hello.sh`:
```bash
#!/bin/bash
# hello.sh — my first script

echo "Hello, $USER!"
echo "Today is $(date)"
echo "You are currently in $(pwd)"
echo "There are $(ls | wc -l) files here"
echo "Running on $(hostname)"
```

*(Run from `~/bash-training`)*
```bash
ls -l hello.sh       # -rw-r--r--   no 'x' anywhere
chmod +x hello.sh
ls -l hello.sh       # -rwxr-xr-x   three 'x's appeared
./hello.sh

chmod -x hello.sh
./hello.sh           # -> bash: ./hello.sh: Permission denied
chmod +x hello.sh
```

<br>
<br>

### 11:00–11:40 — The Filesystem, PATH, and Naming a Script
*(Activity: 20 min)*

Our script only runs if we're in the same folder and prefix it with `./`. That's not how real commands work — you type `git` or `docker` from *anywhere*. Fixing that means understanding where programs actually live on a Linux machine, which is worth knowing in its own right.

#### What is a "binary"?

You already know `ls` is a program on disk. Let's look at it.

*(Run from `~/bash-training`)*
```bash
which ls
file $(which ls)
```

`which` tells you the path. `file` tells you what kind of thing it is — for `ls` you'll see something like *"ELF 64-bit LSB executable"* (Linux) or *"Mach-O 64-bit executable"* (Mac).

A **binary** is a **compiled executable** — machine code, not human-readable text. Somebody wrote `ls` in C, compiled it, and the result is that file.

Compare with your own script:

*(Run from `~/bash-training`)*
```bash
file hello.sh
```

That says *"Bourne-Again shell script, ASCII text executable"* — plain text, and the shebang tells the system what to run it with.

**ASK** <br>
So both `ls` and `hello.sh` are executable files. What's the actual difference between them? <br>
**ANSWER** <br>
`ls` is **compiled machine code** — the CPU runs it directly. `hello.sh` is **plain text** that needs an interpreter, which the shebang names. Practically it means a binary is faster and self-contained, while a script is readable, editable and portable. And crucially, **the shell treats both identically** — a command is a command. That's why once we put our script in the right place, it becomes as much a "real command" as `ls` is.

That parallel is worth naming: `node app.js` versus a compiled Go binary is the same distinction, one layer up.

#### Where programs live

Have a look at the top level of the filesystem:

*(Run from `~/bash-training`)*
```bash
ls /
```

The Linux filesystem is standardised — the **Filesystem Hierarchy Standard** — so these folders mean the same thing on almost any Linux machine you ever log into. The ones that matter:

| Folder | Purpose |
|---|---|
| **`/bin`** | **Essential** commands needed for the system to function at all — `ls`, `cp`, `mv`, `cat`, `bash`. Deliberately minimal, so the system can boot and be repaired even if other disks aren't mounted |
| **`/usr/bin`** | The **bulk of normal user commands** — `git`, `grep`, `python`, `docker`. Most of what you type lives here. Installed by the system's package manager |
| **`/usr/local/bin`** | Software **installed manually by an administrator**, outside the package manager. The convention is: package manager writes to `/usr/bin`, *you* write to `/usr/local/bin`, so an OS upgrade never clobbers your stuff |
| **`/sbin`, `/usr/sbin`** | **System** binaries, normally root-only — `reboot`, `fdisk`, `iptables`. The `s` is for "system" (or "superuser") |
| **`~/bin`** or `~/.local/bin` | **Your personal** scripts. Not shared with other users. This is where ours goes today |
| **`/opt`** | Large **self-contained third-party** packages that ship as one directory rather than scattering files around |
| **`/etc`** | **Configuration** files. Text, system-wide, editable. (`/etc/passwd`, which you grepped earlier, lives here) |
| **`/var`** | **Variable** data that changes as the system runs — logs in `/var/log`, caches, web content in `/var/www` |
| **`/tmp`** | **Temporary** files. Wiped on reboot. Anything can write here |
| **`/home`** | User home directories. Your `~` is `/home/yourname` (or `/Users/yourname` on Mac) |

**NOTE FOR TRAINERS** <br>
Two things will contradict this on students' machines, so pre-empt them. **(1) macOS**: Homebrew installs to `/opt/homebrew/bin` on Apple Silicon and `/usr/local/bin` on Intel, so `which git` gives different answers around the room — that's correct, not broken. **(2) Modern Linux**: many distributions now make `/bin` a *symlink* to `/usr/bin`, so the historical split is preserved in name only. Neither undermines the point; the *convention* is what matters, and every one of these paths still means what it says. <br>
**END OF NOTE**

**ASK** <br>
Two of those folders will matter enormously later in this course. Which, and why? <br>
**ANSWER** <br>
**`/etc`** and **`/var`**. When we provision a web server with Terraform in a few sessions, we write our HTML to `/var/www/html` — because `/var` is where variable, changing data lives, and `www` is the web server's content. And configuration we change lives in `/etc`. Knowing the convention means those paths will look obvious rather than arbitrary when they appear.

#### The PATH

When you type a bare command name, bash searches a list of folders held in a variable called `PATH`.

*(Run from `~/bash-training`)*
```bash
echo $PATH
```

Something like `/usr/local/bin:/usr/bin:/bin` — folders separated by **colons**. When you type `git`, bash walks that list in order and runs the **first** `git` it finds.

Prove the "in order" part:

*(Run from `~/bash-training`)*
```bash
which -a python3      # ALL the matches, in PATH order
type ls               # what IS this — binary, alias, or function?
```

To make *our* script callable by name, we put it in a folder on that list — or add a new folder to it.

The tidy convention is a personal `bin` in your home directory:

*(Run from `~/bash-training`)*
```bash
mkdir -p ~/bin
mv hello.sh ~/bin/hello
export PATH="$HOME/bin:$PATH"
```

Line by line:
- `mkdir -p ~/bin` — `-p` means "make it, don't complain if it exists"
- `mv hello.sh ~/bin/hello` — move it in, **dropping the `.sh`**
- `export PATH="$HOME/bin:$PATH"` — "set PATH to my new folder, then a colon, then everything PATH already was"

Putting it at the **front** means bash checks `~/bin` first.

Now prove it from somewhere completely different:

*(Run from `~/` — get there with `cd ~`)*
```bash
hello
which hello
```

No `./`, no `.sh`. It behaves exactly like a built-in command — and `which` confirms bash found it the same way it finds `git`.

**ASK** <br>
We dropped the `.sh` when we moved it. Why doesn't that break anything? <br>
**ANSWER** <br>
The extension is just part of the name; it means **nothing** to Linux. What determines how the file runs is the **shebang** on line one. That's why real commands — `git`, `docker`, `ls` — have no extension at all. Windows works the opposite way, where `.exe` genuinely matters, which is where the confusion comes from.

**NOTE FOR TRAINERS** <br>
`export PATH=...` only lasts for **the current terminal session**. Demonstrate this: open a brand new terminal, type `hello`, watch it fail. Then add the line to `~/.bashrc` (or `~/.zshrc` on macOS), run `source ~/.bashrc`, and show it surviving. That file runs every time a shell opens — it's exactly how `nvm`, the Azure CLI and Homebrew hook themselves in, and students have almost certainly had tools ask them to add a line to it without knowing why. <br>
**END OF NOTE**

#### Passing input in: positional arguments

A script doing the same thing every time is useful. A script you can *tell what to do* is far more useful. When you run `mkdir project` or `docker build myimage`, those extra words are **arguments**.

*(Run from `~/bash-training`)*
- Run: `touch greet`
- Then: `code greet`

```bash
#!/bin/bash
# greet — say hello to whoever we're told to

echo "First argument:  $1"
echo "Second argument: $2"
echo "All arguments:   $@"
echo "Number of args:  $#"
```

| Variable | Means |
|---|---|
| `$1` | the **first** argument |
| `$2` | the **second** (and `$3`, `$4`...) |
| `$@` | **all** the arguments |
| `$#` | **how many** arguments |
| `$0` | the **script's own name** |

*(Run from `~/bash-training`)*
```bash
chmod +x greet
./greet Alice Bob
```

`$1` is `Alice`, `$2` is `Bob`, `$#` is `2`. This is exactly the mechanism behind `scaffold countries` after the break.

**Asking interactively with `read`**

```bash
#!/bin/bash
read -p "What should we call the resource? " resource
echo "Great — we'll scaffold '$resource'"
```

`read -p` prints the prompt, waits for input, stores it in `resource`. Use it afterwards as `$resource`.

**HANDS ON (20 min)** <br>

Part A *(10 min)* — the filesystem, hands on.
1. *(Run from `~/bash-training`)* Run `ls /` and open two folders you've never looked in. What's in `/var/log`?
2. Run `which ls`, `which git`, `which docker`. Which folders did each come from? Compare with the person next to you — are they the same?
3. Run `file $(which ls)` and `file hello.sh`. Explain the difference to your neighbour
4. Run `echo $PATH` and count how many folders are on it

Part B *(10 min)* — naming your script.
5. Create `~/bin`, move `hello.sh` in as `hello`, and add `~/bin` to your PATH
6. Prove it runs from `~/` and from `/tmp`
7. Create the `greet` script, install it to `~/bin/greet`, and run `greet DevOps Engineer` from a different folder
8. Modify `greet` to print `Hello <first-arg>, nice to meet you`
9. Write a tiny `ask` script using `read` that asks for a name and echoes it back
10. Add today's new commands — `which`, `type`, `file`, `chmod` — to your dictionary
**END OF NOTE**

**💬 SLACK — snippet 3**:
```bash
# Investigating where things live
ls /
which ls
which git
file $(which ls)
file hello.sh
echo $PATH
which -a python3
type ls

# Making your script a real command
mkdir -p ~/bin
mv hello.sh ~/bin/hello
export PATH="$HOME/bin:$PATH"

# Make it permanent (Mac users: use ~/.zshrc)
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

**Solution**

Part A — expected answers:

```bash
ls /                    # bin, etc, home, opt, tmp, usr, var, ...
ls /var/log             # system + application logs. syslog, auth.log, dpkg.log...
which ls                # /bin/ls   (or /usr/bin/ls on newer distros)
which git               # /usr/bin/git   (Mac+Homebrew: /opt/homebrew/bin/git)
which docker            # /usr/local/bin/docker  or  /usr/bin/docker
file $(which ls)        # ELF 64-bit LSB executable  -> a COMPILED BINARY
file hello.sh           # Bourne-Again shell script  -> PLAIN TEXT
echo $PATH              # typically 4-8 folders, colon-separated
```

**Step 2 is the interesting one** — answers will differ around the room, and that's the teaching point. A Mac user with Homebrew gets `/opt/homebrew/bin/git`; a WSL user gets `/usr/bin/git`. Same command, different location, because different installers use different conventions. That's precisely why `PATH` exists rather than hardcoded paths.

Part B:

`greet` (after step 8):
```bash
#!/bin/bash
# greet — say hello to whoever we're told to

echo "Hello $1, nice to meet you"
```

`ask`:
```bash
#!/bin/bash
# ask — prompt for a name and echo it back

read -p "What's your name? " name
echo "Nice to meet you, $name"
```

*(Run from `~/bash-training`)*
```bash
chmod +x greet ask
mv greet ~/bin/greet
mv ask   ~/bin/ask
```

*(Run from `~/` — prove it works anywhere)*
```bash
greet DevOps Engineer     # -> Hello DevOps, nice to meet you
                          #    ($1 is "DevOps", $2 is "Engineer" — note the space split them)
ask
```

*(Run from `/tmp` — prove it again from somewhere unrelated)*
```bash
cd /tmp && hello
```

**Worth pointing out from step 7:** `greet DevOps Engineer` prints `Hello DevOps` — because the space made those **two** arguments. To pass one, quote it: `greet "DevOps Engineer"`. That's a preview of why quoting matters, and it'll bite them in the capstone if they don't internalise it.

<br>
<br>
### 11:40–12:00 — Logic: Conditionals, Loops and Functions
*(Activity: 10 min + challenge)*

We have named scripts that take input. The last piece before we build something real is **logic** — making a script *decide* and *repeat*.

You know all three concepts from JavaScript. **The concepts transfer; the syntax does not.** So I'll go slowly on the syntax and lean on what you already know for the meaning.

**Conditionals — `if`**

```bash
if [ CONDITION ]; then
  # commands when TRUE
else
  # commands when FALSE
fi
```

Naming every part:
- `if` — starts it
- `[ CONDITION ]` — the thing being tested
- `; then` — "...**then** do the following". `then` must be separated from the condition, hence the `;` (or put `then` on its own line)
- `else` — optional
- `fi` — ends it. Literally "if" backwards. Bash does this again with `case`/`esac`

**The square brackets are actually a command.** This is the bit nobody tells beginners, and it explains the most common error you'll hit today. `[` is **not punctuation** — it's a command (an alias for `test`), and `]` is its final argument.

That's **why the spaces matter**:
- `[ -z "$1" ]` ✅ works
- `[-z "$1"]` ❌ fails — bash can't see `[` as a separate command without the space

**ASK** <br>
In JavaScript, `if(x)` and `if (x)` are identical. Why is bash different? <br>
**ANSWER** <br>
Because in JavaScript `if` is **language syntax** — the parser understands it structurally. In bash, `[` is a **program being run**, and bash splits a command line into words by spaces. `[-z` looks like one word, so bash goes hunting for a command called `[-z` and fails. Once you see `[` as a command rather than a bracket, the spacing rule stops being arbitrary. If a student's `if` is mysteriously broken, this is the cause 90% of the time.

The test operators you'll actually use:

| Test | True when |
|---|---|
| `-z "$var"` | the variable is **empty** (`z` = zero length) |
| `-n "$var"` | the variable is **not empty** |
| `-f path` | a **file** exists there |
| `-d path` | a **directory** exists there |
| `"$a" = "$b"` | two **strings** are equal |
| `"$a" != "$b"` | two **strings** differ |
| `$a -eq $b` | two **numbers** are equal |
| `$a -ne $b` | numbers **not** equal |
| `$a -lt $b` | **less than** |
| `$a -gt $b` | **greater than** |

Strings and numbers use **different operators**: `=` for text, `-eq` for numbers. That catches everyone once. And `!` means "not" — `[ ! -f "Dockerfile" ]` reads "if a Dockerfile does **not** exist".

The single most common real use is a **guard against bad input** at the top of a script:

```bash
#!/bin/bash
if [ -z "$1" ]; then
  echo "Usage: scaffold <resource-name>" >&2
  exit 1
fi
echo "Scaffolding $1..."
```

Two new pieces:

- **`>&2`** — send this to **standard error** rather than normal output. Every program has two output channels: `1` is stdout (results), `2` is stderr (complaints). `>&2` redirects to channel 2. Error messages belong there so automated tools can tell your *results* apart from your *complaints*
- **`exit 1`** — stop immediately and report **failure**

Another everyday `if` — don't clobber something existing:

```bash
if [ -d "server" ]; then
  echo "server/ already exists, skipping"
else
  mkdir server
fi
```

**Exit codes — how a script says "I succeeded" or "I failed"**

Every command hands back a number when it finishes: an **exit code**. `0` means success; **any other number means failure**. Backwards from what you'd expect — think of it as "zero problems".

*(Run from `~/bash-training`)*
```bash
ls /nonexistent-folder
echo $?     # -> non-zero (e.g. 2). That command failed.

ls ~
echo $?     # -> 0. Success.
```

When *your* script calls `exit 1`, you're telling whatever ran it "I failed".

**NOTE FOR TRAINERS** <br>
This is **the single most important concept to land for later sessions**. In the Jenkins session, a pipeline decides whether a build passes or fails purely by reading the exit code of the scripts it runs. A script that returns `0` when something went wrong makes a broken build look green and ships broken software. "Fail loudly with a non-zero exit code" is a professional habit, not a nicety. Say so explicitly, and say it more than once. <br>
**END OF NOTE**

**Loops — `for`**

```bash
for VARIABLE in LIST; do
  # commands, usually using "$VARIABLE"
done
```

- `for dir` — invent a variable (you pick the name — **no `$` when declaring**)
- `in controllers models routers db` — the list, separated by **spaces**
- `; do` — "for each of those, **do** this"
- `done` — ends the loop (as `fi` ended the `if`)

So:

```bash
for dir in controllers models routers db; do
  mkdir -p "server/$dir"
done
```

Runs `mkdir -p "server/controllers"`, then `models`, then `routers`, then `db`. One line of intent instead of four `mkdir` lines. (Quote `"server/$dir"` as a habit, so it survives names containing spaces.)

Over files with a wildcard:
```bash
for file in *.js; do
  echo "Found JavaScript file: $file"
done
```

Over a number range:
```bash
for i in {1..5}; do
  echo "Number $i"
done
```

**ASK** <br>
Compare that to `array.forEach(item => ...)`. What's genuinely different? <br>
**ANSWER** <br>
Structurally almost nothing — invent a variable, iterate a collection, run a block. The real differences are that **bash has no arrays of objects**, only lists of strings, and that the "list" is often generated by the shell itself: `*.js` expands to matching filenames *before* the loop even starts, and `{1..5}` expands to `1 2 3 4 5`. So bash loops feel less like iteration and more like "here is a list of words, do this to each one."

**Functions — naming a block of steps**

```bash
name_of_function() {
  # commands
}
```

- `log()` — the name you invent, followed by **empty** `()`
- `{ ... }` — the block

A genuinely useful one — a logger stamping every message with the time:

```bash
log() {
  echo "[$(date +%H:%M:%S)] $1"
}

log "Starting up"
log "Creating folders"
```

Two things to notice. `date +%H:%M:%S` formats as hours:minutes:seconds. And — this catches people — **inside a function, `$1` is the *function's* first argument, not the script's**. So `log "Starting up"` passes that string as the function's `$1`.

**ASK** <br>
Bash functions have no parameter list — you don't write `log(message)`. How do you know what a function expects? <br>
**ANSWER** <br>
**You don't, unless someone wrote a comment.** Bash functions just receive `$1`, `$2` and so on, with no declaration, no arity check and no error if you pass the wrong number. It's genuinely worse than JavaScript here. Which is why a one-line comment above every function saying what it takes isn't optional politeness — it's the only documentation that will ever exist.

**HANDS ON (10 min)** <br>
*(Run from `~/bash-training`)*
1. Write a script `check` taking one argument, printing "That file exists" if a file by that name exists (`-f`), "No such file" otherwise. Test both cases
2. Add a guard at the top: no argument → usage message to `>&2` and `exit 1`. Confirm `echo $?` shows `1`
3. Write a `for` loop creating five files named `test1.txt` through `test5.txt`
4. Add `if`, `for`, functions, `$?` and `>&2` to your dictionary
**END OF NOTE**

**💬 SLACK — snippet 4**:
```bash
# The three shapes. Note WHERE THE SPACES ARE in the if.

if [ -z "$1" ]; then
  echo "Usage: ..." >&2
  exit 1
fi

for dir in controllers models routers db; do
  mkdir -p "server/$dir"
done

log() {
  echo "[$(date +%H:%M:%S)] $1"
}
```

**Solution**

`check`:
```bash
#!/bin/bash
# check — report whether a named file exists
# Takes: $1 = filename to check

# Guard: stop early if no filename was given
if [ -z "$1" ]; then
  echo "Usage: check <filename>" >&2
  exit 1
fi

if [ -f "$1" ]; then
  echo "That file exists"
else
  echo "No such file"
fi
```

*(Run from `~/bash-training`)*
```bash
chmod +x check
touch real.txt

./check real.txt        # -> That file exists
./check nope.txt        # -> No such file

./check                 # -> usage message (to stderr)
echo $?                 # -> 1, proving it failed on purpose

# Step 3
for i in 1 2 3 4 5; do
  touch "test$i.txt"
done
ls test*.txt            # -> test1.txt ... test5.txt
```

Worth demonstrating the stderr split, since it's abstract otherwise:
```bash
./check > /dev/null     # throw away NORMAL output — the error still appears
./check 2> /dev/null    # throw away ERROR output — nothing appears
```
That's why `>&2` matters: the two streams can be handled separately.

---

**Challenge**

*Direct* students, **in pairs**, to write a bash function that:
- Accepts three arguments: `start`, `stop`, `final`
- Counts down from `start` to `stop`, printing each number
- Instead of printing `stop`, prints `final`
- **OPTIONAL** — handle `stop` being greater than `start`

*Provide* this example output for `countdown 10 5 "Blastoff!"`:

```
10
9
8
7
6
Blastoff!
```

*Grant* ~8 minutes. Hints if stuck: a countdown needs a loop that goes *down*, and `seq` generates descending lists — `seq 10 -1 6` prints 10 down to 6.

**SOLUTION**

```bash
#!/bin/bash

# countdown — print numbers from start down to stop, then a final word
# Takes: $1 = start, $2 = stop, $3 = final message
countdown() {
  local start="$1"
  local stop="$2"
  local final="$3"

  # seq START STEP END — a step of -1 counts downwards.
  # We stop at (stop + 1) because 'stop' itself is replaced by 'final'.
  for i in $(seq "$start" -1 $((stop + 1))); do
    echo "$i"
  done

  echo "$final"
}

countdown 10 5 "Blastoff!"
```

Three things worth calling out:
- **`local start="$1"`** — `local` keeps the variable **inside** the function so it can't clash with anything outside. Bash variables are global by default, which is the opposite of what you'd expect from JavaScript's `let`. Good habit
- **`$(( stop + 1 ))`** — **arithmetic expansion**. Bash won't do maths unless told to; without the double brackets, `stop + 1` is just text
- **`seq "$start" -1 $((stop + 1))`** — generates the descending list the `for` loop walks

**SOLUTION (optional extension — handles counting up too)**

```bash
#!/bin/bash

countdown() {
  local start="$1"
  local stop="$2"
  local final="$3"

  if [ "$start" -gt "$stop" ]; then
    for i in $(seq "$start" -1 $((stop + 1))); do   # counting DOWN
      echo "$i"
    done
  else
    for i in $(seq "$start" 1 $((stop - 1))); do    # counting UP
      echo "$i"
    done
  fi

  echo "$final"
}

countdown 10 5 "Blastoff!"
countdown 1 6 "Liftoff!"
```

---

**Writing whole files from a script — the heredoc**

One last technique, needed to build our app after the break. Sometimes a script must **write out the entire contents of a file**, not just create an empty one. That's a **heredoc** ("here document"):

```bash
cat > server/index.js << 'EOF'
const app = require("./app");
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Listening on port ${PORT}`));
EOF
```

Piece by piece:
- `cat` — outputs whatever it's given
- `> server/index.js` — redirect that output into this file
- `<< 'EOF'` — "here comes a block of text; keep reading until a line that is just `EOF`"
- everything between the markers becomes the file contents
- the closing `EOF`, alone on its line, ends it

`EOF` is just a chosen marker word — any word works, as long as both match.

**The quoting on the marker is important and completely new to most people:**

| Written as | Behaviour | Use when |
|---|---|---|
| `<< 'EOF'` (single quotes) | Writes text **literally**, leaving `$` exactly as typed | The file contains its **own** `$` variables — like the JavaScript above, whose `${PORT}` belongs to **Node**, not bash |
| `<< EOF` (no quotes) | Bash **substitutes** its own variables first | You *want* your script's variables (like `$resource`) filled in |

**ASK** <br>
That JavaScript uses `${PORT}` and a template literal with backticks. If we used an **unquoted** heredoc, what would bash do to it? <br>
**ANSWER** <br>
Bash would try to substitute `${PORT}` with its *own* variable called `PORT` — which doesn't exist — and write an **empty string** into your JavaScript. You'd get `console.log(\`Listening on port \`)` and a confusing bug that looks like a Node problem. That's why the quoting matters, and it's the single most likely thing to go wrong in the next session.

We use both after the break, and I'll flag which is which every time.

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>
### 12:15–13:00 — Building an App Scaffold: the `countries` Script
*(Activity: 45 min)*

Now the whole morning comes together into something genuinely useful.

Every time you start a new Express API you make the same folders and the same starter files by hand. That's *exactly* the repetitive, error-prone, "did I forget one?" task we automate. You know Express and `package.json` well — the *contents* will be completely familiar. **The new part is having a script write them for you.**

We're writing a script called `scaffold` that, given a resource name like `countries`, builds this in one command:

```
.
├── package.json
├── README.md
└── server
    ├── app.js
    ├── controllers
    │   └── countries.js
    ├── db
    │   ├── connect.js
    │   ├── countries.sql
    │   └── setup.js
    ├── index.js
    ├── models
    │   └── Country.js
    └── routers
        └── countries.js
```

Notice the naming: controller, router and SQL file are **plural lowercase** (`countries`), but the model is **singular capitalised** (`Country`). We handle that with arguments.

**NOTE FOR TRAINERS — how to run this session** <br>
This is the one place today where **paste, don't type** is the right instruction. Stages 2 to 5 are long heredoc blocks whose *contents* students already know cold — transcribing forty lines of Express boilerplate teaches nothing and costs twenty minutes. <br>
Post each stage to Slack as you reach it, and spend the reclaimed time on the **three things that are actually new**: `set -e`, the `${2:-${resource^}}` default, and the quoted-versus-unquoted heredoc distinction. Have students read each block and tell *you* which heredocs are quoted and why. <br>
**END OF NOTE**

Create the file first:

*(Run from `~/bash-training`)*
- Run: `touch scaffold`
- Then: `code scaffold`

**Stage 1 — the skeleton, guards, and folders**

```bash
#!/bin/bash
# scaffold — generate an Express MVC application skeleton
# Usage: scaffold <resource> [ModelName]
#   e.g. scaffold countries Country

set -e   # stop immediately if any command fails

resource="$1"
model="${2:-${resource^}}"   # 2nd arg, or capitalised resource as a default

# Guard: we must have a resource name
if [ -z "$resource" ]; then
  echo "Usage: scaffold <resource> [ModelName]" >&2
  echo "  e.g. scaffold countries Country" >&2
  exit 1
fi

log() { echo "[scaffold] $1"; }

log "Creating app for resource '$resource' (model '$model')"

# The four sub-folders under server/, made in one loop
for dir in controllers db models routers; do
  mkdir -p "server/$dir"
done
```

Three things to unpack, because they're new:

**`set -e`** — "if **any** command in this script fails, stop the whole script right there." Without it a script charges on after an error, potentially making a bigger mess. With it, you don't need to check the exit code after every line by hand.

**`resource="$1"`** — copies the first argument into a nicely-named variable so the rest reads clearly. Note: **no spaces around the `=`**. `resource = "$1"` is an error — bash would look for a command called `resource`.

**`model="${2:-${resource^}}"`** — two clever things at once:
- `${2:-something}` means "use `$2`, **but if it's missing or empty, use `something`**". The `:-` is the default-value operator
- `${resource^}` **capitalises the first letter** — that's what `^` does
- Together: if a model name was given, use it; otherwise capitalise the resource. So `scaffold countries` → model `Countries`, while `scaffold countries Country` → exactly `Country`

**ASK** <br>
`${2:-default}` should feel familiar. What's the JavaScript equivalent? <br>
**ANSWER** <br>
A default parameter — `function scaffold(resource, model = capitalise(resource))` — or the `??` nullish-coalescing operator. Same idea entirely: use what was passed, fall back if not. Bash just writes it more cryptically.

**ASK** <br>
Why put the `if [ -z "$resource" ]` guard at the very top, before creating a single folder? <br>
**ANSWER** <br>
If someone runs `scaffold` with no argument, `$resource` is empty, and every path becomes malformed — `server/models/.js`, files named after nothing. Failing fast with a clear usage message is far kinder than half-creating a broken mess and leaving the user to clean it up. It's the same instinct as validating request params at the top of an Express route handler rather than letting `undefined` propagate into your database query.

**Stage 2 — the top-level files**

Heredocs, **unquoted** (`<< EOF`), because we *want* `$resource` substituted:

```bash
cat > package.json << EOF
{
  "name": "${resource}-api",
  "version": "1.0.0",
  "main": "server/index.js",
  "scripts": {
    "start": "node server/index.js",
    "db:setup": "node server/db/setup.js"
  },
  "dependencies": {
    "express": "^4.19.2",
    "pg": "^8.11.5"
  }
}
EOF

cat > README.md << EOF
# ${resource} API

A small Express MVC API for ${resource}, generated by the scaffold script.

## Getting started
\`\`\`
npm install
npm run db:setup
npm start
\`\`\`
EOF

log "Created package.json and README.md"
```

*(Those escaped backticks in the README heredoc — the backslashes stop bash treating them as a command to run. They come out as plain triple-backticks in the finished file.)*

**Stage 3 — the server entry point and app wiring**

**This is where the heredoc quoting difference matters** — point it out explicitly:

- `index.js` has **no bash variables**, but *does* contain `${PORT}`, which belongs to **Node**. So we **quote** the marker (`<< 'EOF'`) to keep everything literal
- `app.js` **does** need `$resource` substituted, so it stays **unquoted** (`<< EOF`)

```bash
cat > server/index.js << 'EOF'
const app = require("./app");

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Listening on port ${PORT}`));
EOF

cat > server/app.js << EOF
const express = require("express");
const ${resource}Router = require("./routers/${resource}");

const app = express();
app.use(express.json());

app.use("/${resource}", ${resource}Router);

module.exports = app;
EOF

log "Created server/index.js and server/app.js"
```

**Stage 4 — controller, router, model**

All three **unquoted**, because each needs `$resource` and/or `$model` filled in:

```bash
cat > server/controllers/${resource}.js << EOF
const ${model} = require("../models/${model}");

async function index(req, res) {
  const rows = await ${model}.findAll();
  res.json(rows);
}

module.exports = { index };
EOF

cat > server/routers/${resource}.js << EOF
const express = require("express");
const controller = require("../controllers/${resource}");

const router = express.Router();
router.get("/", controller.index);

module.exports = router;
EOF

cat > server/models/${model}.js << EOF
const db = require("../db/connect");

class ${model} {
  static async findAll() {
    const result = await db.query("SELECT * FROM ${resource}");
    return result.rows;
  }
}

module.exports = ${model};
EOF

log "Created controller, router and model"
```

**Stage 5 — the database files**

```bash
cat > server/db/connect.js << 'EOF'
const { Pool } = require("pg");

const db = new Pool({
  connectionString: process.env.DATABASE_URL,
});

module.exports = db;
EOF

cat > server/db/${resource}.sql << EOF
DROP TABLE IF EXISTS ${resource};

CREATE TABLE ${resource} (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL
);

INSERT INTO ${resource} (name) VALUES ('Example one'), ('Example two');
EOF

cat > server/db/setup.js << EOF
const fs = require("fs");
const path = require("path");
const db = require("./connect");

async function setup() {
  const sql = fs.readFileSync(path.join(__dirname, "${resource}.sql"), "utf-8");
  await db.query(sql);
  console.log("Database set up for ${resource}");
  await db.end();
}

setup();
EOF

log "Created db/connect.js, db/${resource}.sql and db/setup.js"
log "Done. Run 'tree' to see your new structure."
```

**ASK** *(before running it — good comprehension check)* <br>
Look at Stage 5. `connect.js` is quoted, `setup.js` is not. Why the difference? <br>
**ANSWER** <br>
`connect.js` contains no bash variables at all — nothing to substitute, so quote it and keep it literal. `setup.js` contains `"${resource}.sql"`, which we genuinely want bash to fill in with the resource name. **The rule: quote unless you need substitution.** Quoting is the safer default, because an accidental substitution silently corrupts your file rather than erroring.

**Now use it**

Install the script so it's callable by name:

*(Run from `~/bash-training`)*
```bash
chmod +x scaffold
mv scaffold ~/bin/scaffold
```

Then make a fresh app folder and run it there:

*(Run from `~/`)*
- Run: `mkdir countries-app` → **cd inside** with `cd countries-app`

*(Run from `~/countries-app`)*
```bash
scaffold countries Country
tree            # or: ls -R  if you don't have tree
```

If everything's right, `tree` shows exactly the structure from the top of this section. **You've just replaced ten minutes of typing with one command** — identical every time, for everyone on the team.

**HANDS ON (45 min)** <br>
Build the `scaffold` script alongside me, stage by stage. Paste stages 2–5 from Slack — but **read each one and be ready to say which heredocs are quoted and why**.

1. *(Run from `~/countries-app`)* Run `scaffold countries Country`, confirm the tree matches exactly
2. *(Run from `~/`)* `mkdir books-app` → **cd inside**, then *(from `~/books-app`)* run `scaffold books Book`. Watch the same structure appear with different names — **this is the payoff of arguments over hard-coding**
3. *(Run from `~/books-app`)* `cat server/models/Book.js` and confirm the substitutions worked
4. *(Run from `~/books-app`)* `cat server/index.js` and confirm `${PORT}` survived **literally** — proving the quoted heredoc did its job
5. **(Stretch)** Add a guard near the top: if a `server/` folder already exists here, warn and `exit 1` rather than overwriting
6. **(Stretch)** Add a final line running `git init` so every scaffolded app starts as a Git repo
7. **(Stretch)** Run `scaffold` with **no arguments** and confirm the guard fires and `echo $?` gives `1`
**END OF NOTE**

**💬 SLACK — snippet 5.** Post Stages 2, 3, 4 and 5 as **four separate messages**, as you reach each one. Posting all four at once means everyone races ahead and nobody reads them.

**Solution**

The complete assembled `scaffold`, including both stretch goals:

```bash
#!/bin/bash
# scaffold — generate an Express MVC application skeleton
# Usage: scaffold <resource> [ModelName]
#   e.g. scaffold countries Country

set -e

resource="$1"
model="${2:-${resource^}}"

# Guard 1: we must have a resource name
if [ -z "$resource" ]; then
  echo "Usage: scaffold <resource> [ModelName]" >&2
  echo "  e.g. scaffold countries Country" >&2
  exit 1
fi

# Guard 2 (stretch): refuse to overwrite an existing app
if [ -d "server" ]; then
  echo "A server/ directory already exists here — aborting so nothing is overwritten." >&2
  exit 1
fi

log() { echo "[scaffold] $1"; }

log "Creating app for resource '$resource' (model '$model')"

for dir in controllers db models routers; do
  mkdir -p "server/$dir"
done

cat > package.json << EOF
{
  "name": "${resource}-api",
  "version": "1.0.0",
  "main": "server/index.js",
  "scripts": {
    "start": "node server/index.js",
    "db:setup": "node server/db/setup.js"
  },
  "dependencies": {
    "express": "^4.19.2",
    "pg": "^8.11.5"
  }
}
EOF

cat > README.md << EOF
# ${resource} API

A small Express MVC API for ${resource}, generated by the scaffold script.

## Getting started
\`\`\`
npm install
npm run db:setup
npm start
\`\`\`
EOF

log "Created package.json and README.md"

cat > server/index.js << 'EOF'
const app = require("./app");

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Listening on port ${PORT}`));
EOF

cat > server/app.js << EOF
const express = require("express");
const ${resource}Router = require("./routers/${resource}");

const app = express();
app.use(express.json());

app.use("/${resource}", ${resource}Router);

module.exports = app;
EOF

log "Created server/index.js and server/app.js"

cat > server/controllers/${resource}.js << EOF
const ${model} = require("../models/${model}");

async function index(req, res) {
  const rows = await ${model}.findAll();
  res.json(rows);
}

module.exports = { index };
EOF

cat > server/routers/${resource}.js << EOF
const express = require("express");
const controller = require("../controllers/${resource}");

const router = express.Router();
router.get("/", controller.index);

module.exports = router;
EOF

cat > server/models/${model}.js << EOF
const db = require("../db/connect");

class ${model} {
  static async findAll() {
    const result = await db.query("SELECT * FROM ${resource}");
    return result.rows;
  }
}

module.exports = ${model};
EOF

log "Created controller, router and model"

cat > server/db/connect.js << 'EOF'
const { Pool } = require("pg");

const db = new Pool({
  connectionString: process.env.DATABASE_URL,
});

module.exports = db;
EOF

cat > server/db/${resource}.sql << EOF
DROP TABLE IF EXISTS ${resource};

CREATE TABLE ${resource} (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL
);

INSERT INTO ${resource} (name) VALUES ('Example one'), ('Example two');
EOF

cat > server/db/setup.js << EOF
const fs = require("fs");
const path = require("path");
const db = require("./connect");

async function setup() {
  const sql = fs.readFileSync(path.join(__dirname, "${resource}.sql"), "utf-8");
  await db.query(sql);
  console.log("Database set up for ${resource}");
  await db.end();
}

setup();
EOF

log "Created db/connect.js, db/${resource}.sql and db/setup.js"

# Stretch: start every scaffolded app as a git repo
git init -q
log "Initialised a git repository"

log "Done. Run 'tree' to see your new structure."
```

Running it:

*(Run from `~/bash-training`)*
```bash
chmod +x scaffold
mv scaffold ~/bin/scaffold
```

*(Run from `~/`)*
```bash
mkdir books-app && cd books-app
```

*(Run from `~/books-app`)*
```bash
scaffold books Book
tree
cat server/models/Book.js     # -> class Book, SELECT * FROM books
cat server/index.js           # -> ${PORT} intact, proving the quoted heredoc worked
```

Step 7 — the guard firing:
```bash
cd ~ && mkdir empty-test && cd empty-test
scaffold                      # -> Usage: scaffold <resource> [ModelName]
echo $?                       # -> 1
```

**ASK** *(to close the morning)* <br>
We've hard-coded nothing about "countries" — it's all driven by arguments. Why does that make it dramatically more valuable to a team than a script that just creates a fixed countries app? <br>
**ANSWER** <br>
Because now it's a **tool**, not a one-off. Anyone can scaffold any resource with a consistent, agreed structure, and nobody has to remember the folder layout or argue about it in code review. Consistency across a team is one of the quiet superpowers of automation — and it's exactly why we'll write infrastructure as reusable Terraform modules later in the course. It's also, incidentally, what `npx create-react-app` and `express-generator` do. You just built one of those.

<br>
<br>

*(Take a 1 hour lunch break here.)*

<br>
<br>

### 14:00–14:15 — From Scripting to Automation for DevOps

Welcome back. This morning was about *writing scripts*. This afternoon is about **automation as a way of working** — a bigger idea than any single script.

A script is a **tool**. **Automation** is the discipline of making sure the important, repeatable work of running software happens the same way every time, without a human having to remember the steps or be awake to press the button.

Think back to the DevOps habits from Session 1 — "automate the boring, repeatable stuff", "small frequent changes", "everything as code". Scripts are how those habits actually get realised. Your scaffold script means every new service starts identically. A build script means every image is built identically. A deploy script means every release happens identically.

Once the steps live in a script, they can be:

- **Reviewed** — a teammate reads it in a pull request before it ever runs
- **Version-controlled** — you can see how the process changed over time, and roll it back
- **Triggered automatically** — on a schedule via **`cron`** (a scheduler built into every Linux system: "every night at 2am", "every 5 minutes", entirely unattended), or by an event like "someone pushed to `main`" (that's Jenkins, in a couple of sessions)
- **Run anywhere** — your laptop, a colleague's, a build server, a cloud shell — behaving the same

**ASK** <br>
We could keep doing all this by hand and get the same *result* on any given day. So what does automation actually buy a DevOps team — beyond saving typing? <br>
**ANSWER** <br>
Consistency and safety at scale. Humans doing repetitive steps by hand **drift** — they skip a step, do them out of order, do it slightly differently on one server. A script does it identically every time, leaves a reviewable record of *what* runs, **fails loudly** when something's wrong via exit codes, and runs without anyone present. The value isn't saved keystrokes; it's that the process becomes **reliable and auditable**.

**ASK** <br>
You already have automated tests. How is a deploy script different from a test suite, and how is it the same? <br>
**ANSWER** <br>
**Same:** both encode knowledge that would otherwise live in someone's head, both run identically every time, and both fail loudly rather than silently. **Different:** a test suite *observes* — it changes nothing. A deploy script *acts*, and its mistakes are real and sometimes irreversible. Which is why everything we do from here obsesses over failing safely: checking before acting, failing fast, and making it obvious when something went wrong.

Now — bash is the **lingua franca** of Linux. That phrase just means "the common language everyone shares": whatever else a team uses, they can all fall back on bash. And it's most of what a DevOps engineer scripts.

But it's not the only shell you'll meet, especially in a Microsoft-heavy environment like Azure. So next we step into **PowerShell**, inside the Azure Cloud Shell, and see how it's similar and how it's fundamentally different.

<br>
<br>
### 14:15–15:00 — PowerShell on the Azure Cloud Shell
*(Activity: 25 min)*

You've driven Azure two ways so far: clicking in the Portal, and typing `az` commands in bash. There's a third, and in a lot of Microsoft shops it's the primary one: **PowerShell**.

**Getting to it — the Cloud Shell**

Nothing to install. Azure has a browser-based terminal built into the Portal.

*(In the Azure Portal — top-right toolbar)*
- Sign in to `portal.azure.com`
- Click the **`>_`** icon (Cloud Shell)
- If prompted, choose **PowerShell** (not Bash) — there's a dropdown top-left to switch
- First time, it asks to create a small storage account to persist your files. Normal — let it

You now have a full PowerShell session, **already authenticated as you**, with the Azure `Az` module preinstalled. No login step, no CLI install — that's the appeal.

**ASK** <br>
This morning we spent time on `PATH` and where binaries live. The Cloud Shell already has `az`, `kubectl`, `terraform`, `git` and dozens more available. What does that tell you about it? <br>
**ANSWER** <br>
It's a **container** Microsoft prepared and runs for you, with all those tools already installed and on the `PATH`. That's why it starts instantly and needs no setup — and it's exactly the same reasoning behind the Docker images you already build. Someone did the install work once, into an image, so nobody has to do it again. That storage account it asks for exists because a container's own filesystem is disposable, so your files need somewhere persistent to live.

**The big conceptual difference: objects, not text**

The one thing to really understand, because it's genuinely different from bash.

In bash, everything flowing through a pipe is **text**. `ls | grep countries` passes a stream of characters, and `grep` pattern-matches on strings. It works, but it's crude — you're constantly slicing text apart to get at the bits you want.

In PowerShell, everything flowing through a pipe is a proper **object** — a structured thing with named properties. When you list files, each item isn't a line of text; it's a file object with `.Name`, `.Length`, `.LastWriteTime`. You filter and sort on those **properties** directly.

**ASK** <br>
This morning I asked how a bash pipeline compares to `.filter().map()`. Given what I've just said — which is PowerShell closer to? <br>
**ANSWER** <br>
**PowerShell is much closer to your JavaScript array methods.** `Where-Object { $_.Name -like "*.json" }` is essentially `.filter(f => f.name.endsWith('.json'))` — you're reading a property off a structured object, not pattern-matching a string. Which means PowerShell will feel *more* natural to you than bash did, and the mental adjustment is smaller than you'd expect. The trade-off is that bash's text-everything model works with any tool ever written, whereas PowerShell's objects only work with tools that produce PowerShell objects.

**Cmdlets — the Verb-Noun pattern**

PowerShell commands are **cmdlets** ("command-lets"), and they follow a rigid naming pattern: **`Verb-Noun`**.

| Cmdlet | What it does | Bash equivalent |
|---|---|---|
| `Get-ChildItem` | List items in a location | `ls` |
| `Set-Location` | Change directory | `cd` |
| `Get-Content` | Read a file | `cat` |
| `New-Item` | Create a file or folder | `touch` / `mkdir` |
| `Where-Object` | Filter to matching items | `grep` |
| `Get-Help` | Show docs for a cmdlet | `man` |
| `Get-Command` | List available commands | `which` |

Because the verbs are standardised (`Get`, `Set`, `New`, `Remove`, `Start`, `Stop`...), you can often **guess** a cmdlet's name. Remove something? `Remove-Item`. Start a service? `Start-Service`.

That predictability is deliberate — and the exact opposite of bash's terse, inconsistent names (`ls`, `cat`, `grep`, `rm`), which you simply have to memorise. Which of those trade-offs you prefer says a lot about whether you'll enjoy PowerShell.

PowerShell also ships **aliases** so bash muscle memory mostly works: `ls`, `cd`, `cat`, `pwd`, `cp`, `rm` are all aliased. Great for getting started — but write **full cmdlet names in scripts**, because aliases can differ between systems.

**Filtering and sorting — the pipeline in action**

*(Run in the Azure Cloud Shell, in **PowerShell** mode)*
```powershell
Get-ChildItem | Sort-Object Length -Descending | Select-Object -First 3
```

Read it: list the items, sort by their `Length` **property** (biggest first), keep the first three. No `wc`, no `head`, no text parsing.

*(Run in the Azure Cloud Shell, in **PowerShell** mode)*
```powershell
Get-ChildItem | Where-Object { $_.Name -like "*.json" }
```

Unpacking:
- `Where-Object` keeps items where the condition inside `{ }` is true — PowerShell's `grep`
- `$_` means "**the current object** flowing down the pipeline"
- `-like "*.json"` does wildcard matching

**Variables**

Same idea as bash, different punctuation — `$` on variables, and (**unlike bash**) you *do* put spaces around `=`:

*(Run in the Azure Cloud Shell, in **PowerShell** mode)*
```powershell
$name = "countries"
Write-Output "Scaffolding $name"
```

**Talking to Azure — the `Az` module**

This is why PowerShell matters for you specifically. Compare against the `az` CLI commands from Session 1:

| Task | Azure CLI (bash) | PowerShell (`Az` module) |
|---|---|---|
| Show current subscription | `az account show` | `Get-AzContext` |
| List resource groups | `az group list -o table` | `Get-AzResourceGroup` |
| Create a resource group | `az group create --name rg1 --location uksouth` | `New-AzResourceGroup -Name rg1 -Location uksouth` |
| List all resources | `az resource list -o table` | `Get-AzResource` |
| Delete a resource group | `az group delete --name rg1` | `Remove-AzResourceGroup -Name rg1` |

Both do the *same thing* under the hood — both are wrappers around the same **Azure REST API**, the point we made in Session 1. Which you use comes down to your team and what the rest of your tooling is written in.

**ASK** <br>
Both hit the same REST API and return the same resource groups. So is learning PowerShell as well as the CLI a waste of time? <br>
**ANSWER** <br>
No — because **you don't get to choose the environment you walk into**. Plenty of enterprises, especially Microsoft-heavy ones, have years of existing automation in PowerShell, use Azure Automation runbooks (which are PowerShell), and expect you to work in it. Turning up and rewriting it all in bash is not an option. And the object pipeline genuinely makes some tasks cleaner. Being fluent in both means you're useful whatever the team already runs.

**HANDS ON (25 min)** <br>
*(Run everything in this exercise in the **Azure Cloud Shell**, in **PowerShell** mode)*

Part A — get your bearings.
1. Run `Get-ChildItem`, then its alias `ls`. Confirm they're identical
2. Run `Get-Command -Verb Get | Select-Object -First 20` — see how many "Get" cmdlets exist
3. Run `Get-Help Get-AzResourceGroup` — the built-in docs for any cmdlet

Part B — objects, not text.
4. `Get-ChildItem | Sort-Object Length -Descending | Select-Object Name, Length -First 5`. Notice you selected **properties by name**
5. `Get-Process | Where-Object { $_.CPU -gt 1 } | Select-Object Name, CPU` — filtering live processes on a **numeric** property

Part C — Azure with the Az module.
6. `Get-AzContext` — which subscription are you in?
7. `Get-AzResourceGroup` — list them as objects
8. `New-AzResourceGroup -Name ps-demo-rg -Location uksouth`
9. Confirm it exists, then filter for it with `Where-Object`
10. Clean it up with `Remove-AzResourceGroup`

Part D — comparison.
11. In your **dictionary**, add a short table: for five bash commands you learned this morning, write the PowerShell equivalent. Do it from memory first, then check

Part E (stretch).
12. Store a resource group in a variable and pipe it to `Get-Member` — this lists **every property and method** the object has. It's how you *discover* what you can do with something in PowerShell
**END OF NOTE**

**💬 SLACK — snippet 6**:
```powershell
# Part A
Get-ChildItem
ls
Get-Command -Verb Get | Select-Object -First 20
Get-Help Get-AzResourceGroup

# Part B
Get-ChildItem | Sort-Object Length -Descending | Select-Object Name, Length -First 5
Get-Process | Where-Object { $_.CPU -gt 1 } | Select-Object Name, CPU

# Part C
Get-AzContext
Get-AzResourceGroup
New-AzResourceGroup -Name ps-demo-rg -Location uksouth
Get-AzResourceGroup | Where-Object { $_.ResourceGroupName -like "ps-demo*" }
Remove-AzResourceGroup -Name ps-demo-rg -Force

# Part E
$rg = Get-AzResourceGroup -Name ps-demo-rg
$rg | Get-Member
```

**Solution**

*(Run in the Azure Cloud Shell, in **PowerShell** mode)*
```powershell
# --- Part A ---
Get-ChildItem
ls                                    # identical — 'ls' is an alias for Get-ChildItem
Get-Command -Verb Get | Select-Object -First 20
Get-Help Get-AzResourceGroup

# --- Part B ---
Get-ChildItem | Sort-Object Length -Descending | Select-Object Name, Length -First 5
Get-Process | Where-Object { $_.CPU -gt 1 } | Select-Object Name, CPU

# --- Part C ---
Get-AzContext
Get-AzResourceGroup
New-AzResourceGroup -Name ps-demo-rg -Location uksouth
Get-AzResourceGroup | Where-Object { $_.ResourceGroupName -like "ps-demo*" }
Remove-AzResourceGroup -Name ps-demo-rg -Force

# --- Part E (stretch) ---
$rg = Get-AzResourceGroup -Name ps-demo-rg
$rg | Get-Member
```

**Part D — the comparison table they should end up with:**

| Bash | PowerShell | Note |
|---|---|---|
| `ls -la` | `Get-ChildItem -Force` | `-Force` shows hidden items |
| `grep "text"` | `Where-Object { $_.Name -like "*text*" }` | bash matches **text**; PowerShell matches a **property** |
| `head -5` | `Select-Object -First 5` | |
| `sort` | `Sort-Object` | PowerShell sorts by a **named property**, bash sorts lines |
| `wc -l` | `Measure-Object` | e.g. `Get-ChildItem \| Measure-Object` |
| `cat file` | `Get-Content file` | |
| `which git` | `Get-Command git` | |

The row worth discussing is `grep` versus `Where-Object`. They're not really equivalent: `grep` searches **raw text** and works on the output of literally any program ever written. `Where-Object` reads a **property** and only works if the thing upstream produced a PowerShell object. More powerful when it applies; useless when it doesn't.

**NOTE FOR TRAINERS** <br>
If students are on a fresh Azure account with no resource groups, steps 7–9 still work — the list starts empty then shows `ps-demo-rg`. Emphasise `Get-Member` in the stretch: **"when you don't know what an object can do, pipe it to `Get-Member`"** is the single most useful habit for learning PowerShell independently, and it's the closest thing PowerShell has to `console.log(Object.keys(x))`. <br>
**END OF NOTE**

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>

### 15:15–16:45 — Capstone: Scaffold an App & Automate a Container Build
*(Activity: 90 min)*

The main event. It deliberately pulls together everything you know: this morning's **bash scripting**, your existing **Docker** skills, and your **Azure** knowledge.

You'll build **two real, named scripts** a DevOps engineer would genuinely keep in their toolkit.

**The scenario:** your team spins up small Express services constantly, and every one needs scaffolding consistently *and* containerising and pushing to a registry the same way every time. You're automating both halves.

Work individually or in pairs. Get Parts 1 and 2 working; Part 3 is stretch.

**NOTE FOR TRAINERS** <br>
Tell them explicitly at the start: **Part 1 is mostly assembly, Part 2 is mostly thinking.** `new-app` is their morning scaffold plus two extra files — encourage copying from their own script rather than retyping. `containerise` is genuinely new logic and where the learning is, so protect the time for it. If someone is still fighting Part 1 at 16:00, give them the completed `new-app` from `completed-code/` and move them on. <br>
**END OF NOTE**

---

#### Part 1 (≈30 min) — Script one: `new-app`, the scaffolder

Take your morning `scaffold` script and harden it into a tool called `new-app`.

**Requirements:**
1. Takes a resource name as `$1` and an optional model name as `$2` (defaulting to a capitalised resource)
2. **Validates input**: no resource name → clear usage message to `>&2` and `exit 1`
3. **Refuses to overwrite**: if `server/` already exists here, warn and `exit 1`
4. Creates the full MVC structure from this morning
5. **Also** writes a working `Dockerfile` into the project root, so Part 2 has something to build
6. **Also** writes a `.dockerignore` containing `node_modules` and `.git`
7. Uses a `log()` function so every step prints a **timestamped** message
8. Ends by printing a clear "next steps" message
9. Install it to `~/bin/new-app` so it's callable from anywhere

**ASK** *(before they start)* <br>
Requirement 6 is a `.dockerignore` with `node_modules` in it. Why does that matter for the build in Part 2? <br>
**ANSWER** <br>
Because `docker build` sends the whole folder to the Docker daemon as **build context** first. Without `.dockerignore`, a local `node_modules` — potentially hundreds of megabytes — gets copied in, making the build slow, and then `COPY . .` overwrites the `node_modules` the image just installed with **your host's** version, which may have been built for a different architecture. It's a small file that prevents a genuinely confusing class of bug.

**Test it:**

*(Run from `~/`)*
- Run: `mkdir countries-app` → **cd inside**

*(Run from `~/countries-app`)*
```bash
new-app countries Country
tree
```

**💬 SLACK — snippet 7.** Post the two Docker files — there's no learning in transcribing them:
```dockerfile
# Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```
```
# .dockerignore
node_modules
.git
```

**Solution**

`new-app` — the morning's `scaffold` plus a `Dockerfile`, a `.dockerignore`, timestamped logging and next steps. Only the **new** parts are commented; the rest is unchanged from this morning:

```bash
#!/bin/bash
# new-app — scaffold an Express MVC app, ready to containerise
# Usage: new-app <resource> [ModelName]

set -e

resource="$1"
model="${2:-${resource^}}"

# 2. Validate input
if [ -z "$resource" ]; then
  echo "Usage: new-app <resource> [ModelName]" >&2
  echo "  e.g. new-app countries Country" >&2
  exit 1
fi

# 3. Refuse to overwrite
if [ -d "server" ]; then
  echo "A server/ directory already exists here — aborting so nothing is overwritten." >&2
  exit 1
fi

# 7. Timestamped logger — note the $(date) inside
log() { echo "[$(date +%H:%M:%S)] new-app: $1"; }

log "Scaffolding '$resource' (model '$model')"

for dir in controllers db models routers; do
  mkdir -p "server/$dir"
done

cat > package.json << EOF
{
  "name": "${resource}-api",
  "version": "1.0.0",
  "main": "server/index.js",
  "scripts": {
    "start": "node server/index.js",
    "db:setup": "node server/db/setup.js"
  },
  "dependencies": {
    "express": "^4.19.2",
    "pg": "^8.11.5"
  }
}
EOF

cat > README.md << EOF
# ${resource} API

A small Express MVC API for ${resource}, generated by new-app.
EOF

cat > server/index.js << 'EOF'
const app = require("./app");

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Listening on port ${PORT}`));
EOF

cat > server/app.js << EOF
const express = require("express");
const ${resource}Router = require("./routers/${resource}");

const app = express();
app.use(express.json());

app.use("/${resource}", ${resource}Router);

module.exports = app;
EOF

cat > server/controllers/${resource}.js << EOF
const ${model} = require("../models/${model}");

async function index(req, res) {
  const rows = await ${model}.findAll();
  res.json(rows);
}

module.exports = { index };
EOF

cat > server/routers/${resource}.js << EOF
const express = require("express");
const controller = require("../controllers/${resource}");

const router = express.Router();
router.get("/", controller.index);

module.exports = router;
EOF

cat > server/models/${model}.js << EOF
const db = require("../db/connect");

class ${model} {
  static async findAll() {
    const result = await db.query("SELECT * FROM ${resource}");
    return result.rows;
  }
}

module.exports = ${model};
EOF

cat > server/db/connect.js << 'EOF'
const { Pool } = require("pg");

const db = new Pool({
  connectionString: process.env.DATABASE_URL,
});

module.exports = db;
EOF

cat > server/db/${resource}.sql << EOF
DROP TABLE IF EXISTS ${resource};

CREATE TABLE ${resource} (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL
);

INSERT INTO ${resource} (name) VALUES ('Example one'), ('Example two');
EOF

cat > server/db/setup.js << EOF
const fs = require("fs");
const path = require("path");
const db = require("./connect");

async function setup() {
  const sql = fs.readFileSync(path.join(__dirname, "${resource}.sql"), "utf-8");
  await db.query(sql);
  console.log("Database set up for ${resource}");
  await db.end();
}

setup();
EOF

# 5 + 6. The Docker files — QUOTED markers, since nothing needs substituting
cat > Dockerfile << 'EOF'
FROM node:20-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
EOF

cat > .dockerignore << 'EOF'
node_modules
.git
EOF

log "Scaffold complete."

# 8. Next steps
echo ""
echo "Next steps:"
echo "  npm install"
echo "  containerise ${resource}-api v1"
```

Installing and running:

*(Run from `~/bash-training`)*
```bash
chmod +x new-app
mv new-app ~/bin/new-app
```

*(Run from `~/`)*
```bash
mkdir countries-app && cd countries-app
```

*(Run from `~/countries-app`)*
```bash
new-app countries Country
tree
cat Dockerfile        # confirm the quoted heredoc kept it literal
```

---

#### Part 2 (≈40 min) — Script two: `containerise`, the build-and-push

**This is the part with the real learning in it.** Write a script called `containerise`.

**Requirements:**
1. **Usage:** `containerise <image-name> [tag]`, tag defaulting to `latest`
2. **Validate input:** no image name → usage message to `>&2` and `exit 1`
3. **Pre-flight checks** — fail early and clearly if the environment isn't ready:
   - No `Dockerfile` in the current directory (`-f`) → error and `exit 1`
   - `docker` not available (hint: `command -v docker`) → error and `exit 1`
4. **Build** the image, tagged with the name and tag
5. **Check the build's exit code.** If it failed, clear failure message to `>&2` and `exit 1`
6. **Report success** with the full image name and tag
7. Use the same timestamped `log()` helper
8. Install it to `~/bin/containerise`

**Test it:**

*(Run from `~/countries-app`)*
```bash
containerise countries-api v1
docker images
```

**Then — the push (Azure tie-in).** Extend it so that if a registry name is supplied as a **third** argument, it also pushes to **Azure Container Registry**.

**Solution**

```bash
#!/bin/bash
# containerise — build a Docker image from the current app, optionally push to ACR
# Usage: containerise <image-name> [tag] [acr-registry-name]

set -e

image="$1"
tag="${2:-latest}"     # default the tag to 'latest' if none given
registry="$3"          # optional third argument

log() { echo "[$(date +%H:%M:%S)] containerise: $1"; }

# 2. Validate input
if [ -z "$image" ]; then
  echo "Usage: containerise <image-name> [tag] [acr-registry-name]" >&2
  exit 1
fi

# 3. Pre-flight checks
if [ ! -f "Dockerfile" ]; then
  echo "No Dockerfile found in $(pwd) — are you in the app folder?" >&2
  exit 1
fi

if ! command -v docker > /dev/null; then
  echo "docker is not installed or not on your PATH." >&2
  exit 1
fi

# 4 + 5. Build, and fail loudly if the build fails
log "Building ${image}:${tag}"
if ! docker build -t "${image}:${tag}" . ; then
  echo "Docker build FAILED for ${image}:${tag}" >&2
  exit 1
fi

# 6. Success
log "Built ${image}:${tag} successfully"

# Optional: push to Azure Container Registry
if [ -n "$registry" ]; then
  full="${registry}.azurecr.io/${image}:${tag}"

  log "Tagging for ACR as $full"
  docker tag "${image}:${tag}" "$full"

  log "Logging in to ACR '$registry'"
  az acr login --name "$registry" || { echo "ACR login failed" >&2; exit 1; }

  log "Pushing $full"
  docker push "$full" || { echo "Push failed" >&2; exit 1; }

  log "Pushed $full successfully"
fi
```

Four bits of new syntax worth explaining to the room:

- **`if [ ! -f "Dockerfile" ]`** — `!` means "not", so this reads "if a Dockerfile does **not** exist"
- **`if ! command -v docker > /dev/null`** — `command -v docker` looks up whether `docker` exists and prints its path (it's the scriptable cousin of the `which` you used this morning). `> /dev/null` throws that printout away, because we only care **whether it succeeded**, not what it said. `/dev/null` is a special "black hole" file that discards anything written to it. The leading `!` flips the result, so the block runs when docker is **absent**
- **`|| { echo "..." >&2; exit 1; }`** — the `||` from this morning: "if the left-hand command failed, run this block instead". `{ }` groups multiple commands. A compact way to bail out with a message
- **`${registry}.azurecr.io/${image}:${tag}`** — building an image name out of three variables. Note the `{ }` braces around each: without them, bash would read `$registry.azurecr` as a variable named `registry.azurecr`. Braces mark where the variable name ends

**ASK** <br>
Step 3 checks for a `Dockerfile` and for `docker` **before** doing anything. Why not just let `docker build` fail on its own? <br>
**ANSWER** <br>
Two reasons. First, **the error message**. `docker build` failing in a missing-Dockerfile folder produces something cryptic; your check produces "No Dockerfile found in /home/you/wrong-folder — are you in the app folder?" which tells the user exactly what to do. Second, **failing cheaply**. Checks that cost nothing should run before work that costs something. That ordering — cheapest checks first — becomes a genuine design principle when we build pipelines in a few sessions, where a stage might take ten minutes.

Installing and running:

*(Run from `~/bash-training`)*
```bash
chmod +x containerise
mv containerise ~/bin/containerise
```

*(Run from `~/countries-app`)*
```bash
npm install
containerise countries-api v1
docker images
```

**And test that it fails properly** — this matters as much as testing that it works:

*(Run from `~/` — deliberately the wrong folder)*
```bash
containerise countries-api v1     # -> "No Dockerfile found in /home/you"
echo $?                           # -> 1
```

**NOTE FOR TRAINERS** <br>
Provisioning an ACR per student may not be practical. Options, in order of preference: **(a)** provide **one shared ACR** for the room, students push under distinct image names; **(b)** students create a Basic-tier ACR in their own subscription (`az acr create --name <unique> --resource-group <rg> --sku Basic`) and tear it down after; **(c)** if registries aren't available, stop after the local `docker build` and just *write* the push logic without running it — the scripting practice is the real objective. <br>
Note Docker must run on the student's **local machine or VM**, **not** the Azure Cloud Shell, which has no Docker daemon. <br>
**END OF NOTE**

**ASK** *(mid-exercise checkpoint, ~16:15)* <br>
We deliberately check the exit code of `docker build` and `exit 1` if it failed. Imagine this script is later run automatically by a Jenkins pipeline. What goes wrong if we *skip* that check and always exit `0`? <br>
**ANSWER** <br>
The pipeline thinks the build succeeded even when it didn't, and carries on to the next stage — deploying, tagging as "latest", telling everyone it's green — while shipping a broken or stale image. **Honest exit codes are the contract between your script and every automated system that will ever run it.** This is the single most important idea from today, and it's why we've come back to exit codes four times.

---

#### Part 3 (≈20 min) — Stretch: chain them and add polish

If both scripts work, level them up:

1. **A one-command workflow.** Write a third tiny script, `ship`, that runs `new-app` then `containerise` using `&&`. Think about what should happen if the scaffold fails — should the build still run?
2. **Logging to a file.** Make `containerise` write output to a dated log file *as well as* the screen, the way an unattended job would need
3. **A confirmation prompt.** Before pushing, use `read -p` to ask "Push to `<registry>`? (y/n)" and only push if `y`. *(Where would that be a problem in an automated pipeline? Discuss.)*
4. **Idempotency.** Make `new-app` safe to re-run: instead of failing when `server/` exists, skip files that already exist and create only the missing ones

**Solution**

**1. `ship`** — because of `&&`, `containerise` only runs if `new-app` fully succeeded, so a failed scaffold never gets built:

```bash
#!/bin/bash
# ship — scaffold an app then containerise it, in one command
# Usage: ship <resource> [ModelName]

set -e

resource="$1"
if [ -z "$resource" ]; then
  echo "Usage: ship <resource> [ModelName]" >&2
  exit 1
fi

new-app "$resource" "$2" && containerise "${resource}-api" v1
```

**2. Logging to a file** — simplest version is redirecting at call time. `>>` appends, `2>&1` folds the error channel into the same file so failures are captured too:

*(Run from `~/countries-app`)*
```bash
containerise countries-api v1 >> build-$(date +%F).log 2>&1
```

Or bake it into the script, just after `log()`. `tee -a` writes to the screen **and** appends to the file:

```bash
exec > >(tee -a "build-$(date +%F).log") 2>&1
```

**3. Confirmation prompt** — inside the `if [ -n "$registry" ]` block, before tagging:

```bash
read -p "Push ${image}:${tag} to ${registry}? (y/n) " answer
if [ "$answer" != "y" ]; then
  log "Push cancelled by user"
  exit 0
fi
```

*(Discussion point: this would **hang forever** in an automated pipeline — no human to type `y`, so the job sits there until it times out. In real automation you'd drive it with a flag or environment variable instead. A good example of a habit that's helpful by hand and harmful in automation, and the same trap as `apt install` without `-y`.)*

**4. Idempotent `new-app`** — swap the hard failure for a per-file skip:

```bash
# a small helper, defined near log()
# Takes: $1 = target path. Returns 0 if we should write, 1 if we should skip.
write_if_missing() {
  if [ -f "$1" ]; then
    log "$1 already exists, skipping"
    return 1
  fi
  return 0
}

# then guard each file:
if write_if_missing package.json; then
cat > package.json << EOF
...
EOF
fi
```

*(Fiddly to retrofit onto every heredoc. A common real-world answer is to generate into a temp folder and copy across only what's missing. Fine to discuss rather than fully implement.)*

---

#### What to show at 16:45

Be ready to demonstrate, in a completely fresh folder:

*(Run from `~/`)*
```bash
mkdir books-demo && cd books-demo
```

*(Run from `~/books-demo`)*
```bash
new-app books Book          # scaffolds the app + Dockerfile
npm install
containerise books-api v1   # builds the image
docker images               # proves it exists
```

And be ready to explain **one place** where you used an **exit code** to fail safely, and why it matters.

<br>
<br>

### 16:45–17:00 — Wrap-up & Q&A

Let's pull the day together.

This morning you went from gluing two commands together with `&&`, to writing a **named command** that scaffolds an entire application from a single word of input. That's a genuine leap — you've written a real tool, and it lives on your `PATH` alongside `git` and `docker`.

You also learned where things actually live on a Linux machine — `/bin`, `/usr/bin`, `/usr/local/bin`, `/etc`, `/var` — which will stop those paths looking arbitrary when they turn up in Terraform and Kubernetes later.

This afternoon you saw that automation is bigger than any one script: it's the discipline that makes repetitive work consistent, reviewable and safe to run without a human watching. You met PowerShell and saw that its object pipeline is genuinely closer to your JavaScript instincts than bash's text streams. And in the capstone you connected all three worlds — bash, Docker and Azure — into two scripts that scaffold and containerise an app the same way, every time.

**ASK** <br>
Everything you built today runs when *you* type it. What's the missing piece that would make these run **automatically** — every time someone pushes code — with nobody typing anything? <br>
**ANSWER** <br>
A **pipeline**. Something that watches for an event (a Git push) and runs your scripts for you, reading their exit codes to decide pass or fail. That's the **Jenkins** session coming up — and the reason we hammered exit codes today is that they are **the only language a pipeline speaks**. Your scripts are already pipeline-ready; all that's missing is the thing that pulls the trigger.

Where this sits in the course:
- **Today** — you can script, name and run automation by hand, and you know why it matters
- **Jenkins & pipelines** — those scripts get triggered automatically on every code change
- **Terraform & IaC** — the same "everything as code" habit, applied to Azure infrastructure
- **Kubernetes** — running and scaling the containers you learned to build and push
- **Integration** — all of it in one pipeline: code change → running infrastructure, automatically and safely

**Before you go:** make sure your `my-command-dictionary.md` is filled in and committed. It's the thing from today you'll actually use again, and writing it in your own words is what makes it stick.

**Q&A** — take remaining questions.

<br>
<br>

### Exercise (take-home / reinforcement)

Before the next session, individually or in pairs. **Do at least one practical and one research task.**

**Practical**

1. Rebuild `new-app` **from memory**, without looking at today's notes, and get it scaffolding correctly. Rebuilding from scratch is the fastest way to make it stick
2. Extend `new-app` to accept **multiple** resource names at once — `new-app countries books users` — creating a controller, router, model and SQL file for each. *(Hint: loop over `$@`)*
3. Add a `--help` flag to both scripts: first argument `--help` or `-h` → print usage and `exit 0`
4. Write a small `df-check` script that logs disk usage with a timestamp to a file, and schedule it with `cron` to run every 5 minutes for testing

**Research**

5. **Finish your command dictionary.** Every row filled in, in your own words, plus the three commands you found yourself. Commit it to a repo — this is a thing you'll genuinely refer back to
6. **Map the filesystem.** Pick **three** of `/etc`, `/var`, `/opt`, `/tmp`, `/usr/local/bin`, `/home`. For each, write two sentences on what it's for, and find one **real example** on your own machine. (`ls /etc | head -20` is a good start)
7. Write a short paragraph, for a colleague who's never used a terminal, explaining **what `PATH` is and why it exists**. If you can explain it simply, you understand it
8. **(Stretch)** Research and write down the difference between `set -e`, `set -u` and `set -o pipefail`, and why experienced engineers often put `set -euo pipefail` at the top of a serious script
9. **(Stretch)** In PowerShell in the Cloud Shell, write a one-liner listing all your resource groups with just names and locations, sorted by location

**Solutions** *(for the guided ones — 2, 3, 8, 9)*

**Take-home 2** — looping over every argument with `$@`:
```bash
#!/bin/bash
# new-app (multi) — scaffold one resource per argument
set -e

if [ -z "$1" ]; then
  echo "Usage: new-app <resource> [resource2 ...]" >&2
  exit 1
fi

# "$@" quoted keeps arguments containing spaces intact
for resource in "$@"; do
  model="${resource^}"
  mkdir -p server/controllers server/models server/routers server/db
  touch "server/controllers/${resource}.js" \
        "server/routers/${resource}.js" \
        "server/models/${model}.js" \
        "server/db/${resource}.sql"
  echo "Scaffolded $resource"
done
```

**Take-home 3** — a `--help` flag. Goes right after the shebang, **before** the input guard, so `--help` doesn't trip the "no resource name" error:
```bash
if [ "$1" = "--help" ] || [ "$1" = "-h" ]; then
  echo "Usage: new-app <resource> [ModelName]"
  echo "  Scaffolds an Express MVC app in the current folder."
  exit 0        # note: 0, not 1 — asking for help is not a failure
fi
```

That `exit 0` is worth noticing. Printing help is a **successful** run of the program, so it must not report failure — otherwise a pipeline running `new-app --help` would mark itself red.

**Take-home 8** — the three `set` options:

| Option | Does | Why |
|---|---|---|
| `set -e` | Exit immediately if any command fails | Stops a script charging on after an error and making things worse |
| `set -u` | Treat an **undefined variable** as an error | Catches typos. Without it, `rm -rf "$DIRR/"` with a misspelt variable becomes `rm -rf "/"` |
| `set -o pipefail` | A pipeline fails if **any** command in it fails, not just the last | Without it, `false \| echo "ok"` reports success, hiding the failure |

Together, `set -euo pipefail` means "fail fast, fail on typos, and don't let failures hide inside pipelines". It's the standard opening line of a serious bash script.

**Take-home 9** — the PowerShell one-liner:
```powershell
Get-AzResourceGroup | Sort-Object Location | Select-Object ResourceGroupName, Location
```

<br>
<br>

### Conclusion

- **Inform** students this marks the end of the Bash Scripting & Automation session
- **Confirm** everyone's `my-command-dictionary.md` is filled in and committed — it's the reusable artefact from today
- **Tell** students the next session moves into **Jenkins & pipelines**, where the scripts they wrote today start running automatically on every code change — and where today's obsession with exit codes pays off
- **Direct** students to the take-home exercises, and to the [Bash reference](https://www.gnu.org/software/bash/manual/bash.html), the [Filesystem Hierarchy Standard](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html) and the [Azure PowerShell docs](https://learn.microsoft.com/en-us/powershell/azure/)

---

[Back](./README.md)

---
