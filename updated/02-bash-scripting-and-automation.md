# Bash Scripting & Automation for DevOps — Trainer Script

A full day taking students from "I can move around the filesystem" to "I can write, name, and run real scripts that scaffold an application and automate a container build". This is written as a script — read it out, adapt where it feels natural, but the intent is that you could deliver this cold from the page.

## Organisation

### Audience

Students who have already covered **Azure cloud fundamentals** and **Docker**, and who are comfortable with a handful of navigation commands (`cd`, `ls`, `mkdir`) but have **not** written a shell script before. By the end of the morning every student should be able to write a script, give it a name, run it like a real command, and have it scaffold a full Express MVC application. The afternoon shifts from *writing scripts* to *automation as a DevOps discipline*, including PowerShell in the Azure Cloud Shell, and finishes with a substantial two-hour capstone.

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
| 11:30–12:00 | Logic: conditionals, loops and functions | Talk + demo + short exercise |
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

You already know `cd`, `ls`, and `mkdir` — so we're not starting from zero. We're starting from "I can walk around the filesystem" and ending at "I can automate real work". Lunch is at 1 for an hour; we'll break mid-morning and mid-afternoon, and shout if you need one sooner.

<br>
<br>

### 09:15–09:45 — The Shell, and Chaining Commands Together

Let's start with a quick reframe of what the shell actually *is*, because it changes how you think about everything else today.

When you type `ls` and hit enter, you're not talking to some magic box — you're talking to a program called a **shell**, and on almost every Linux server and container that shell is **bash**. Bash reads what you type, works out which command you meant, runs it, shows you the output, and waits for the next line. That's the whole loop.

The powerful bit — the bit we're going to lean on all day — is that bash lets you **combine** commands. You already know the commands individually. Today is mostly about the operators *between* them.

Let me walk through the ones that matter.

**Running things in sequence with `;`**

A semicolon just means "run this, then run that, regardless of what happened":

```bash
mkdir project ; cd project ; ls
```

Three commands, one line, run in order. The catch: the semicolon doesn't care if a command *failed*. If `mkdir project` fails because the folder already exists, it'll still try to `cd` and `ls`.

**Running the next thing only if the last one worked, with `&&`**

This is the one you'll use constantly:

```bash
mkdir project && cd project
```

The `&&` means "only run `cd project` **if** `mkdir project` succeeded." If the folder couldn't be made, we don't blunder on into it.

```bash
sudo apt update && sudo apt install -y git
```

We only bother installing if the update step worked.

**Running something only if the last thing *failed*, with `||`**

The mirror image:

```bash
cd project || echo "Couldn't enter project folder"
```

"Try to `cd`; if that fails, print the warning." You'll often see `&&` and `||` used together to make a tiny inline if/else.

```bash
cd project && echo "It worked" || echo "Couldn't enter project folder"
```

**ASK** <br>
What's the practical difference between `mkdir app ; cd app` and `mkdir app && cd app`? Why might the second one save you from a nasty mistake? <br>
**ANSWER** <br>
With `;`, if `mkdir app` fails (say the folder already exists, or you don't have permission), bash still runs `cd app`. With `&&`, a failed `mkdir` stops the chain, so you don't accidentally carry on operating in the wrong directory. In a script that then deletes or overwrites files, that difference can be the difference between a clean run and wiping the wrong folder.

**Piping output from one command into another with `|`**

This is the idea that makes the shell feel like a superpower. A **pipe** takes the output of the command on the left and feeds it in as the input of the command on the right:

```bash
ls /etc | sort | head -5
```

`ls /etc` produces a list (a list of configuration files and directories used by the OS), `sort` alphabetises it, `head -5` keeps the first five. Each command does one small job; the pipe stitches them into a pipeline. Another:

**ASK** <br>
What do you think this command does? Feel free to Google <br>
```bash
cat access.log | grep "404" | wc -l
```

**ANSWER**<br>
"Read the log, keep only the lines containing 404, then count how many lines that is." You've just counted your 404 errors without opening the file.

**Redirecting output into a file with `>` and `>>`**

By default a command prints to your screen. You can send that output to a file instead:

```bash
ls -la > listing.txt      # create/overwrite listing.txt with the output
echo "deployed" >> log.txt # append one line to log.txt, keeping what's there
```

The distinction matters: `>` **overwrites**, `>>` **appends**. Get them the wrong way round and you'll clobber a file you meant to add to.

**Capturing a command's output into text with `$( )`**

Finally, "command substitution" — run a command and drop its output right into another line:

```bash
date
```

This does what you'd expect

```bash
echo "This backup ran at $(date)"
```

`$(date)` runs `date`, and its output gets slotted into the sentence. We'll use this constantly in scripts to timestamp things.


So that's the toolkit: `;`, `&&`, `||`, `|`, `>`, `>>`, and `$( )`. Six operators. Everything we write today is your existing commands, glued together with these.

<br>
<br>

### 09:45–10:15 — Hands-On: Chaining & Pipelines
*(Exercise — 30 minutes)*

Your turn. Work through these at your own pace; the goal is to get the operators into your fingers before we start writing scripts. Don't just copy-paste — type them, and predict what each will do *before* you hit enter.

**HANDS ON (30 min)** <br>

Part A — sequencing and safety.
1. In your home directory, run `mkdir demo && cd demo` — confirm you moved into it with `pwd`.
2. Now run `mkdir demo && cd demo` **again** from inside. What happens, and why did `cd` not run the second time? Try the same with `;` instead of `&&` and note the difference.
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
11. In one line, count how many files in `/usr/bin` have names containing the word "python": `ls /usr/bin | grep python | wc -l`.
12. Build a pipeline that lists your files, sorts them by size, and shows only the largest few. (Hint: `ls -lS` sorts by size.)

**END OF NOTE**

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

Everything we've done so far, we've typed one line at a time. That's fine for two or three commands. But real setup work is dozens of steps, run repeatedly, that need to behave *identically* every time. Typing them by hand invites typos, skipped steps, and the classic "well, it worked on my machine".

A **script** is the fix. It's just a plain text file containing the commands you'd otherwise type, run top to bottom. Because it's a file, you can save it, version-control it in Git, review it, share it, and re-run it exactly the same way a thousand times.

Let's write one.

**Create the file**

```bash
touch hello.sh
```

Open it in your editor (`code hello.sh` or `nano hello.sh`) and put in:

```bash
#!/bin/bash
# hello.sh — my first script

echo "Hello, $USER!"
echo "Today is $(date)"
echo "You are currently in $(pwd)"
```

Two things to explain here.

**The shebang.** That first line, `#!/bin/bash`, is called a **shebang** (hash-bang). It tells the operating system *which program should interpret the rest of this file*. Here we're saying "run this with bash". If we were writing Python we'd say `#!/usr/bin/env python3`. Without it, the system has to guess, and you can't reliably run the file as a standalone program. It **must** be the very first line — no blank lines above it.

**Comments.** Any line starting with `#` (other than the shebang) is a **comment** — bash ignores it. It's there for humans. 

**Try to run it**

```bash
./hello.sh
```

You'll almost certainly get **Permission denied**. That's expected, and it's a good thing.

**ASK** <br>
Why do you think Linux refuses to run a text file you just created as a program, by default? <br>
**ANSWER** <br>
It's a safety default. If every text file were automatically executable, then anything you downloaded — an email attachment, a file off the web — could run as a program the moment you touched it. Linux makes you *explicitly* mark a file as "yes, this is allowed to run" before it will execute it.

**Make it executable**


```bash
chmod +x hello.sh
```

Now:

```bash
./hello.sh
```

It runs.

**Why the `./`?** When you type `hello.sh` on its own, bash searches a specific list of folders (the `PATH`) for a command by that name — and your current folder isn't on that list. The `./` means "look right *here*, in this directory". We'll get rid of that clunky `./` in the next section by giving the script a real home.

**HANDS ON (10 min)** <br>
1. Create `hello.sh` as above, make it executable, and run it.
2. Add two more lines: one that prints how many files are in the current directory (`echo "There are $(ls | wc -l) files here"`), and one that prints your machine's hostname (`echo "Running on $(hostname)"`).
3. Deliberately remove the execute permission again with `chmod -x hello.sh` and try to run it, to see the error return. Then put it back.
**END OF NOTE**

<br>
<br>

### 11:00–11:30 — Giving a Script a Name, and Feeding It Input

Right now our script only runs if we're sitting in the same folder and prefix it with `./`. That's not how real commands work — you can type `git` or `docker` or `ls` from *anywhere*. Let's make our scripts behave the same way, and then learn how to pass information *into* them.

**Putting a script on the PATH**

When you type a bare command name, bash looks through the folders listed in a variable called `PATH`. Have a look at yours:

```bash
echo $PATH
```

You'll see something like `/usr/local/bin:/usr/bin:/bin` — a colon-separated list of folders. To make our script callable by name, we put it in one of those folders (or add a new folder to the list).

The tidy convention is a personal `bin` folder in your home directory:

```bash
mkdir -p ~/bin
mv hello.sh ~/bin/hello        # note: we drop the .sh extension
export PATH="$HOME/bin:$PATH"  # add ~/bin to the front of the PATH
```

Now, from **any** directory:

```bash
hello
```

It just works — no `./`, no `.sh`. It looks and behaves like a built-in command.

**NOTE FOR TRAINERS** <br>
`export PATH=...` only lasts for the current terminal session. To make it permanent, that line goes in `~/.bashrc` (or `~/.zshrc` on macOS), which runs every time a new shell opens. Show them this — add the line to `~/.bashrc`, run `source ~/.bashrc`, and demonstrate it surviving a new terminal. This is exactly how tools like `nvm`, the Azure CLI, and others hook themselves in. <br>
**END OF NOTE**

**ASK** <br>
We dropped the `.sh` from the filename when we moved it. Why doesn't that break anything — how does the system still know it's a bash script? <br>
**ANSWER** <br>
The extension is just part of the name; it means nothing to Linux. What actually determines how the file runs is the **shebang** (`#!/bin/bash`) on the first line. That's why real commands like `git` have no extension at all.

**Passing input in: positional arguments**

A script that does the exact same thing every time is useful. A script you can *tell what to do* is far more useful. When you run a command with extra words after it — `mkdir project`, `docker build myimage` — those words are **arguments**, and inside a script you can read them.

Make a new script, `greet`:

```bash
#!/bin/bash
# greet — say hello to whoever we're told to

echo "First argument:  $1"
echo "Second argument: $2"
echo "All arguments:   $@"
echo "Number of args:  $#"
```

Run it:

```bash
./greet Alice Bob
```

- `$1` is the first argument (`Alice`), `$2` the second (`Bob`)
- `$@` is *all* the arguments
- `$#` is *how many* arguments there were
- (`$0` is the name of the script itself)

This is the mechanism we'll use in a moment to write `scaffold countries` — where `countries` is our first arguemtn`$1` and scaffold builds the template of a MVC.

**Asking the user a question with `read`**

Sometimes you want to prompt interactively instead:

```bash
#!/bin/bash
read -p "What should we call the resource? " resource
echo "Great — we'll scaffold '$resource'"
```

`read -p` prints a prompt and stores whatever the user types into the variable `resource`.

**HANDS ON (10 min)** <br>
1. Create the `greet` script above, make it executable, move it to `~/bin/greet`, and run `greet DevOps Engineer` from a different folder.
2. Modify it so it prints `Hello <first-arg>, nice to meet you` — and if you pass no arguments, it just prints the raw empty `$1`. (We'll handle that "no argument" case properly in the next section.)
3. Write a tiny `ask` script using `read` that asks for a name and echoes it back.
**END OF NOTE**

<br>
<br>

### 11:30–12:00 — Logic: Conditionals, Loops and Functions

We now have named scripts that take input. The last piece before we build something real is **logic** — making a script *decide* and *repeat*. This is what separates a script from a plain list of commands.

**Conditionals — `if`**

The shape:

```bash
if [ CONDITION ]; then
  # do this
else
  # otherwise do this
fi
```

The most useful part is the tests you can put in the brackets:

| Test | True when |
|---|---|
| `-z "$var"` | the variable is **empty** |
| `-n "$var"` | the variable is **not empty** |
| `-f path` | a **file** exists at that path |
| `-d path` | a **directory** exists at that path |
| `"$a" = "$b"` | two strings are equal |
| `"$a" != "$b"` | two strings differ |
| `$a -eq $b` | two numbers are equal (`-ne`, `-lt`, `-gt` also) |

The single most common real-world use: **guard against bad input**. Here's the pattern you'll use in every serious script:

```bash
#!/bin/bash
if [ -z "$1" ]; then
  echo "Usage: scaffold <resource-name>" >&2
  exit 1
fi
echo "Scaffolding $1..."
```

If no argument was given (`$1` is empty), print a usage message **to standard error** (`>&2`) and `exit 1`. That `exit 1` matters enormously — we'll come back to it.

Another everyday use — don't clobber something that already exists:

```bash
if [ -d "server" ]; then
  echo "server/ already exists, skipping"
else
  mkdir server
fi
```

**Exit codes — how a script says "I succeeded" or "I failed"**

Every command, when it finishes, hands back a number called an **exit code**. `0` means success; anything else means some kind of failure. You can see the last one:

```bash
ls /nonexistent-folder
echo $?     # prints a non-zero number — that command failed
```

*Find a file which exists*

```bash
cat <file-name>
echo $?
```

When *your* script calls `exit 1`, you're telling whatever ran it "I failed". This is not academic:

**NOTE FOR TRAINERS** <br>
This is the single most important concept to land for later sessions. In the Jenkins/pipelines session, a pipeline decides whether a build **passes or fails** purely by looking at the exit code of the scripts it runs. A script that returns `0` even when something went wrong will make a broken build look green. "Fail loudly with a non-zero exit code" is a professional habit, not a nicety. <br>
**END OF NOTE**

**Loops — `for`**

Repeat an action over a list:

```bash
for dir in controllers models routers db; do
  mkdir -p "server/$dir"
done
```

One line of intent — "make these four folders under server/" — instead of four `mkdir` lines. You can loop over files too:

```bash
for file in *.js; do
  echo "Found JavaScript file: $file"
done
```


**Functions — naming a block of steps**

When a chunk of a script gets reused, wrap it in a **function**:

```bash
log() {
  echo "[$(date +%H:%M:%S)] $1"
}

log "Starting up"
log "Creating folders"
```

We define `log` once, then call it like a mini-command, passing it an argument (`$1` inside the function). Every message now gets a neat timestamp, and if we ever want to change the format we change it in one place.

**Writing whole files from a script — the heredoc**

One last technique, because we need it to build our app in a moment. Sometimes a script needs to *write out the entire contents of a file*, not just make an empty one. The tool for that is a **heredoc**:

```bash
cat > server/index.js << 'EOF'
const app = require("./app");
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Listening on port ${PORT}`));
EOF
```

Read it as: "take everything between `<< 'EOF'` and the closing `EOF`, and write it into `server/index.js`." The word `EOF` is just a marker — you could use any word. Quoting it as `'EOF'` tells bash *not* to try and interpret `$` signs inside the block, which matters when the file contents contain their own variables (like that JavaScript above).

**HANDS ON (10 min)** <br>
1. Write a script `check` that takes one argument and prints "That file exists" if a file by that name exists (`-f`), and "No such file" otherwise. Test it against a file that exists and one that doesn't.
2. Add a guard clause at the top: if no argument is given, print a usage message to `>&2` and `exit 1`. Confirm `echo $?` shows `1` after running it with no argument.
3. Write a `for` loop that creates five files named `test1.txt` through `test5.txt`. (Hint: `for i in 1 2 3 4 5`.)
**END OF NOTE**

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>

### 12:15–13:00 — Building an App Scaffold: the `countries` Script
*(Exercise — 45 minutes)*

Now we bring the whole morning together into something genuinely useful. Every time you start a new Express API, you make the same folders and the same starter files by hand. That's *exactly* the kind of repetitive, error-prone, "did I forget one?" task we automate.

We're going to write a script called `scaffold` that, given a resource name like `countries`, builds this entire structure for you in one command:

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

Notice the naming: the controller, router and SQL file are **plural and lowercase** (`countries`), but the model is **singular and capitalised** (`Country`). We'll handle that with arguments.

Let me walk through building it, and you build alongside me. We'll construct it in stages so each part uses something we learned this morning.

**Stage 1 — the skeleton, guards, and folders**

Create `scaffold` and start with the input-handling we just learned:

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

Two new things worth pausing on:

- `set -e` at the top means "if any command in this script fails, stop the whole script right there." It saves us writing an exit-code check after every single line.
- `model="${2:-${resource^}}"` is doing two clever things at once. `${2:-...}` means "use the second argument, **or** this default if it wasn't given." And `${resource^}` capitalises the first letter. So `scaffold countries` gives a model of `Countries`, while `scaffold countries Country` gives exactly `Country`.

**ASK** <br>
Why is it worth putting that `if [ -z "$resource" ]` guard at the very top, before we create a single folder? <br>
**ANSWER** <br>
If someone runs `scaffold` with no argument, `$resource` is empty, and every path we build (`server/models/.js`, files named after nothing) would be malformed. Failing fast with a clear usage message is far kinder than half-creating a broken mess of files and leaving the user to clean it up.

**Stage 2 — the top-level files**

Here we use heredocs. Note the `$resource` variables get filled in because these heredocs are *unquoted* (`<< EOF`, not `<< 'EOF'`):

# EMILE NOTE
- Add nodemon and other depencies, make sure they're latest
- Explain different between 'EOF' and EOF

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

**Stage 3 — the server entry point and app wiring**

`index.js` has no variables of its own, so we quote the marker (`'EOF'`) to keep the backticks and `${}` literal. `app.js` *does* need our resource name substituted, so it stays unquoted:

# EMILE NOTE
- Make sure accurate to what the students are familiar with (especially app.js middleware)

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

# EMILE NOTE
- Update model with parameters etc..

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

# EMILE NOTE
- Update export as db

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

```bash
chmod +x scaffold
mv scaffold ~/bin/scaffold      # so we can call it by name anywhere
cd ~ && mkdir countries-app && cd countries-app
scaffold countries Country
tree
```

If everything's right, `tree` shows the exact structure from the top of this section. You've just replaced ten minutes of clicking and typing with one command — and it's identical, every single time, for everyone on the team.

**HANDS ON (remaining time)** <br>
Build the `scaffold` script alongside me, stage by stage, then:
1. Run `scaffold countries Country` in a fresh empty folder and confirm the tree matches exactly.
2. Run it again in *another* fresh folder as `scaffold books Book` — watch the same structure appear with different names. This is the payoff of using arguments instead of hard-coding.
3. `cat` a couple of the generated files (e.g. `server/models/Book.js`) and confirm the substitutions worked.
4. (Stretch) Add a guard near the top: if a `server/` folder already exists in the current directory, print a warning and `exit 1` rather than overwriting.
5. (Stretch) Add a final line that runs `git init` so every scaffolded app starts as a Git repo.
**END OF NOTE**

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

Think back to the DevOps habits from the Azure session — "automate the boring, repeatable stuff", "small frequent changes", "everything as code". Scripts are how those habits actually get realised. That scaffold script you wrote means every new service in your team starts identically. Later, a build script will mean every image is built and pushed identically. A deploy script will mean every release happens identically. Once the steps live in a script, they can be:

- **Reviewed** — a teammate reads the script in a pull request before it ever runs
- **Version-controlled** — you can see exactly how the process changed over time, and roll it back
- **Triggered automatically** — by a schedule (`cron`, which some of you have met), or by an event like "someone pushed to `main`" (that's Jenkins, in a couple of sessions)
- **Run anywhere** — your laptop, a colleague's, a build server, a cloud shell — and behave the same

**ASK** <br>
We could keep doing all of this by hand and get the same *result* on any given day. So what does moving it into automated scripts actually buy a DevOps team — beyond just saving a bit of typing? <br>
**ANSWER** <br>
Consistency and safety at scale. Humans doing repetitive steps by hand drift — they skip a step, do them out of order, or do it slightly differently on server #23. A script does it identically every time, leaves a reviewable record of *what* runs, fails loudly when something's wrong (exit codes), and can run without anyone present. The value isn't the saved keystrokes; it's that the process becomes reliable and auditable.

Now — bash is the lingua franca of Linux, and it's most of what a DevOps engineer scripts. But it's not the only shell you'll meet, especially in a Microsoft-heavy environment like Azure. So for the next stretch we're going to step into **PowerShell**, right inside the Azure Cloud Shell, and see both how it's similar and how it's fundamentally different.

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

PowerShell commands are called **cmdlets**, and they follow a rigid, predictable naming pattern: **`Verb-Noun`**.

- `Get-ChildItem` — list items in a location (the equivalent of `ls`)
- `Set-Location` — change directory (the equivalent of `cd`)
- `Get-Content` — read a file (like `cat`)
- `New-Item` — create a file or folder (like `touch` / `mkdir`)
- `Get-Help` — get help on any cmdlet
- `Get-Command` — list available commands

Because the verbs are standardised (`Get`, `Set`, `New`, `Remove`, `Start`, `Stop`...), you can often *guess* a cmdlet's name. Want to remove something? `Remove-Item`. Want to start a service? `Start-Service`. That predictability is deliberate.

Helpfully, PowerShell also ships **aliases** so muscle memory from bash mostly works: `ls`, `cd`, `cat`, `pwd`, `cp`, `rm` are all aliased to the equivalent cmdlets. Great for getting started; but write the full cmdlet names in scripts, because the aliases can differ between systems.

**Filtering and sorting — the pipeline in action**

Here's where objects shine. To find the three largest files in a folder:

```powershell
Get-ChildItem | Sort-Object Length -Descending | Select-Object -First 3
```

Read it: list the items, sort them by their `Length` property (biggest first), keep the first three. No `wc`, no `head`, no text parsing — you sorted on a real numeric property. To filter:

```powershell
Get-ChildItem | Where-Object { $_.Name -like "*.json" }
```

`Where-Object` keeps only items where the condition is true; `$_` means "the current object in the pipeline"; `-like` does wildcard matching. Compare that to piping through `grep` in bash — here you're matching on the object's actual `.Name` property.

**Variables**

Same idea as bash, slightly different punctuation — variables start with `$`, and you *do* use spaces around `=`:

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

Both are doing the *same thing* under the hood — both are just polite wrappers around the same **Azure REST API**. Which you use is largely down to your team and what the rest of your tooling is written in. Microsoft-centric shops lean PowerShell; Linux-centric and cross-cloud shops lean the CLI. Knowing both makes you flexible.

**ASK** <br>
Both `az group list` and `Get-AzResourceGroup` end up hitting the same Azure REST API and give you the same resource groups. So is learning PowerShell as well as the CLI a waste of time? <br>
**ANSWER** <br>
No — because you don't get to choose the environment you walk into. Plenty of enterprises, especially Microsoft-heavy ones, have years of existing automation written in PowerShell, use Azure Automation runbooks (which are PowerShell), and expect you to work in it. And the object pipeline genuinely makes some tasks cleaner. Being fluent in both means you're useful whatever the team already runs.

**HANDS ON (25 min)** <br>
In the Azure Cloud Shell, switched to **PowerShell**:

Part A — get your bearings.
1. Run `Get-ChildItem` and then its alias `ls`. Confirm they do the same thing.
2. Run `Get-Command -Verb Get | Select-Object -First 20` to see how many "Get" cmdlets exist.
3. Pick any cmdlet and run `Get-Help Get-AzResourceGroup` to see the built-in documentation.

Part B — objects, not text.
4. Run `Get-ChildItem | Sort-Object Length -Descending | Select-Object Name, Length -First 5`. Notice you selected specific *properties* by name.
5. Run `Get-Process | Where-Object { $_.CPU -gt 1 } | Select-Object Name, CPU` — you've filtered live processes on a numeric property.

Part C — Azure with the Az module.
6. `Get-AzContext` — confirm which subscription you're in.
7. `Get-AzResourceGroup` — list your resource groups as objects.
8. Create one: `New-AzResourceGroup -Name ps-demo-rg -Location uksouth`
9. Confirm it exists, then filter for it: `Get-AzResourceGroup | Where-Object { $_.ResourceGroupName -like "ps-demo*" }`
10. Clean it up: `Remove-AzResourceGroup -Name ps-demo-rg -Force`

Part D (stretch).
11. Store a resource group in a variable and inspect its properties: `$rg = Get-AzResourceGroup -Name <one-of-yours>; $rg | Get-Member` — `Get-Member` lists every property and method the object has. This is how you *discover* what you can do with something in PowerShell.
**END OF NOTE**

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
1. It takes a resource name as `$1` and an optional model name as `$2` (defaulting to a capitalised resource), exactly as this morning.
2. It **validates input**: no resource name → print a clear usage message to `>&2` and `exit 1`.
3. It **refuses to overwrite**: if a `server/` directory already exists in the current folder, warn and `exit 1` rather than clobbering someone's work.
4. It creates the full MVC structure from this morning (`package.json`, `README.md`, `server/` with `controllers`, `db`, `models`, `routers`, and all the files).
5. It **also** writes a working `Dockerfile` into the project root (so Part 2 has something to build):

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

6. It **also** writes a `.dockerignore` containing `node_modules` and `.git`.
7. It uses a `log()` function so every step prints a tidy, timestamped message.
8. It ends by printing a clear "next steps" message telling the user what to run.
9. Install it to `~/bin/new-app` so it's callable by name from anywhere.

**Test it:** in a fresh empty folder, run `new-app countries Country`, then `tree`, and confirm the structure plus the `Dockerfile` and `.dockerignore`.

---

#### Part 2 (≈45 min) — Script two: `containerise`, the build-and-push

Now the automation that turns that scaffolded app into a published image. Write a script called `containerise`.

**Requirements:**
1. **Usage:** `containerise <image-name> [tag]`, where tag defaults to `latest`.
2. **Validate input:** no image name → usage message to `>&2` and `exit 1`.
3. **Pre-flight checks** — fail early and clearly if the environment isn't ready:
   - If there's no `Dockerfile` in the current directory (`-f`), print an error and `exit 1`.
   - If `docker` isn't available (hint: `command -v docker`), print an error and `exit 1`.
4. **Build** the image, tagging it with the name and tag:
   ```bash
   docker build -t "${image}:${tag}" .
   ```
5. **Check the build's exit code.** If the build failed, log a clear failure message to `>&2` and `exit 1`. (This is where exit codes stop being theoretical — a broken build must make the script fail.)
6. **Report success** with the full image name and tag.
7. Use the same `log()` timestamp helper as Part 1.
8. Install it to `~/bin/containerise`.

**Test it:** from inside your scaffolded `countries` app, run `containerise countries-api v1` and watch it build. Run `docker images` to confirm the image exists.

**Then — the push (Azure tie-in).** Extend the script so that if a registry name is supplied as a third argument, it also **pushes to Azure Container Registry**:

```bash
# containerise <image-name> [tag] [acr-registry-name]
registry="$3"

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

**NOTE FOR TRAINERS** <br>
Provisioning an ACR per student may not be practical. Sensible options, in order of preference: (a) provide **one shared ACR** for the room and have students push under distinct tags/image names; (b) let students create a Basic-tier ACR in their own subscription (`az acr create --name <unique> --resource-group <rg> --sku Basic`) and tear it down after; or (c) if registries aren't available at all, have everyone stop after the local `docker build` step and just *write* the push logic without running it — the scripting practice is the real objective. Note that Docker itself must run on the student's Linux/VM environment, **not** the Azure Cloud Shell, which has no Docker daemon. <br>
**END OF NOTE**

**ASK** *(mid-exercise checkpoint, ~15:55)* <br>
In Part 2 we deliberately check the exit code of `docker build` and `exit 1` if it failed. Imagine this script is later run automatically by a Jenkins pipeline. What goes wrong if we *skip* that check and always let the script exit `0`? <br>
**ANSWER** <br>
The pipeline would think the build succeeded even when it didn't, and happily carry on to the next stage — deploying, tagging as "latest", telling everyone it's green — while shipping a broken or stale image. Honest exit codes are the contract between your script and every automated system that will ever run it.

---

#### Part 3 (≈20 min) — Stretch: chain them and add polish

If you've got both scripts working, level them up:

1. **A one-command workflow.** Write a third tiny script, `ship`, that runs `new-app` and then `containerise` in sequence using `&&`, so a single command scaffolds *and* containerises. Think about what should happen if the scaffold step fails — should the build still run?
2. **Logging to a file.** Make `containerise` append its output to a timestamped log file (`>> build-$(date +%F).log 2>&1`) as well as the screen, so there's a record of every build — the way an unattended automated job would need.
3. **A confirmation prompt.** Before pushing to a registry, use `read -p` to ask "Push to <registry>? (y/n)" and only push if the answer is `y`. (Where might an interactive prompt like this be a *problem* in an automated pipeline? Discuss.)
4. **Idempotency.** Make `new-app` safe to re-run: instead of failing outright when `server/` exists, have it skip files that already exist and only create the missing ones.

---

#### What to show at 16:45

Be ready to demonstrate, in a fresh folder:
```bash
new-app books Book        # scaffolds the app + Dockerfile
cd books-app-or-wherever  # (adjust to where you ran it)
containerise books-api v1 # builds the image
docker images             # proves it exists
```
And to explain one place in your scripts where you used an **exit code** to fail safely, and why it matters.

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

1. Rebuild your `new-app` script from memory, without looking at today's notes, and get it scaffolding correctly. Rebuilding from scratch is the fastest way to make it stick.
2. Extend `new-app` so it accepts **multiple** resource names at once — e.g. `new-app countries books users` — creating a controller, router, model and SQL file for each. (Hint: loop over `$@`.)
3. Add a `--help` flag to both scripts: if the first argument is `--help` or `-h`, print usage and exit `0`.
4. Write a small `df-check` script that logs disk usage with a timestamp to a file, and schedule it with `cron` to run every 5 minutes for testing. (Recall exit codes and `>>` for the log.)
5. In PowerShell in the Cloud Shell, write a one-liner that lists all your resource groups and outputs just their names and locations, sorted by location. (Hint: `Get-AzResourceGroup | Sort-Object Location | Select-Object ResourceGroupName, Location`.)
6. (Stretch) Rewrite your `containerise` **build** step's success/failure reporting so it prints the image's size on success (`docker images` filtered to your image).
7. (Stretch) Research and write down: what's the difference between `set -e`, `set -u`, and `set -o pipefail`, and why do experienced engineers often put `set -euo pipefail` at the top of a serious bash script?

<br>
<br>

### Conclusion

- **Inform** students this marks the end of the Bash Scripting & Automation session
- **Tell** students the next session moves into **Jenkins & pipelines**, where the scripts they wrote today start running automatically on every code change — and where today's obsession with exit codes suddenly pays off
- **Direct** students to the take-home exercises, and to the [Bash reference](https://www.gnu.org/software/bash/manual/bash.html) and [Azure PowerShell docs](https://learn.microsoft.com/en-us/powershell/azure/) for further reading

---

[Back](./README.md)

---
