# Bash Scripting & Automation for DevOps — Trainer Script

A full day taking students from "I can move around the filesystem" to "I can write, name, and run real scripts that scaffold an application and automate a container build". This is written as a script — read it out, adapt where it feels natural, but the intent is that you could deliver this cold from the page.

## Organisation

### Audience

Students who have already covered **Azure cloud fundamentals** and **Docker**, and who are comfortable with a handful of navigation commands (`cd`, `ls`, `mkdir`) but have **not** written a shell script before. They know Express APIs and `package.json` well from earlier work, so the *contents* of the files we generate will be familiar — but **bash logic (`if` statements, loops, functions) is completely new**, so those sections are explained slowly and from first principles.

### How this document is laid out — read before delivering

**Every command block has a *Run from* line above it**, so you always know which folder you should be sitting in. Where a command changes directory, that's called out explicitly, e.g.:

- Run: `mkdir server` → **cd inside**

The folders we use today:

| Folder | What it's for |
|---|---|
| `~/bash-training` | Scratch folder for all morning practice commands |
| `~/bin` | Where finished scripts get installed so they run by name |
| `~/countries-app`, `~/books-app` | The scaffolded applications we generate |

**Every activity has a `**Solution**` block** immediately after it, so you can reveal the answer without hunting for it.

Set the scratch folder up before you start:

*(Run from `~/`)*
```bash
mkdir -p ~/bash-training
cd ~/bash-training
```

### Duration & schedule

Full day, **09:00–17:00**, with a one-hour lunch and two 15-minute breaks. Roughly **4 hours of instruction and 3 hours of hands-on exercises**, weighted towards a large end-of-day capstone.

| Time | Section | Shape |
|---|---|---|
| 09:00–09:15 | Welcome & recap: where we are | Talk |
| 09:15–09:45 | The shell, and chaining commands together | Talk + demo |
| 09:45–10:15 | Hands-on: chaining & pipelines | **Exercise (30 min)** |
| **10:15–10:30** | **Break** | |
| 10:30–11:00 | Your first script: shebang, permissions, running it | Talk + demo + short exercise |
| 11:00–11:30 | Giving a script a name, and feeding it input | Talk + demo + short exercise |
| 11:30–12:00 | Logic: conditionals, loops and functions | Talk + demo + **challenge** |
| **12:00–12:15** | **Break** | |
| 12:15–13:00 | Building an app scaffold: the `countries` script | **Exercise (45 min)** |
| **13:00–14:00** | **Lunch** | |
| 14:00–14:15 | From scripting to automation for DevOps | Talk |
| 14:15–15:00 | PowerShell on the Azure Cloud Shell | Talk + demo + exercise |
| **15:00–15:15** | **Break** | |
| 15:15–17:00 | Capstone: scaffold an app & automate a container build | **Exercise (1 hr 45 min)** |

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
  - Students have a **Linux environment** with a real terminal: WSL2 on Windows, macOS's terminal, or the VM from the earlier Azure/Terraform sessions all work
  - **Docker** is installed and working (`docker --version`) — needed for the afternoon capstone
  - Students can sign in to the **Azure Portal** (`portal.azure.com`) — needed for the PowerShell / Cloud Shell section
  - A text editor they're comfortable with — VS Code is ideal, `nano` is fine

## Learning objectives

- **Chain** commands together with `;`, `&&`, `||`, pipes and redirection, and understand exit codes
- **Write** a bash script with a shebang, make it executable, and run it
- **Give** a script a name and put it on the `PATH` so it runs like any built-in command
- **Use** variables, positional arguments, conditionals, loops and functions in bash
- **Scaffold** a complete Express MVC application from a single script
- **Understand** what makes automation a DevOps *discipline*, not just a convenience
- **Navigate** the Azure Cloud Shell and run basic PowerShell, including the `Az` module
- **Combine** everything in a capstone: one script that scaffolds an app, another that builds and pushes a Docker image

<br>

## Sequence

### 09:00–09:15 — Welcome & Recap: Where We Are

Morning everyone. Quick recap of where we are, because today builds directly on two things you've already done.

You've covered **Azure fundamentals** — resource groups, the Portal, the CLI, the idea that everything clickable has a scriptable equivalent. And you've covered **Docker** — building an image from a Dockerfile, running containers, pushing images to a registry.

Today is the glue between those. It's about the **shell** and **scripting** — the single most-used skill a DevOps engineer has, and the thing that turns "I did a task once, by hand" into "this task now runs the same way every time, for everyone, forever".

Here's the shape of the day. This morning is pure bash: we'll go from chaining a couple of commands together, up to writing a proper named script that scaffolds an entire application's file structure in one command. After lunch we shift gears — we'll talk about what automation actually *means* for a DevOps engineer, have a look at PowerShell inside Azure's Cloud Shell, and then spend the back half of the afternoon on a big hands-on capstone that pulls bash, Docker and Azure together.

You already know `cd`, `ls`, and `mkdir` — so we're not starting from zero. But `if` statements, loops and functions in bash will be brand new to most of you, and I'll take those slowly. Lunch is at 1 for an hour; we'll break mid-morning and mid-afternoon, and shout if you need one sooner.

**Everyone set up a scratch folder now, so we're all in the same place:**

*(Run from `~/`)*
- Run: `mkdir -p ~/bash-training` → **cd inside** with `cd ~/bash-training`
- Confirm where you are with `pwd`

<br>
<br>

### 09:15–09:45 — The Shell, and Chaining Commands Together

Let's start with a quick reframe of what the shell actually *is*, because it changes how you think about everything else today.

When you type `ls` and hit enter, you're not talking to some magic box — you're talking to a program called a **shell**, and on almost every Linux server and container that shell is **bash**. Bash reads what you type, works out which command you meant, runs it, shows you the output, and waits for the next line. That's the whole loop.

Before we chain anything, one idea to hold onto: **every command has an input, an output, and a place it sends errors.** By default output and errors both land on your screen. Almost everything clever we do today is just *redirecting* where that output goes — into another command, or into a file. Keep that picture in mind.

The powerful bit — the bit we're going to lean on all day — is that bash lets you **combine** commands. You already know the commands individually. Today is mostly about the operators *between* them.

**Running things in sequence with `;`**

A semicolon means "run this, **then** run that, regardless of what happened".

*(Run from `~/bash-training`)*
```bash
mkdir project ; cd project ; ls
```

Three commands, one line, run in order. Read the `;` as the word "then". The catch: the semicolon doesn't care if a command *failed*. If `mkdir project` fails because the folder already exists, bash still barrels on and tries to `cd` and `ls`.

*Note you are now inside `~/bash-training/project` — hop back out:*

*(Run from `~/bash-training/project`)*
```bash
cd ~/bash-training
```

**Running the next thing only if the last one worked, with `&&`**

This is the one you'll use constantly.

*(Run from `~/bash-training`)*
```bash
mkdir project2 && cd project2
```

`&&` means "only run the right-hand command **if** the left-hand one succeeded." Read it as "and-then-only-if-that-worked". If the folder couldn't be made, we don't blunder on into it.

*(Run from `~/bash-training/project2`)*
```bash
cd ~/bash-training
```

Another everyday example — only install if the package list refreshed successfully:

*(Run from `~/bash-training`)*
```bash
sudo apt update && sudo apt install -y git
```

**Running something only if the last thing *failed*, with `||`**

The mirror image. Read `||` as "or-else".

*(Run from `~/bash-training`)*
```bash
cd project-that-does-not-exist || echo "Couldn't enter that folder"
```

"Try to `cd`; if that fails, print the warning." You'll often see `&&` and `||` used together to make a tiny inline if/else:

*(Run from `~/bash-training`)*
```bash
cd project && echo "It worked" || echo "Couldn't enter project folder"
```

*(That one leaves you inside `project` — `cd ~/bash-training` again.)*

**ASK** <br>
What's the practical difference between `mkdir app ; cd app` and `mkdir app && cd app`? Why might the second one save you from a nasty mistake? <br>
**ANSWER** <br>
With `;`, if `mkdir app` fails (say the folder already exists, or you don't have permission), bash still runs `cd app`. With `&&`, a failed `mkdir` stops the chain, so you don't accidentally carry on operating in the wrong directory. In a script that then deletes or overwrites files, that difference can be the difference between a clean run and wiping the wrong folder.

**Piping output from one command into another with `|`**

This is the idea that makes the shell feel like a superpower. A **pipe** (`|`) takes the *output* of the command on its left and feeds it in as the *input* of the command on its right. Instead of the first command's output landing on your screen, it flows straight into the next command.

Before we pipe, let's meet the three small commands we'll be piping *into*, because they're new:

- **`sort`** — takes lines of text and puts them in alphabetical order. `sort -r` reverses it (Z to A).
- **`head -5`** — takes lines of text and keeps only the **first 5**. (`tail -5` does the same for the *last* 5.)
- **`wc -l`** — "word count, lines". Takes lines of text and tells you **how many** there were.
- **`grep "something"`** — the filter. Takes lines of text and keeps **only the lines containing** "something", throwing the rest away. Think of it as "search". It's the single most-used command in a DevOps engineer's day, usually for hunting through logs.

Now the pipeline. Build this up one stage at a time on screen so students watch the list shrink:

*(Run from `~/bash-training`)*
```bash
ls /etc
```
That prints a long list of the system's configuration files. Now sort it:

*(Run from `~/bash-training`)*
```bash
ls /etc | sort
```
Now keep only the first five:

*(Run from `~/bash-training`)*
```bash
ls /etc | sort | head -5
```

Each command does one small job; the pipe stitches them into a **pipeline**, output flowing left to right. That's the Unix philosophy in one line: lots of small tools that do one thing well, joined together.

Let's see `grep` filter rather than sort:

*(Run from `~/bash-training`)*
```bash
ls /etc | grep conf
```
That keeps only the entries whose names contain "conf". And to count them instead of listing them:

*(Run from `~/bash-training`)*
```bash
ls /etc | grep conf | wc -l
```

**ASK** <br>
Given what you now know about `grep` and `wc -l`, what do you think this command does? <br>
```bash
cat access.log | grep "404" | wc -l
```

**ANSWER**<br>
"Read the log file, keep only the lines containing 404, then count how many lines that is." You've just counted your 404 errors without ever opening the file. That exact pipeline — `cat` a log, `grep` for the error, `wc -l` to count — is something you'll type in real jobs constantly.

**Redirecting output into a file with `>` and `>>`**

By default a command prints to your screen. You can send that output into a file instead.

*(Run from `~/bash-training`)*
```bash
ls -la > listing.txt
```
Nothing appears on screen — because the output went into `listing.txt`. Read it back:

*(Run from `~/bash-training`)*
```bash
cat listing.txt
```

Now the crucial distinction:
- `>` **overwrites** — it wipes the file and writes fresh
- `>>` **appends** — it adds to the end, keeping what was already there

*(Run from `~/bash-training`)*
```bash
echo "deployed" >> log.txt
echo "deployed again" >> log.txt
cat log.txt
```

Get those two the wrong way round and you'll destroy a file you meant to add to. This bites everyone once.

**Capturing a command's output into text with `$( )`**

Finally, **command substitution** — run a command and drop *its output* right into another line. Anything inside `$( )` runs first, and its output replaces the `$( )` before the outer line runs.

*(Run from `~/bash-training`)*
```bash
date
```
That prints the date and time on its own. Now watch it slot into a sentence:

*(Run from `~/bash-training`)*
```bash
echo "This backup ran at $(date)"
```

`$(date)` runs `date`, and its output gets substituted into the string. We'll use this constantly in scripts to timestamp things.

So that's the toolkit: `;`, `&&`, `||`, `|`, `>`, `>>`, and `$( )` — plus the small filter commands `sort`, `head`, `wc -l` and `grep`. Everything we write today is your existing commands, glued together with these.

<br>
<br>

### 09:45–10:15 — Hands-On: Chaining & Pipelines
*(Exercise — 30 minutes)*

Your turn. Work through these at your own pace; the goal is to get the operators into your fingers before we start writing scripts. Don't just copy-paste — type them, and **predict what each will do before you hit enter**.

**HANDS ON (30 min)** <br>

*(Run everything in this exercise from `~/bash-training`)*

Part A — sequencing and safety.
1. Run `mkdir demo && cd demo` — confirm you moved into it with `pwd`. → **you are now in `~/bash-training/demo`**
2. From *inside* `demo`, run `mkdir demo && cd demo` **again**. What happens, and why did `cd` not run the second time? Then try the same with `;` instead of `&&` and note the difference.
3. Create three empty files in one line: `touch a.txt b.txt c.txt`, then confirm with `ls`.

Part B — pipes.
4. Run `ls -la /etc | wc -l` — you've just counted how many entries are in `/etc`.
5. Run `ls /etc | sort | head -10` and then `ls /etc | sort -r | head -10`. What did `-r` change?
6. Run `history | grep cd` — you've searched your own command history for every time you used `cd`.
7. Run `cat /etc/passwd | grep "$USER"` — find your own user's line in the system's user file.

Part C — redirection and substitution.
8. Run `ls -la > my-listing.txt`, then `cat my-listing.txt`. You captured a listing into a file.
9. Run `echo "First line" > notes.txt` then `echo "Second line" >> notes.txt`, and `cat notes.txt`. Now run the *first* command again — what happened to "Second line", and why?
10. Run `echo "Report generated on $(date) by $(whoami)" > report.txt` and read it back. You've built a sentence out of two commands' output.

Part D (stretch).
11. In one line, count how many files in `/usr/bin` have names containing the word "python".
12. Build a pipeline that lists your files, sorts them by size, and shows only the largest few. (Hint: `ls -lS` sorts by size.)

**END OF NOTE**

**Solution**

*(Run from `~/bash-training`)*
```bash
# --- Part A ---
mkdir demo && cd demo
pwd                          # -> /home/<you>/bash-training/demo

mkdir demo && cd demo        # cd does NOT run: mkdir fails (folder exists), && stops the chain
mkdir demo ; cd demo         # cd DOES run: ';' ignores the failed mkdir and carries on
cd ~/bash-training/demo      # get back to a known place

touch a.txt b.txt c.txt
ls                           # -> a.txt  b.txt  c.txt

# --- Part B ---
ls -la /etc | wc -l
ls /etc | sort | head -10
ls /etc | sort -r | head -10 # -r reverses the sort: Z to A instead of A to Z
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

# --- Part D ---
ls /usr/bin | grep python | wc -l
ls -lS | head -5
```

**ASK** *(bring the room back together)* <br>
In step 9, running the first `echo` again wiped out "Second line". If that had been a real log file you'd been appending to all week, what one-character change would have saved you? <br>
**ANSWER** <br>
Using `>>` (append) instead of `>` (overwrite). This is one of the most common ways people accidentally destroy data in a shell — worth burning into muscle memory now.

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>

### 10:30–11:00 — Your First Script: Shebang, Permissions, Running It

Everything so far, we've typed one line at a time. That's fine for two or three commands. But real setup work is dozens of steps, run repeatedly, that need to behave *identically* every time. Typing them by hand invites typos, skipped steps, and the classic "well, it worked on my machine".

A **script** is the fix. It's just a plain text file containing the commands you'd otherwise type, run top to bottom. Because it's a file, you can save it, put it in Git, review it, share it, and re-run it exactly the same way a thousand times.

**Create the file**

*(Run from `~/bash-training`)*
- Run: `touch hello.sh`
- Then open it: `code hello.sh` (or `nano hello.sh`)

Put in:

```bash
#!/bin/bash
# hello.sh — my first script

echo "Hello, $USER!"
echo "Today is $(date)"
echo "You are currently in $(pwd)"
```

Two things to explain slowly, because they trip everyone up the first time.

**The shebang.** That first line, `#!/bin/bash`, is called a **shebang** (from "hash" `#` plus "bang" `!`). It tells the operating system *which program should interpret the rest of this file*. Here we're saying "run this using the bash program, which lives at `/bin/bash`". If we were writing Python we'd write `#!/usr/bin/env python3` instead. Without a shebang the system has to guess, and you can't reliably run the file as a standalone program. It **must** be the very first line — no blank lines, no spaces above it.

**Comments.** Any line starting with `#` (other than the shebang, which is special because of the `!` after it) is a **comment** — bash reads it and ignores it entirely. Comments are notes for humans, explaining *why* the code does something.

**Try to run it**

*(Run from `~/bash-training`)*
```bash
./hello.sh
```

You'll get **Permission denied**. That's expected, and it's a good thing.

**ASK** <br>
Why do you think Linux refuses to run a text file you just created as a program, by default? <br>
**ANSWER** <br>
It's a safety default. If every text file were automatically executable, then anything you downloaded — an email attachment, a file off the web — could run as a program the moment you touched it. Linux makes you *explicitly* mark a file as "yes, this is allowed to run" before it will execute it.

**Make it executable**

`chmod` is short for "**ch**ange **mod**e" — it changes a file's permissions. `+x` means "**add** the e**x**ecute permission". So this line says "allow this file to be run as a program":

*(Run from `~/bash-training`)*
```bash
chmod +x hello.sh
./hello.sh
```

It runs.

**Why the `./`?** When you type `hello.sh` on its own, bash searches a specific list of folders (the `PATH`, which we meet next) for a command by that name — and your *current* folder is not on that list. The `./` means "the file is right **here**, in this directory" (`.` is shorthand for "current directory", the same `.` you've seen in `docker build .`). We'll get rid of that clunky `./` in the next section.

**HANDS ON (10 min)** <br>
*(Run from `~/bash-training`)*
1. Create `hello.sh` as above, make it executable, and run it.
2. Add two more lines: one that prints how many files are in the current directory, and one that prints your machine's hostname.
3. Deliberately remove the execute permission with `chmod -x hello.sh` and try to run it, to see the error come back. Then restore it.
**END OF NOTE**

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
chmod +x hello.sh
./hello.sh

chmod -x hello.sh    # remove the execute permission
./hello.sh           # -> bash: ./hello.sh: Permission denied
chmod +x hello.sh    # put it back
```

<br>
<br>

### 11:00–11:30 — Giving a Script a Name, and Feeding It Input

Right now our script only runs if we're sitting in the same folder and prefix it with `./`. That's not how real commands work — you can type `git` or `docker` from *anywhere*. Let's fix that, then learn how to pass information *into* a script.

**Putting a script on the PATH**

When you type a bare command name, bash looks through a list of folders held in a variable called `PATH`. Have a look at yours:

*(Run from `~/bash-training`)*
```bash
echo $PATH
```

You'll see something like `/usr/local/bin:/usr/bin:/bin` — a list of folders separated by **colons**. When you type `git`, bash walks that list looking for a file called `git` and runs the first one it finds. To make *our* script callable by name, we put it in one of those folders — or add a new folder to the list.

The tidy convention is a personal `bin` folder in your home directory:

*(Run from `~/bash-training`)*
- Run: `mkdir -p ~/bin` — the `-p` means "make it, and don't complain if it already exists"
- Run: `mv hello.sh ~/bin/hello` — move the script in, **dropping the `.sh` extension**
- Run: `export PATH="$HOME/bin:$PATH"` — add `~/bin` to the **front** of the PATH list

```bash
mkdir -p ~/bin
mv hello.sh ~/bin/hello
export PATH="$HOME/bin:$PATH"
```

That last line reads as: "set `PATH` to be my new `~/bin` folder, then a colon, then everything `PATH` already was." Putting it at the front means bash checks `~/bin` first.

Now prove it works from somewhere completely different:

*(Run from `~/` — get there with `cd ~`)*
```bash
hello
```

No `./`, no `.sh`. It looks and behaves like a built-in command.

**NOTE FOR TRAINERS** <br>
`export PATH=...` only lasts for the current terminal session. To make it permanent, that line goes in `~/.bashrc` (or `~/.zshrc` on macOS), which runs every time a new shell opens. Show them this — add the line to `~/.bashrc`, run `source ~/.bashrc`, and demonstrate it surviving a brand new terminal window. This is exactly how tools like `nvm` and the Azure CLI hook themselves in. <br>
**END OF NOTE**

**ASK** <br>
We dropped the `.sh` from the filename when we moved it. Why doesn't that break anything — how does the system still know it's a bash script? <br>
**ANSWER** <br>
The extension is just part of the name; it means nothing to Linux. What actually determines how the file runs is the **shebang** (`#!/bin/bash`) on the first line. That's why real commands like `git` and `docker` have no extension at all.

**Passing input in: positional arguments**

A script that does the exact same thing every time is useful. A script you can *tell what to do* is far more useful. When you run a command with extra words after it — `mkdir project`, `docker build myimage` — those extra words are **arguments**, and inside a script you read them using special variables.

*(Run from `~/bash-training` — `cd ~/bash-training` first)*
- Run: `touch greet`
- Then open it: `code greet`

```bash
#!/bin/bash
# greet — say hello to whoever we're told to

echo "First argument:  $1"
echo "Second argument: $2"
echo "All arguments:   $@"
echo "Number of args:  $#"
```

The special variables, one at a time:

| Variable | Means |
|---|---|
| `$1` | the **first** argument |
| `$2` | the **second** argument (and `$3`, `$4`... likewise) |
| `$@` | **all** the arguments together |
| `$#` | **how many** arguments there were (a count) |
| `$0` | the **name of the script itself** |

*(Run from `~/bash-training`)*
```bash
chmod +x greet
./greet Alice Bob
```

So `$1` becomes `Alice`, `$2` becomes `Bob`, `$#` becomes `2`. This is exactly the mechanism we'll use later for `scaffold countries` — where `countries` is the first argument `$1`, and the script builds a whole MVC app named after it.

**Asking the user a question with `read`**

Sometimes you want to prompt interactively instead of taking an argument:

```bash
#!/bin/bash
read -p "What should we call the resource? " resource
echo "Great — we'll scaffold '$resource'"
```

`read -p` prints the prompt in quotes, waits for the user to type something and hit enter, then stores whatever they typed into the variable `resource`. You use it afterwards as `$resource`, just like any other variable.

**HANDS ON (10 min)** <br>
*(Run from `~/bash-training`)*
1. Create the `greet` script above, make it executable, move it to `~/bin/greet`, then run `greet DevOps Engineer` **from `~/`** to prove it works anywhere.
2. Modify it so it prints `Hello <first-arg>, nice to meet you`.
3. Write a tiny `ask` script using `read` that asks for a name and echoes it back.
**END OF NOTE**

**Solution**

`greet` (after step 2):
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

*(Run from `~/` — `cd ~` to prove it runs from anywhere)*
```bash
greet DevOps Engineer     # -> Hello DevOps, nice to meet you   ($1 is "DevOps", $2 is "Engineer")
ask
```

<br>
<br>

### 11:30–12:00 — Logic: Conditionals, Loops and Functions

We now have named scripts that take input. The last piece before we build something real is **logic** — making a script *decide* and *repeat*. This is what separates a script from a plain list of commands, and it's brand new territory, so I'm going slowly here.

**Conditionals — `if`**

An `if` statement lets a script take one path or another depending on whether something is true. The shape looks odd the first time, so let's name every single part:

```bash
if [ CONDITION ]; then
  # commands to run when the condition is TRUE
else
  # commands to run when the condition is FALSE
fi
```

- `if` — starts the statement
- `[ CONDITION ]` — the thing being tested
- `; then` — "...**then** do the following". `then` must be separated from the condition, which is why there's a `;` (you could also put `then` on its own new line)
- `else` — the "otherwise" branch. **Optional** — plenty of `if`s have no `else`
- `fi` — ends the `if`. It is literally "if" spelled backwards. Bash does this again with `case`/`esac`. Weird, but you'll get used to it

**The square brackets are actually a command.** This is the bit nobody tells beginners, and it explains the most common error you'll hit. `[` is not decoration — it's a command (an alias for `test`), and `]` is its final argument. That's **why the spaces matter so much**:

- `[ -z "$1" ]` ✅ works
- `[-z "$1"]` ❌ fails — bash can't see `[` as a separate command without the space

If a student's `if` is mysteriously broken, 90% of the time it's a missing space inside the brackets.

Inside the brackets you use **test operators**. These are the ones you'll actually use:

| Test | True when |
|---|---|
| `-z "$var"` | the variable is **empty** (`z` for "zero length") |
| `-n "$var"` | the variable is **not empty** |
| `-f path` | a **file** exists at that path |
| `-d path` | a **directory** exists at that path |
| `"$a" = "$b"` | two **strings** are equal |
| `"$a" != "$b"` | two **strings** are different |
| `$a -eq $b` | two **numbers** are equal |
| `$a -ne $b` | two numbers are **not** equal |
| `$a -lt $b` | first number is **less than** second |
| `$a -gt $b` | first number is **greater than** second |

Note that strings and numbers use *different* operators: `=` for text, `-eq` for numbers. That catches everyone once. And `!` in front of a test means "not" — `[ ! -f "Dockerfile" ]` reads "if a Dockerfile does **not** exist".

The single most common real-world use of `if` is to **guard against bad input** at the top of a script:

```bash
#!/bin/bash
if [ -z "$1" ]; then
  echo "Usage: scaffold <resource-name>" >&2
  exit 1
fi
echo "Scaffolding $1..."
```

Read it as: *if the first argument is empty*, print a usage message and stop. Two new pieces:

- `>&2` — send this output to **standard error** rather than normal output. Every program has two output channels: channel `1` is normal output ("stdout"), channel `2` is errors ("stderr"). `>&2` means "redirect to channel 2". Error messages belong on the error channel so that automated tools can tell your real results apart from your complaints.
- `exit 1` — stop the script immediately and report **failure** (unpacked below).

Another everyday `if` — don't overwrite something that already exists:

```bash
if [ -d "server" ]; then
  echo "server/ already exists, skipping"
else
  mkdir server
fi
```

*If a directory called `server` exists*, say so; *otherwise*, make it.

**Exit codes — how a script says "I succeeded" or "I failed"**

Every command, when it finishes, silently hands back a number called an **exit code**. `0` means success; **any other number means failure**. It's backwards from what you'd expect — zero is the good one. Think of it as "zero problems".

You can see the exit code of the last command with the special variable `$?`:

*(Run from `~/bash-training`)*
```bash
ls /nonexistent-folder
echo $?     # -> a non-zero number (e.g. 2). That command failed.
```

And with a command that works:

*(Run from `~/bash-training`)*
```bash
ls ~
echo $?     # -> 0. Success.
```

When *your own* script calls `exit 1`, you're deliberately telling whatever ran it "I failed".

**NOTE FOR TRAINERS** <br>
This is the single most important concept to land for later sessions. In the Jenkins/pipelines session, a pipeline decides whether a build **passes or fails** purely by looking at the exit code of the scripts it runs. A script that returns `0` even when something went wrong will make a broken build look green. "Fail loudly with a non-zero exit code" is a professional habit, not a nicety. <br>
**END OF NOTE**

**Loops — `for`**

A `for` loop repeats a block of commands once for each item in a list. Again, let's name every part:

```bash
for VARIABLE in LIST; do
  # commands, usually using "$VARIABLE"
done
```

- `for dir` — invent a variable called `dir` (you pick the name — no `$` when you declare it)
- `in controllers models routers db` — the list of items, separated by **spaces**
- `; do` — "...for each of those, **do** the following"
- `done` — ends the loop (the way `fi` ended the `if`)

On each pass through the loop, the variable holds the next item from the list. So this:

```bash
for dir in controllers models routers db; do
  mkdir -p "server/$dir"
done
```

runs `mkdir -p "server/controllers"`, then `mkdir -p "server/models"`, then `routers`, then `db`. One line of intent instead of four separate `mkdir` lines. (We wrap `"server/$dir"` in quotes as a good habit, so it still works if a name ever contained a space.)

You can also loop over files using a wildcard:

```bash
for file in *.js; do
  echo "Found JavaScript file: $file"
done
```

`*.js` expands to every file ending in `.js` in the current folder, and the loop visits each one in turn.

And you can loop over a range of numbers using `{1..5}` (brace expansion) or a C-style loop:

```bash
for i in {1..5}; do
  echo "Number $i"
done
```

**Functions — naming a block of steps**

A **function** lets you give a name to a group of commands, then call that name whenever you want to run them — write the steps once, reuse them everywhere. The shape:

```bash
name_of_function() {
  # commands go here
}
```

- `log()` — the name you're inventing, followed by **empty** `()`
- `{ ... }` — the block of commands that make up the function

Here's a genuinely useful one — a logger that stamps every message with the time:

```bash
log() {
  echo "[$(date +%H:%M:%S)] $1"
}

log "Starting up"
log "Creating folders"
```

Two things to notice. First, `date +%H:%M:%S` formats the date as just hours:minutes:seconds. Second — and this catches people — **inside a function, `$1` means the *function's* first argument, not the script's.** So `log "Starting up"` passes `"Starting up"` in as the function's `$1`. Define the format once; change it in one place forever after.

---

**Challenge**

*Direct* students, **in pairs**, to write a bash function that does the following:
* Accepts three arguments: `start`, `stop`, and `final`
* Counts down from the `start` number to the `stop` number, printing each number as it goes
* Instead of printing the `stop` number, prints the value of `final`
* *OPTIONAL* — extend it to handle `stop` being greater than `start`

*Provide* this example output for `countdown 10 5 "Blastoff!"` as an aid:

```
10
9
8
7
6
Blastoff!
```

*Grant* students ~8 minutes. Hints to offer if they're stuck: a countdown needs a loop that goes *down*, and `seq` can generate a descending list — `seq 10 -1 6` prints 10 down to 6.

**Solution**

```bash
#!/bin/bash

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

New pieces in that solution, worth calling out:
* `local start="$1"` — `local` keeps the variable **inside the function**, so it can't clash with anything outside it. Good habit.
* `$(( stop + 1 ))` — the double brackets are **arithmetic expansion**: bash does maths inside them. Bash won't do maths without being told to, so `stop + 1` on its own would just be text.
* `seq "$start" -1 $((stop + 1))` — generates the descending list of numbers, which the `for` loop then walks through.

**Solution (optional extension — handles counting up too)**

```bash
#!/bin/bash

countdown() {
  local start="$1"
  local stop="$2"
  local final="$3"

  if [ "$start" -gt "$stop" ]; then
    # Counting DOWN
    for i in $(seq "$start" -1 $((stop + 1))); do
      echo "$i"
    done
  else
    # Counting UP
    for i in $(seq "$start" 1 $((stop - 1))); do
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

One last technique, because we need it to build our app after the break. Sometimes a script needs to *write out the entire contents of a file*, not just create an empty one. The tool for that is a **heredoc** (short for "here document"):

```bash
cat > server/index.js << 'EOF'
const app = require("./app");
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Listening on port ${PORT}`));
EOF
```

Reading it piece by piece:
- `cat` — a command that simply outputs whatever it's given
- `> server/index.js` — redirect that output into this file (overwriting it)
- `<< 'EOF'` — "here comes a block of text; keep reading until you hit a line that is just `EOF`"
- everything between the two markers becomes the file's contents
- the closing `EOF`, on its own line with nothing else, ends the block

`EOF` is just a chosen marker word ("end of file") — you could use any word, as long as the opening and closing match.

**The quoting on that marker is important, and completely new to most people:**

| Written as | Behaviour | Use when |
|---|---|---|
| `<< 'EOF'` (single quotes) | Writes the text **literally**, leaving `$` signs exactly as typed | The file's contents contain their own `$` variables — like the JavaScript above, whose `${PORT}` belongs to **Node**, not bash |
| `<< EOF` (no quotes) | Bash **substitutes** its own variables into the text first | You *want* your script's variables (like `$resource`) filled in |

We'll use both in the scaffold after the break, and I'll flag which is which every time.

**HANDS ON (10 min)** <br>
*(Run from `~/bash-training`)*
1. Write a script `check` that takes one argument and prints "That file exists" if a file by that name exists (`-f`), and "No such file" otherwise. Test it against a file that exists and one that doesn't.
2. Add a guard clause at the top: if no argument is given, print a usage message to `>&2` and `exit 1`. Confirm `echo $?` shows `1` after running it with no argument.
3. Write a `for` loop that creates five files named `test1.txt` through `test5.txt`.
**END OF NOTE**

**Solution**

`check`:
```bash
#!/bin/bash
# check — report whether a named file exists

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

./check                 # -> prints the usage message (to stderr)
echo $?                 # -> 1, proving it failed on purpose

# Step 3 — the for loop
for i in 1 2 3 4 5; do
  touch "test$i.txt"
done
ls test*.txt            # -> test1.txt test2.txt test3.txt test4.txt test5.txt
```

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>
### 12:15–13:00 — Building an App Scaffold: the `countries` Script
*(Exercise — 45 minutes)*

Now we bring the whole morning together into something genuinely useful. Every time you start a new Express API, you make the same folders and the same starter files by hand. That's *exactly* the kind of repetitive, error-prone, "did I forget one?" task we automate. You already know Express and `package.json` well, so the *contents* of these files will be familiar — the new part is having a script write them for you.

We're going to write a script called `scaffold` that, given a resource name like `countries`, builds this entire structure in one command:

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

Notice the naming: the controller, router and SQL file are **plural and lowercase** (`countries`), but the model is **singular and capitalised** (`Country`). We'll handle that difference with arguments.

Build it alongside me, stage by stage. Create the file first:

*(Run from `~/bash-training`)*
- Run: `touch scaffold`
- Then open it: `code scaffold`

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

- **`set -e`** at the very top means "if **any** command in this script fails (returns a non-zero exit code), stop the whole script right there." Without it, a script keeps charging on after an error, potentially making a bigger mess. With it, we don't need to check the exit code after every single line by hand.

- **`resource="$1"`** copies the first argument into a nicely-named variable, so the rest of the script reads clearly (`$resource` rather than `$1` scattered everywhere). Note: **no spaces around the `=`** when assigning a variable in bash. `resource = "$1"` is an error — bash would think you're trying to run a command called `resource`.

- **`model="${2:-${resource^}}"`** is doing two clever things at once, so let's take it apart:
  - `${2:-something}` means "use the second argument `$2`, **but if it's missing or empty, use `something` instead**". The `:-` is the "default value" operator.
  - `${resource^}` takes the `resource` variable and **capitalises its first letter** — the `^` does that.
  - Put together: if the user gave a model name, use it; otherwise capitalise the resource name. So `scaffold countries` gives a model of `Countries`, while `scaffold countries Country` gives exactly `Country`.

**ASK** <br>
Why is it worth putting that `if [ -z "$resource" ]` guard at the very top, before we create a single folder? <br>
**ANSWER** <br>
If someone runs `scaffold` with no argument, `$resource` is empty, and every path we build (`server/models/.js`, files named after nothing) would be malformed. Failing fast with a clear usage message is far kinder than half-creating a broken mess of files and leaving the user to clean it up.

**Stage 2 — the top-level files**

Here we use heredocs. These are **unquoted** (`<< EOF`), because we *want* bash to substitute our `$resource` variable into the file contents:

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

*(Those `\`\`\`` inside the README heredoc are **escaped backticks** — the backslashes stop bash trying to treat them as a command to run. They come out as plain triple-backticks in the finished README file.)*

**Stage 3 — the server entry point and app wiring**

This is where the heredoc quoting difference matters, so point it out explicitly:

- `index.js` has **no bash variables**, but it *does* contain `${PORT}`, which belongs to **Node**, not bash. So we **quote** the marker (`<< 'EOF'`) to keep everything literal.
- `app.js` **does** need our `$resource` substituted in, so it stays **unquoted** (`<< EOF`).

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

All three are **unquoted** heredocs, because each needs `$resource` and/or `$model` filled in:

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

**Now use it**

First install the script so it's callable by name:

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
tree
```

If everything's right, `tree` shows the exact structure from the top of this section. You've just replaced ten minutes of typing with one command — and it's identical, every single time, for everyone on the team.

**HANDS ON (remaining time)** <br>
Build the `scaffold` script alongside me, stage by stage, then:

1. *(Run from `~/countries-app`)* Run `scaffold countries Country` and confirm the tree matches exactly.
2. *(Run from `~/`)* Run `mkdir books-app` → **cd inside**, then *(from `~/books-app`)* run `scaffold books Book`. Watch the same structure appear with different names — this is the payoff of using arguments instead of hard-coding.
3. *(Run from `~/books-app`)* `cat server/models/Book.js` and confirm the substitutions worked.
4. **(Stretch)** Add a guard near the top: if a `server/` folder already exists in the current directory, print a warning and `exit 1` rather than overwriting.
5. **(Stretch)** Add a final line that runs `git init` so every scaffolded app starts as a Git repo.
**END OF NOTE**

**Solution**

The complete assembled `scaffold` script, including both stretch goals (the `server/` guard and `git init`):

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
cat server/models/Book.js
```

**ASK** *(to close the morning)* <br>
We've hard-coded nothing about "countries" into this script — it's all driven by arguments. Why does that make it dramatically more valuable to a team than a script that just creates a fixed "countries" app? <br>
**ANSWER** <br>
Because now it's a *tool*, not a one-off. Anyone can scaffold any resource — `countries`, `books`, `users` — with a consistent, reviewed, agreed structure, and nobody has to remember the folder layout. Consistency across a team is one of the quiet superpowers of automation, and it's the exact same reason we'll write infrastructure as reusable Terraform modules later in the course.

<br>
<br>

*(Take a 1 hour lunch break here.)*

<br>
<br>

### 14:00–14:15 — From Scripting to Automation for DevOps

Welcome back. This morning was about *writing scripts*. This afternoon is about *automation as a way of working* — which is a bigger idea than any single script.

Here's the shift. A script is a tool. **Automation** is the discipline of making sure the important, repeatable work of running software happens the same way every time, without a human having to remember the steps or be awake to press the button.

Think back to the DevOps habits from the Azure session — "automate the boring, repeatable stuff", "small frequent changes", "everything as code". Scripts are how those habits actually get realised. That scaffold script you wrote means every new service in your team starts identically. Later, a build script means every image is built and pushed identically. A deploy script means every release happens identically. Once the steps live in a script, they can be:

- **Reviewed** — a teammate reads the script in a pull request before it ever runs
- **Version-controlled** — you can see exactly how the process changed over time, and roll it back
- **Triggered automatically** — either on a schedule, using **`cron`** (a scheduler built into every Linux system that runs commands at set times — "every night at 2am", "every 5 minutes", entirely unattended), or by an event like "someone pushed to `main`" (that's Jenkins, in a couple of sessions)
- **Run anywhere** — your laptop, a colleague's, a build server, a cloud shell — and behave the same

**ASK** <br>
We could keep doing all of this by hand and get the same *result* on any given day. So what does moving it into automated scripts actually buy a DevOps team — beyond just saving a bit of typing? <br>
**ANSWER** <br>
Consistency and safety at scale. Humans doing repetitive steps by hand drift — they skip a step, do them out of order, or do it slightly differently on one server. A script does it identically every time, leaves a reviewable record of *what* runs, fails loudly when something's wrong (exit codes), and can run without anyone present. The value isn't the saved keystrokes; it's that the process becomes reliable and auditable.

Now — bash is the **lingua franca** of Linux. That phrase just means "the common language everyone shares": whatever else a team uses, they can all fall back on bash. And it's most of what a DevOps engineer scripts. But it's not the only shell you'll meet, especially in a Microsoft-heavy environment like Azure. So for the next stretch we're going to step into **PowerShell**, right inside the Azure Cloud Shell, and see both how it's similar and how it's fundamentally different.

<br>
<br>
### 14:15–15:00 — PowerShell on the Azure Cloud Shell

You've been driving Azure two ways so far: clicking in the Portal, and typing `az` commands in bash. There's a third, and in a lot of Microsoft shops it's the primary one: **PowerShell**.

**Getting to it — the Cloud Shell**

You don't need to install anything. Azure has a browser-based terminal built right into the Portal, called the **Cloud Shell**.

- Sign in to `portal.azure.com`
- Click the **`>_`** icon in the top toolbar (Cloud Shell)
- If prompted, choose **PowerShell** (not Bash) — you can switch between the two with a dropdown at the top-left of the shell
- The first time, it'll ask to create a small storage account to persist your files — that's normal, let it

You now have a full PowerShell session, already authenticated as you, with the Azure `Az` module preinstalled. No login step, no CLI install — that's the appeal.

**The big conceptual difference: objects, not text**

This is the one thing to really understand about PowerShell, because it's genuinely different from bash.

In bash, everything flowing through a pipe is **text**. When you write `ls | grep countries`, you're passing a stream of characters, and `grep` is doing pattern-matching on strings. It works, but it's a bit crude — you're constantly slicing text apart to get at the bits you want.

In PowerShell, everything flowing through a pipe is a proper **object** — a structured thing with named properties. When you list files, each item isn't a line of text; it's a file object with a `.Name`, a `.Length`, a `.LastWriteTime`, and so on. You filter and sort on those *properties* directly, no text-wrangling required.

**Cmdlets — the Verb-Noun pattern**

PowerShell commands are called **cmdlets** (pronounced "command-lets"), and they follow a rigid, predictable naming pattern: **`Verb-Noun`**.

| Cmdlet | What it does | Bash equivalent |
|---|---|---|
| `Get-ChildItem` | List items in a location | `ls` |
| `Set-Location` | Change directory | `cd` |
| `Get-Content` | Read a file | `cat` |
| `New-Item` | Create a file or folder | `touch` / `mkdir` |
| `Where-Object` | Filter to matching items | `grep` |
| `Get-Help` | Show docs for a cmdlet | `man` |
| `Get-Command` | List available commands | `which` / `compgen` |

Because the verbs are standardised (`Get`, `Set`, `New`, `Remove`, `Start`, `Stop`...), you can often *guess* a cmdlet's name. Want to remove something? `Remove-Item`. Want to start a service? `Start-Service`. That predictability is deliberate — and it's the opposite of bash's terse, inconsistent names (`ls`, `cat`, `grep`, `rm`), which you simply have to memorise.

Helpfully, PowerShell also ships **aliases** so muscle memory from bash mostly works: `ls`, `cd`, `cat`, `pwd`, `cp`, `rm` are all aliased to the equivalent cmdlets. Great for getting started — but write the **full cmdlet names in scripts**, because aliases can differ between systems.

**Filtering and sorting — the pipeline in action**

Here's where objects shine. To find the three largest files in a folder:

*(Run in the Azure Cloud Shell, in **PowerShell** mode)*
```powershell
Get-ChildItem | Sort-Object Length -Descending | Select-Object -First 3
```

Read it: list the items, sort them by their `Length` property (biggest first), keep the first three. No `wc`, no `head`, no text parsing — you sorted on a real numeric property.

To filter:

*(Run in the Azure Cloud Shell, in **PowerShell** mode)*
```powershell
Get-ChildItem | Where-Object { $_.Name -like "*.json" }
```

Unpacking that:
- `Where-Object` keeps only items where the condition inside `{ }` is true — it's PowerShell's `grep`
- `$_` means "**the current object** flowing down the pipeline" — the item being examined right now
- `-like "*.json"` does wildcard matching, the way `*` works in bash

Compare that to piping through `grep` in bash — here you're matching on the object's actual `.Name` property, not on raw text.

**Variables**

Same idea as bash, slightly different punctuation — variables start with `$`, and (**unlike bash**) you *do* put spaces around `=`:

*(Run in the Azure Cloud Shell, in **PowerShell** mode)*
```powershell
$name = "countries"
Write-Output "Scaffolding $name"
```

**Talking to Azure — the `Az` module**

This is why PowerShell matters for you specifically. The `Az` module gives you cmdlets for every Azure resource, and in the Cloud Shell you're already logged in. Compare these to the `az` CLI commands you already know:

| Task | Azure CLI (bash) | PowerShell (`Az` module) |
|---|---|---|
| Show current subscription | `az account show` | `Get-AzContext` |
| List resource groups | `az group list -o table` | `Get-AzResourceGroup` |
| Create a resource group | `az group create --name rg1 --location uksouth` | `New-AzResourceGroup -Name rg1 -Location uksouth` |
| List all resources | `az resource list -o table` | `Get-AzResource` |
| Delete a resource group | `az group delete --name rg1` | `Remove-AzResourceGroup -Name rg1` |

Both are doing the *same thing* under the hood — both are polite wrappers around the same **Azure REST API**. Which you use is largely down to your team and what the rest of your tooling is written in. Microsoft-centric shops lean PowerShell; Linux-centric and cross-cloud shops lean the CLI. Knowing both makes you flexible.

**ASK** <br>
Both `az group list` and `Get-AzResourceGroup` end up hitting the same Azure REST API and give you the same resource groups. So is learning PowerShell as well as the CLI a waste of time? <br>
**ANSWER** <br>
No — because you don't get to choose the environment you walk into. Plenty of enterprises, especially Microsoft-heavy ones, have years of existing automation written in PowerShell, use Azure Automation runbooks (which are PowerShell), and expect you to work in it. And the object pipeline genuinely makes some tasks cleaner. Being fluent in both means you're useful whatever the team already runs.

**HANDS ON (25 min)** <br>
*(Run everything in this exercise in the **Azure Cloud Shell**, switched to **PowerShell** mode)*

Part A — get your bearings.
1. Run `Get-ChildItem` and then its alias `ls`. Confirm they do the same thing.
2. Run `Get-Command -Verb Get | Select-Object -First 20` to see how many "Get" cmdlets exist.
3. Run `Get-Help Get-AzResourceGroup` to see the built-in documentation for a cmdlet.

Part B — objects, not text.
4. Run `Get-ChildItem | Sort-Object Length -Descending | Select-Object Name, Length -First 5`. Notice you selected specific *properties* by name.
5. Run `Get-Process | Where-Object { $_.CPU -gt 1 } | Select-Object Name, CPU` — you've filtered live processes on a numeric property.

Part C — Azure with the Az module.
6. `Get-AzContext` — confirm which subscription you're in.
7. `Get-AzResourceGroup` — list your resource groups as objects.
8. Create one: `New-AzResourceGroup -Name ps-demo-rg -Location uksouth`
9. Confirm it exists, then filter for it with `Where-Object`.
10. Clean it up with `Remove-AzResourceGroup`.

Part D (stretch).
11. Store a resource group in a variable and inspect its properties with `Get-Member` — this lists every property and method the object has. It's how you *discover* what you can do with something in PowerShell.
**END OF NOTE**

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

# --- Part D (stretch) ---
$rg = Get-AzResourceGroup -Name ps-demo-rg
$rg | Get-Member
```

**NOTE FOR TRAINERS** <br>
If some students are on a fresh Azure account with no resource groups, steps 7–9 still work — the list will just start empty and then show `ps-demo-rg`. Emphasise `Get-Member` in the stretch: "when you don't know what an object can do, pipe it to `Get-Member`" is the single most useful habit for learning PowerShell independently. <br>
**END OF NOTE**

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>

### 15:15–17:00 — Capstone: Scaffold an App & Automate a Container Build
*(Exercise — 1 hour 45 minutes)*

This is the main event, and it deliberately pulls together everything you know: this morning's **bash scripting**, your existing **Docker** skills, and your **Azure** knowledge. You'll build **two real, named scripts** that a DevOps engineer would genuinely keep in their toolkit.

The scenario: your team spins up small Express services constantly, and every one needs to be scaffolded consistently *and* containerised and pushed to a registry the same way every time. You're going to automate both halves.

Work individually or in pairs. Aim to get Scripts 1 and 2 fully working; the stretch goals are there if you fly through. There's a lot here on purpose — it's meant to fill the session.

---

#### Part 1 (≈40 min) — Script one: `new-app`, the scaffolder

Take your morning `scaffold` script and harden it into a proper tool called `new-app`.

**Requirements:**
1. Takes a resource name as `$1` and an optional model name as `$2` (defaulting to a capitalised resource), exactly as this morning.
2. **Validates input**: no resource name → print a clear usage message to `>&2` and `exit 1`.
3. **Refuses to overwrite**: if a `server/` directory already exists in the current folder, warn and `exit 1`.
4. Creates the full MVC structure from this morning.
5. **Also** writes a working `Dockerfile` into the project root (so Part 2 has something to build):

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

6. **Also** writes a `.dockerignore` containing `node_modules` and `.git`.
7. Uses a `log()` function so every step prints a tidy, **timestamped** message.
8. Ends by printing a clear "next steps" message.
9. Install it to `~/bin/new-app` so it's callable by name from anywhere.

**Test it:**

*(Run from `~/`)*
- Run: `mkdir countries-app` → **cd inside**

*(Run from `~/countries-app`)*
```bash
new-app countries Country
tree
```

**Solution**

`new-app` — the morning's `scaffold`, plus a `Dockerfile`, a `.dockerignore`, timestamped logging and a next-steps message:

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

# 7. Timestamped logger
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

Installing and running it:

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
```

---

#### Part 2 (≈45 min) — Script two: `containerise`, the build-and-push

Now the automation that turns that scaffolded app into a published image. Write a script called `containerise`.

**Requirements:**
1. **Usage:** `containerise <image-name> [tag]`, where tag defaults to `latest`.
2. **Validate input:** no image name → usage message to `>&2` and `exit 1`.
3. **Pre-flight checks** — fail early and clearly if the environment isn't ready:
   - If there's no `Dockerfile` in the current directory (`-f`), print an error and `exit 1`.
   - If `docker` isn't available (hint: `command -v docker`), print an error and `exit 1`.
4. **Build** the image, tagging it with the name and tag.
5. **Check the build's exit code.** If the build failed, log a clear failure message to `>&2` and `exit 1`. (This is where exit codes stop being theoretical — a broken build **must** make the script fail.)
6. **Report success** with the full image name and tag.
7. Use the same timestamped `log()` helper as Part 1.
8. Install it to `~/bin/containerise`.

**Test it:**

*(Run from `~/countries-app`)*
```bash
containerise countries-api v1
docker images
```

**Then — the push (Azure tie-in).** Extend the script so that if a registry name is supplied as a **third** argument, it also **pushes to Azure Container Registry**.

**Solution**

`containerise` — full script including the optional ACR push:

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

Three bits of new syntax in there worth explaining to the room:

- **`if [ ! -f "Dockerfile" ]`** — the `!` means "not", so this reads "if a Dockerfile does **not** exist".
- **`if ! command -v docker > /dev/null`** — `command -v docker` looks up whether `docker` exists and prints its path. `> /dev/null` throws that printout away (we only care *whether* it succeeded, not what it said — `/dev/null` is a special "black hole" file that discards anything written to it). The leading `!` flips the result, so the block runs when docker is **absent**.
- **`|| { echo "..." >&2; exit 1; }`** — the `||` from this morning: "if the command on the left failed, run this block instead". The `{ }` groups multiple commands together. A compact way to bail out with a message.

Installing and running it:

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

**NOTE FOR TRAINERS** <br>
Provisioning an ACR per student may not be practical. Sensible options, in order of preference: (a) provide **one shared ACR** for the room and have students push under distinct tags/image names; (b) let students create a Basic-tier ACR in their own subscription (`az acr create --name <unique> --resource-group <rg> --sku Basic`) and tear it down after; or (c) if registries aren't available at all, have everyone stop after the local `docker build` step and just *write* the push logic without running it — the scripting practice is the real objective. Note that Docker must run on the student's Linux/VM environment, **not** the Azure Cloud Shell, which has no Docker daemon. <br>
**END OF NOTE**

**ASK** *(mid-exercise checkpoint, ~15:55)* <br>
In Part 2 we deliberately check the exit code of `docker build` and `exit 1` if it failed. Imagine this script is later run automatically by a Jenkins pipeline. What goes wrong if we *skip* that check and always let the script exit `0`? <br>
**ANSWER** <br>
The pipeline would think the build succeeded even when it didn't, and happily carry on to the next stage — deploying, tagging as "latest", telling everyone it's green — while shipping a broken or stale image. Honest exit codes are the contract between your script and every automated system that will ever run it.

---

#### Part 3 (≈20 min) — Stretch: chain them and add polish

If you've got both scripts working, level them up:

1. **A one-command workflow.** Write a third tiny script, `ship`, that runs `new-app` and then `containerise` in sequence using `&&`. Think about what should happen if the scaffold step fails — should the build still run?
2. **Logging to a file.** Make `containerise` write its output to a dated log file as well as the screen, the way an unattended automated job would need.
3. **A confirmation prompt.** Before pushing to a registry, use `read -p` to ask "Push to `<registry>`? (y/n)" and only push if the answer is `y`. *(Where might an interactive prompt like this be a problem in an automated pipeline? Discuss.)*
4. **Idempotency.** Make `new-app` safe to re-run: instead of failing outright when `server/` exists, have it skip files that already exist and create only the missing ones.

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

**2. Logging to a file** — the simplest version is to redirect at call time. `>>` appends, and `2>&1` folds the error channel into the same file so failures are captured too:

*(Run from `~/countries-app`)*
```bash
containerise countries-api v1 >> build-$(date +%F).log 2>&1
```

Or bake it into the script itself, just after the `log()` definition. `tee -a` writes to the screen **and** appends to the file at the same time:

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

*(Discussion point: an interactive prompt like this would **hang forever** in an automated pipeline, because there's no human there to type `y`. In real automation you'd drive this with a command-line flag or an environment variable instead of a prompt — a good example of a habit that's helpful by hand and harmful in automation.)*

**4. Idempotent `new-app`** — swap the hard failure for a per-file skip. The pattern is: check before each write, and only create what's missing:

```bash
# a small helper, defined near log()
write_if_missing() {
  # $1 = target path
  if [ -f "$1" ]; then
    log "$1 already exists, skipping"
    return 1     # non-zero = "skip this one"
  fi
  return 0        # zero = "go ahead and write it"
}

# then guard each file:
if write_if_missing package.json; then
cat > package.json << EOF
...
EOF
fi
```

*(In practice this is fiddly to retrofit onto every heredoc. A common real-world answer is to generate into a temporary folder and copy across only the files that don't already exist. Fine to discuss rather than fully implement in the time.)*

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

And be ready to explain **one place** in your scripts where you used an **exit code** to fail safely, and why it matters.

<br>
<br>

### 16:45–17:00 — Wrap-up & Q&A

Let's pull the day together.

This morning you went from gluing two commands together with `&&`, to writing a named script that scaffolds an entire application's structure from a single word of input. That's a genuine leap — you've written a real tool.

This afternoon you saw that automation is bigger than any one script: it's the discipline that makes repetitive work consistent, reviewable, and safe to run without a human watching. You met PowerShell and the Azure Cloud Shell, and saw that the object pipeline is a genuinely different way of thinking from bash's text streams. And in the capstone you connected all three worlds you now know — bash, Docker, and Azure — into two scripts that scaffold and containerise an app the same way, every time.

**ASK** <br>
Everything you built today runs when *you* type it. What's the missing piece that would make these scripts run **automatically** — say, every time someone pushes code — without anyone typing anything at all? <br>
**ANSWER** <br>
A pipeline. Something that watches for an event (a Git push) and runs your scripts for you, checking their exit codes to decide pass or fail. That's exactly the **Jenkins & pipelines** session coming up — and the reason we hammered exit codes today is that they're the language pipelines speak.

Where this sits in the bigger course:
- **Today** — you can script, name, and run automation by hand, and you understand why that matters
- **Jenkins & pipelines** — those scripts get triggered automatically on every code change
- **Terraform & IaC** — the same "everything as code" habit, applied to the Azure infrastructure itself
- **Kubernetes** — running and scaling the containers you learned to build and push today
- **Integration** — all of it woven into one pipeline: a code change flowing through to running infrastructure, automatically and safely

**Q&A** — take remaining questions.

<br>
<br>

### Exercise (take-home / reinforcement)

Before the next session, individually or in pairs:

1. Rebuild your `new-app` script **from memory**, without looking at today's notes, and get it scaffolding correctly. Rebuilding from scratch is the fastest way to make it stick.
2. Extend `new-app` so it accepts **multiple** resource names at once — e.g. `new-app countries books users` — creating a controller, router, model and SQL file for each. *(Hint: loop over `$@`.)*
3. Add a `--help` flag to both scripts: if the first argument is `--help` or `-h`, print usage and exit `0`.
4. Write a small `df-check` script that logs disk usage with a timestamp to a file, and schedule it with `cron` to run every 5 minutes for testing.
5. In PowerShell in the Cloud Shell, write a one-liner that lists all your resource groups and outputs just their names and locations, sorted by location.
6. **(Stretch)** Rewrite your `containerise` success message so it also prints the image's size on success.
7. **(Stretch)** Research and write down: what's the difference between `set -e`, `set -u`, and `set -o pipefail`, and why do experienced engineers often put `set -euo pipefail` at the top of a serious bash script?

**Solution** *(for the guided ones — 2, 3, 5)*

**Take-home 2** — looping over every argument with `$@`:
```bash
#!/bin/bash
# new-app (multi) — scaffold one resource per argument
set -e

if [ -z "$1" ]; then
  echo "Usage: new-app <resource> [resource2 ...]" >&2
  exit 1
fi

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

**Take-home 3** — a `--help` flag (this block goes right after the shebang, *before* the input guard):
```bash
if [ "$1" = "--help" ] || [ "$1" = "-h" ]; then
  echo "Usage: new-app <resource> [ModelName]"
  echo "  Scaffolds an Express MVC app in the current folder."
  exit 0
fi
```

**Take-home 5** — the PowerShell one-liner:
```powershell
Get-AzResourceGroup | Sort-Object Location | Select-Object ResourceGroupName, Location
```

<br>
<br>

### Conclusion

- **Inform** students this marks the end of the Bash Scripting & Automation session
- **Tell** students the next session moves into **Jenkins & pipelines**, where the scripts they wrote today start running automatically on every code change — and where today's obsession with exit codes suddenly pays off
- **Direct** students to the take-home exercises, and to the [Bash reference](https://www.gnu.org/software/bash/manual/bash.html) and [Azure PowerShell docs](https://learn.microsoft.com/en-us/powershell/azure/) for further reading

---

[Back](./README.md)

---
