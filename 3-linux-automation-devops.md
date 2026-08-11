# Linux Fundamentals and Automation for DevOps

A full day covering the Linux skills DevOps engineers use daily: the filesystem, permissions, shell scripting, process/service management, and a first taste of configuration management — building towards "why we automate this instead of doing it by hand".

## Organisation

### Duration

Full day, **09:30 - 17:00**, split into four sessions with a mid-morning break, lunch, and a mid-afternoon break.

| Time | Session |
|---|---|
| 09:30 - 11:00 | Session 1: Linux Fundamentals & the Filesystem |
| 11:00 - 11:15 | Break |
| 11:15 - 12:45 | Session 2: Shell Scripting for Automation |
| 12:45 - 13:30 | Lunch |
| 13:30 - 15:00 | Session 3: Processes, Services & systemd |
| 15:00 - 15:15 | Break |
| 15:15 - 16:45 | Session 4: Configuration Management & Idempotency |
| 16:45 - 17:00 | Wrap-up & Q&A |

### Set-up

#### For Trainers

**For the slidee presentation** <br>
**Open** the [Slidee](https://github.com/mklilley/slidee) presentation:

- Run `npx slidee` from within the `curriculum` folder to see the available slide decks (those with extension `.slides.md`)
- Click on `See your presentations`
- Open `Weeks X Y > CDO > Linux and Automation for DevOps`
- Hit `s` to open the presenter notes
- Resource to be kept open on a tab throughout the day to refer to

**Check out** [FAQs](/FAQs/README.md) for help with using Slidee

#### For Students
- **Direct** students to the starter code for this module
  - [Student Facing Repo](https://github.com/LaFosseAcademy/cloud-devops-student-repos/tree/main)
- **Use**
  - **/linux-and-automation-for-devops/starter-code**
- **Make sure**
  - You clone the entire repo
  - You have a Linux environment available: a cloud VM (from earlier Terraform sessions), WSL2 on Windows, or a local VM/container all work. macOS/Linux users already have a usable terminal.

## Learning objectives

- **Understand** the Linux filesystem hierarchy, permissions model, and package management
- **Apply** shell scripting to automate repetitive tasks
- **Understand** how Linux manages processes and long-running services via `systemd`
- **Evaluate** why configuration management tools exist and what "idempotency" means in practice
- **Connect** each of these skills back to why they matter for a DevOps engineer specifically

<br>

---

## Session 1: Linux Fundamentals & the Filesystem
### 09:30 - 11:00

### Learning objectives
- Understand why Linux dominates server-side DevOps work
- Navigate the filesystem hierarchy confidently
- Understand and modify file permissions and ownership
- Use a package manager to install and remove software

### Sequence

#### Why Linux for DevOps?

**ASK** <br>
Nearly every cloud VM, container base image, and CI/CD runner we've used so far — what operating system has it been running? <br>
**ANSWER** <br>
Linux (Amazon Linux, Ubuntu, etc.) <br>

Windows and macOS matter for laptops, but the servers, containers, and pipelines a DevOps engineer touches are overwhelmingly Linux. It's free, scriptable, lightweight, and the whole ecosystem of DevOps tooling — Docker, Kubernetes, Ansible, Terraform providers — assumes a Linux-shaped target underneath.

*REFER TO RESOURCE 1 - SLIDEE* <br>

#### The Filesystem Hierarchy

Unlike Windows with its `C:\`, `D:\` drives, Linux has a single unified tree starting at `/` (root).

Let's connect to our VM (from the Terraform sessions, or a local one) and explore:

* `pwd` — print working directory
* `ls -la /` — list the top-level directories

Key directories worth knowing:
* `/home` — user home directories (`/home/azureuser`, `/home/ec2-user`, etc.)
* `/etc` — system-wide configuration files
* `/var` — variable data: logs, caches, web server files (`/var/www/html`, `/var/log`)
* `/usr` — installed software and shared resources
* `/bin`, `/usr/bin` — executable programs
* `/tmp` — temporary files, cleared on reboot

**ASK** <br>
Where did we write our HTML file to in the Terraform sessions? <br>
**ANSWER** <br>
`/var/www/html/index.html` — now we know **why** it lives under `/var`.

#### Users, Groups and Permissions

Every file has an **owner**, a **group**, and a set of **permissions** for three categories of people: the owner, the group, and everyone else.

* Run: `ls -l /var/www/html/index.html`

We'll see something like:
```
-rw-r--r-- 1 azureuser azureuser 42 Aug  3 10:02 index.html
```

Breaking this down:
* `-` : file type (`-` = file, `d` = directory, `l` = symlink)
* `rw-` : owner permissions (read, write, no execute)
* `r--` : group permissions (read only)
* `r--` : everyone else (read only)

**NOTE FOR TRAINERS** <br>
This is the exact same permission model we used with `chmod 400 default-vm-ssh.pem` back in the resource provisioning session — this is a good moment to connect the two. `chmod 400` translates to `r--------`: owner can read, nobody else can do anything. <br>
**END OF NOTE**

* `chmod` — change permissions (e.g. `chmod 644 file.txt`, `chmod +x script.sh`)
* `chown` — change owner (e.g. `sudo chown azureuser:azureuser file.txt`)

**ASK** <br>
Why did our SSH private key need `400` permissions specifically, rather than something looser like `644`? <br>
**ANSWER** <br>
SSH refuses to use a private key that other users could read — `644` would let anyone on the machine read it, defeating the point of having a private key at all.

#### Package Management

Every Linux distribution has a package manager for installing, updating, and removing software. We saw this already:

* Ubuntu/Debian: `apt`
  * `sudo apt update` — refresh the list of available packages
  * `sudo apt install apache2 -y` — install a package
  * `sudo apt remove apache2 -y` — remove a package
* Amazon Linux/RHEL/CentOS: `yum` or the newer `dnf`
  * `sudo yum install httpd -y`

**ASK** <br>
Why do we run `apt update` before `apt install` on a fresh Ubuntu machine? <br>
**ANSWER** <br>
`apt update` refreshes the local index of what packages and versions are *available*; it doesn't install anything itself. Skipping it means we might try to install a package apt doesn't yet know exists, or get an older version.

#### Exercise
Students SSH into their VM from the Terraform sessions (or a fresh one) and:
1. Explore `/etc`, `/var/log`, and their own home directory
2. Create a file, then experiment with `chmod` values and observe the effect with `ls -l`
3. Install a small package (e.g. `tree` or `htop`) and use it

<br>

---

## Session 2: Shell Scripting for Automation
### 11:15 - 12:45

### Learning objectives
- Write and execute a basic bash script
- Use variables, conditionals, and loops in bash
- Understand exit codes and how scripts signal success/failure
- Schedule a script to run automatically with `cron`

### Sequence

#### Why script instead of typing commands?

We've been running commands one at a time via `remote-exec` provisioners in Terraform. That's fine for three lines, but real servers need dozens of setup steps, run repeatedly, and need to behave consistently every time.

**ASK** <br>
What problems can arise from a human manually typing the same 20 setup commands on 3 different servers? <br>
**ANSWER** <br>
Typos, commands run in the wrong order, steps forgotten, inconsistent results between servers ("works on my machine").

A **shell script** is just a text file of commands, run top to bottom, that we can version control, review, and re-run reliably — this is the foundation everything in DevOps automation builds on.

#### Writing our first script

* `touch setup.sh`
* `nano setup.sh` (or open in VSCode)

**setup.sh**
```bash
#!/bin/bash
# NEW CONFIG
echo "Starting setup..."
sudo apt update -y
sudo apt install apache2 -y
echo "Setup complete!"
```

* The first line, `#!/bin/bash`, is called a **shebang** — it tells the OS which interpreter to run the rest of the file with.
* `#` starts a comment, same as `tf` files.

To run it:
* `chmod +x setup.sh` — make it executable (remember our permissions lesson!)
* `./setup.sh`

**ASK** <br>
Why did we need `chmod +x` before we could run it? <br>
**ANSWER** <br>
By default, new files aren't executable — this is a safety default so that downloaded/created files don't silently run as programs.

#### Variables

**setup.sh**
```bash
#!/bin/bash
# NEW CONFIG
PACKAGE_NAME="apache2"

echo "Installing $PACKAGE_NAME..."
sudo apt update -y
sudo apt install $PACKAGE_NAME -y
echo "$PACKAGE_NAME installed!"
```

* No spaces around `=` when assigning a variable
* Reference a variable with `$NAME` or `${NAME}`

This should feel familiar — it's the same idea as a Terraform `variable` block, just far simpler syntax, and evaluated at *run time* rather than *plan time*.

#### Conditionals

**setup.sh**
```bash
#!/bin/bash
PACKAGE_NAME="apache2"

# NEW CONFIG
if dpkg -l | grep -q $PACKAGE_NAME; then
  echo "$PACKAGE_NAME is already installed, skipping."
else
  sudo apt update -y
  sudo apt install $PACKAGE_NAME -y
  echo "$PACKAGE_NAME installed!"
fi
```

**ASK** <br>
Why might we want to check if something's already installed before installing it again? <br>
**ANSWER** <br>
It saves time on repeat runs, and starts to introduce the idea of a script being safe to run more than once — we'll come back to this exact idea (**idempotency**) in Session 4.

#### Loops

**setup.sh**
```bash
#!/bin/bash
# NEW CONFIG
PACKAGES=("apache2" "curl" "git")

for package in "${PACKAGES[@]}"; do
  echo "Installing $package..."
  sudo apt install $package -y
done
```

**ASK** <br>
Where have we seen a very similar concept before, just with different syntax? <br>
**ANSWER** <br>
Terraform's `for_each` — looping over a list to repeat an action for each item.

#### Exit codes

Every command returns an **exit code** when it finishes: `0` means success, anything else means failure.

* `echo $?` — shows the exit code of the last command run

**setup.sh**
```bash
#!/bin/bash
sudo apt install apache2 -y

# NEW CONFIG
if [ $? -eq 0 ]; then
  echo "Install succeeded"
else
  echo "Install failed" >&2
  exit 1
fi
```

**NOTE FOR TRAINERS** <br>
This is worth dwelling on — CI/CD pipelines (which we'll cover in a later session) decide whether a build "passes" or "fails" purely based on the exit code of the last command in a script. A script that always exits `0` regardless of what actually happened will silently hide failures in a pipeline. <br>
**END OF NOTE**

#### Automating with cron

Scripts are great, but sometimes we want something to run automatically — nightly backups, log cleanup, health checks.

**cron** is a Linux service for scheduling recurring tasks.

* `crontab -e` — edit your personal schedule

**crontab**
```
# NEW CONFIG
# minute hour day month weekday   command
0 2 * * * /home/azureuser/setup.sh >> /home/azureuser/setup.log 2>&1
```

* `0 2 * * *` means "at 02:00, every day"
* `>> ... 2>&1` appends both standard output and errors to a log file — cron jobs run unattended, so if we don't log the output somewhere, we'll never see what happened

*REFER TO RESOURCE 2 - SLIDEE* <br>
![cron-syntax](./resources/cron-syntax.png)

**ASK** <br>
Why is logging especially important for a cron job compared to a script we run ourselves interactively? <br>
**ANSWER** <br>
Nobody's watching the terminal when it runs — if it fails silently at 2am, a log file is the only record.

#### Exercise
Students write a script that:
1. Checks disk space with `df -h`
2. Writes the result, with a timestamp, to a log file
3. Is scheduled via `cron` to run every 5 minutes for testing
4. (Stretch) Only sends output if disk usage exceeds a threshold

<br>

---

## Session 3: Processes, Services & systemd
### 13:30 - 15:00

### Learning objectives
- Understand what a process is and how to inspect running ones
- Start, stop, enable, and inspect services using `systemctl`
- Read service logs with `journalctl`
- Understand how our earlier `sudo systemctl start apache2` commands actually work

### Sequence

#### What is a process?

Every running program on Linux is a **process**, identified by a **PID** (process ID).

* `ps aux` — list all running processes
* `top` or `htop` — live, interactive view of processes and resource usage

**ASK** <br>
When we ran `sudo systemctl start apache2` in our Terraform provisioner, what actually started running on the machine? <br>
**ANSWER** <br>
A process (or a few) — the Apache web server, listening on port 80, waiting for HTTP requests.

* `ps aux | grep apache2` — find our Apache process(es) specifically

Useful process commands:
* `kill <PID>` — ask a process to stop (sends `SIGTERM`)
* `kill -9 <PID>` — force-stop a process (sends `SIGKILL`, no cleanup)

**ASK** <br>
Why might we prefer `kill` over `kill -9` where possible? <br>
**ANSWER** <br>
`kill` gives the process a chance to shut down gracefully (close files, finish in-flight requests); `-9` yanks it out immediately, which can leave things in a bad state.

#### Services and systemd

A **service** is a process that's meant to run continuously in the background, ideally restart itself on failure, and start automatically on boot. Almost every modern Linux distribution manages these using **systemd**.

We've already used this without fully unpacking it:

```bash
sudo systemctl start apache2
```

Let's break down the commands available:

* `sudo systemctl start apache2` — start the service now
* `sudo systemctl stop apache2` — stop it
* `sudo systemctl restart apache2` — stop then start
* `sudo systemctl status apache2` — is it running? Recent log lines? Since when?
* `sudo systemctl enable apache2` — start automatically on every boot
* `sudo systemctl disable apache2` — don't start automatically on boot

**ASK** <br>
Why did we need `sudo systemctl start apache2` at all in our Terraform provisioner, if we'd said earlier that `apt install apache2` starts the service automatically on Ubuntu? <br>
**ANSWER** <br>
It's largely redundant on Ubuntu specifically — but it's a good habit because it isn't true on every distribution (Amazon Linux, for instance, doesn't auto-start `httpd` after install), so being explicit keeps a script portable and predictable.

#### Reading logs with journalctl

systemd captures the output of every service it manages into a central logging system.

* `sudo journalctl -u apache2` — logs just for the apache2 service
* `sudo journalctl -u apache2 -f` — "follow" the log live, like `tail -f`
* `sudo journalctl --since "10 minutes ago"` — filter by time

**ASK** <br>
If our web server's `index.html` isn't showing up when we visit its public IP, where would we look first? <br>
**ANSWER** <br>
`sudo systemctl status apache2` to check it's even running, then `sudo journalctl -u apache2` to see if it logged any startup errors.

#### Writing our own systemd service

Let's say we have a small custom app we want managed the same way. We define a **unit file**.

* `sudo nano /etc/systemd/system/my-app.service`

**my-app.service**
```ini
# NEW CONFIG
[Unit]
Description=My Custom App
After=network.target

[Service]
ExecStart=/home/azureuser/my-app/start.sh
Restart=on-failure
User=azureuser

[Install]
WantedBy=multi-user.target
```

* `[Unit]` — metadata, and what this service depends on (`network.target` = wait for networking to be up)
* `[Service]` — how to run it, and what to do if it crashes (`Restart=on-failure`)
* `[Install]` — when it should start automatically (`multi-user.target` = normal boot)

Then:
* `sudo systemctl daemon-reload` — tell systemd to notice the new unit file
* `sudo systemctl start my-app`
* `sudo systemctl enable my-app`

**NOTE FOR TRAINERS** <br>
This is a good moment to connect back to the "you build it, you run it" DevOps mantra — a service that restarts itself automatically on failure, logs centrally, and starts on boot is a big part of what makes an application resilient in production without a human babysitting it. <br>
**END OF NOTE**

#### Exercise
Students:
1. Write a tiny script that runs an infinite loop printing the time every 10 seconds
2. Wrap it as a systemd service with `Restart=on-failure`
3. Kill the process manually and observe systemd restart it
4. Check the output via `journalctl`

<br>

---

## Session 4: Configuration Management & Idempotency
### 15:15 - 16:45

### Learning objectives
- Understand the limits of ad-hoc shell scripts at scale
- Understand what **idempotency** means and why it matters
- Get a first, practical look at Ansible as a configuration management tool
- Understand where configuration management sits relative to Terraform in the toolchain

### Sequence

#### The problem with scripts at scale

We've spent the day writing scripts and provisioners. They work — but let's stress-test the idea.

**ASK** <br>
If we have 50 servers and need to add one new configuration line to all of them, what does that look like with what we've learned so far? <br>
**ANSWER** <br>
SSH into each one and run a script, or `for_each` a `remote-exec` provisioner in Terraform.

**ASK** <br>
What happens if the script partially fails on server #23? What if someone made a manual change on server #40 that our script doesn't account for? <br>
**ANSWER** <br>
We often don't know — the script just ran the commands, it didn't check the actual current state, or handle every possible starting condition gracefully.

*REFER TO RESOURCE 3 - SLIDEE* <br>

This is the same **Desired vs Known vs Actual State** problem we spent a whole day on with Terraform — except Terraform only manages *infrastructure* (VMs, networks, storage). It generally doesn't manage what's happening *inside* those VMs once they exist. That's the gap **configuration management tools** fill.

#### Idempotency

**Idempotency** means: running the same operation multiple times produces the same result as running it once — it's safe to repeat.

**ASK** <br>
Which of these is idempotent?
* `echo "127.0.0.1 myapp" >> /etc/hosts`
* `sudo systemctl enable apache2`
<br>
**ANSWER** <br>
`systemctl enable` is idempotent — running it 10 times leaves the same end state as running it once. `>>` (append) is **not** — running it 10 times adds the line 10 times.

We touched on this in Session 2 with our `if dpkg -l | grep -q` check before installing — we were manually engineering idempotency into a bash script. Configuration management tools build this in by default.

#### Introducing Ansible

**Ansible** is a configuration management tool. Unlike Terraform (which describes *infrastructure*), Ansible describes the *desired configuration state of a machine*: which packages are installed, which services are running, which files exist with what content.

Key differences from our bash scripts:
* Ansible tasks are **declarative** — you describe the end state, not the steps to get there
* Ansible checks the current state before acting, and **only makes changes when needed** — genuinely idempotent by design, not by us remembering to write an `if` check
* No agent needs to be pre-installed on target machines — Ansible connects over SSH and pushes what it needs

#### Installing Ansible and an inventory

* `sudo apt install ansible -y` (on our control machine, not the targets)

We tell Ansible which machines to manage via an **inventory** file.

* `touch inventory.ini`

**inventory.ini**
```ini
# NEW CONFIG
[webservers]
http-server-1 ansible_host=<public-ip-1> ansible_user=azureuser
http-server-2 ansible_host=<public-ip-2> ansible_user=azureuser
```

**ASK** <br>
Where have we already got a list of IP addresses we could plug in here? <br>
**ANSWER** <br>
The `output` blocks from our Terraform Load Balancer session — `terraform output` can feed straight into an Ansible inventory in a real pipeline.

#### Our first playbook

Ansible's equivalent of a `.tf` file is a **playbook**, written in YAML.

* `touch site.yml`

**site.yml**
```yaml
# NEW CONFIG
- name: Configure web servers
  hosts: webservers
  become: true
  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present
        update_cache: true

    - name: Ensure Apache is running and enabled
      service:
        name: apache2
        state: started
        enabled: true

    - name: Deploy index page
      copy:
        content: "Welcome - configured by Ansible"
        dest: /var/www/html/index.html
```

Let's compare this directly to what we hand-wrote in bash and in Terraform's `remote-exec`:
* `state: present` — declares the *desired state*, and Ansible only installs if it isn't already there (compare to our manual `dpkg -l | grep` check)
* `state: started, enabled: true` — same idea as our `systemctl start` / `systemctl enable`, but declared, not scripted
* `copy` — writes the file's content exactly, and does nothing if it's already correct — genuinely idempotent

Run it:
* `ansible-playbook -i inventory.ini site.yml`

Run it again immediately:
* `ansible-playbook -i inventory.ini site.yml` <br>

**ASK** <br>
What would we expect to see differently on the second run? <br>
**ANSWER** <br>
Every task reports `ok` (unchanged) rather than `changed`, because the desired state already matches — this is idempotency in action, visible directly in the output.

#### Where this fits in the DevOps toolchain

*REFER TO RESOURCE 4 - SLIDEE* <br>

| Tool | Manages | Answers the question |
|---|---|---|
| Terraform | Infrastructure (VMs, networks, storage, DNS) | "What resources should exist?" |
| Ansible | Configuration inside those resources | "What should be installed/running/configured?" |
| Bash scripts | One-off or glue tasks | "What one thing needs doing right now?" |

**NOTE FOR TRAINERS** <br>
It's worth being honest that in practice, teams sometimes use Terraform's `remote-exec` provisioner (as we did) for simple cases and reach for full configuration management only once things get complex — there's no hard rule, and reasonable engineers draw this line in different places. <br>
**END OF NOTE**

#### Exercise
Students:
1. Take the VM(s) created in an earlier Terraform session (or spin up a fresh one)
2. Write an inventory file pointing at it
3. Write a playbook that installs a package, starts/enables its service, and deploys a file
4. Run the playbook twice and observe the idempotent `ok` vs `changed` output

<br>

---

## Wrap-up & Q&A
### 16:45 - 17:00

- **Recap** the day's arc: filesystem & permissions → scripting → services → configuration management, each layer solving a limitation of the one before it
- **Connect** today's Linux/automation skills to the Terraform work from earlier sessions — infrastructure and configuration are two halves of the same problem
- **Direct** students to the exercises for further practice
- **Preview** what's coming next: CI/CD pipelines, where scripts like today's get triggered automatically on every code change

---

[Back](./README.md)

---

