# Jenkins & CI/CD Pipelines — Trainer Script

A full day taking entry-level trainees from "I've heard the word pipeline" to "I've built one that tests, builds and ships a Docker image automatically". The emphasis throughout is on **what a pipeline actually is, what it's for, and why teams rely on them** — the mechanics of Jenkins are the vehicle, not the point. This is written as a script — read it out, adapt where it feels natural, but the intent is that you could deliver this cold from the page.

## Organisation

### Audience

Entry-level DevOps trainees who have already covered **Azure cloud fundamentals**, **Docker**, and **bash scripting & automation**. Assume they can build and run a Docker image, write a small shell script, and use Git at a basic level (clone, commit, push). Assume **no** prior exposure to Jenkins or any CI/CD tool, and **no** prior exposure to Groovy — so the `Jenkinsfile` syntax is explained from first principles, block by block. Jenkins runs on **each student's own laptop** — Windows or Mac — so a chunk of today is making that work reliably.

### How this document is laid out — read before delivering

Today you're constantly switching between **a terminal** and **a browser**, which is exactly where trainees get lost. So every instruction block is labelled with where it happens:

- *(Run from `~/jenkins-training`)* — a **terminal** command, in that folder
- *(In the Jenkins UI — Dashboard)* — a **browser** action, starting from that screen
- Navigation steps are written as breadcrumbs, e.g. **Manage Jenkins → Credentials → Add Credentials**

The folders we use today:

| Folder | What it's for |
|---|---|
| `~/jenkins-training` | Where we build the custom Jenkins image and run Docker commands |
| `~/jenkins-training/<your-fork>` | Your cloned fork of the starter repo — where the `Jenkinsfile` lives |

**Every activity has a `**Solution**` block** immediately afterwards, so you can reveal the answer without hunting for it.

Set the working folder up before you start:

*(Run from `~/`)*
- Run: `mkdir -p ~/jenkins-training` → **cd inside** with `cd ~/jenkins-training`
- Confirm with `pwd`

### Duration & schedule

Full day, **09:00–17:00**, with a one-hour lunch and two 15-minute breaks. Roughly **4 hours of instruction and 3 hours of hands-on**, weighted towards an end-of-day build-and-push capstone.

| Time | Section | Shape |
|---|---|---|
| 09:00–09:15 | Welcome & recap: where we are | Talk |
| 09:15–10:00 | What CI/CD is, what pipelines are *for*, and why | Talk + discussion |
| **10:00–10:15** | **Break** | |
| 10:15–11:15 | Installing & touring Jenkins on your laptop | **Exercise (install) + tour** |
| 11:15–12:15 | Your first job: watching Jenkins build something | **Exercise** |
| 12:15–13:00 | Pipeline as code: your first `Jenkinsfile` | **Exercise + Challenge** |
| **13:00–14:00** | **Lunch** | |
| 14:00–14:45 | Anatomy of a pipeline: stages, environment, post, parallel | Talk + short exercise |
| 14:45–15:00 | Connecting Jenkins to Git, and triggering builds | Talk + demo |
| **15:00–15:15** | **Break** | |
| 15:15–16:45 | Capstone: a real test → build → push pipeline | **Exercise (1 hr 30 min)** |
| 16:45–17:00 | Wrap-up & Q&A | Talk |

### Set-up

#### For Trainers

**For the slidee presentation** <br>
**Open** the [Slidee](https://github.com/mklilley/slidee) presentation:

- Run `npx slidee` from within the `curriculum` folder to see the available slide decks (those with extension `.slides.md`)
- Click on `See your presentations`
- Open `Weeks X Y > CDO > Jenkins and CI-CD Pipelines`
- Hit `s` to open the presenter notes
- Resource to be kept open on a tab throughout the day to refer to

#### For Students
- **Direct** students to the starter code for this module
  - [Student Facing Repo](https://github.com/LaFosseAcademy/cloud-devops-student-repos/tree/main)
- **Use**
  - **/jenkins/intro-to-jenkins/starter-code**
- **Make sure**, *before the session starts*, every student has:
  - **Docker Desktop** installed and running (Windows or Mac) — we run Jenkins itself as a container, so this is non-negotiable
  - On **Windows**: Docker Desktop using the **WSL2 backend** (its default), and **Git Bash** installed (comes with [Git for Windows](https://git-scm.com/download/win)) so the multi-line commands below paste cleanly
  - A **GitHub account**, with the starter repo **forked** and cloned locally
  - A free **[Docker Hub](https://hub.docker.com)** account (needed for the afternoon capstone)
  - At least a few GB of free disk — Jenkins plus a couple of images adds up
  - No Terraform or Azure needed today — that's a later session, which today sets up perfectly

**NOTE FOR TRAINERS — the one thing that will eat your morning** <br>
The single biggest time-sink in a room of local laptops is Docker Desktop not being installed or not running. Send a setup checklist the day before and, if you can, have students confirm `docker run hello-world` works *before* they arrive. Ten minutes of prep saves an hour of firefighting. <br>
**END OF NOTE**

## Learning objectives

- **Explain** what CI/CD means, what a pipeline is, and *why* teams use them instead of releasing by hand
- **Install** and run a working Jenkins instance locally on Windows or Mac
- **Create and run** a Freestyle job and watch a build execute
- **Read and write** declarative `Jenkinsfile` syntax — blocks, stages, steps, string interpolation, `sh`
- **Use** environment variables, parameters, `post` actions, and parallel stages
- **Understand** the difference between pipeline-in-the-UI and pipeline-as-code, and why the second one wins
- **Connect** Jenkins to a Git repository and trigger builds automatically
- **Build** a complete pipeline that tests an app, builds a Docker image, and pushes it — the exact shape a real team ships software with

<br>

## Sequence

### 09:00–09:15 — Welcome & Recap: Where We Are

Morning everyone. Let's place today in the story so far, because it slots right in.

You've done **Azure fundamentals** — and one of the big ideas there was "ClickOps is fragile": building things by hand in a portal isn't repeatable, has no review, no audit trail. You've done **Docker** — you can package an app into an image and push it to a registry. And you've done **bash scripting & automation**, where you literally wrote a script by hand that built a Docker image and pushed it. At the very end of that session I asked: *what's the missing piece that would make these scripts run automatically, every time someone pushes code, without anyone typing anything?*

Today is that missing piece. It's called a **pipeline**, and the tool we'll use to build one is **Jenkins**.

Here's the shape of the day. This morning is mostly about the *idea* — what CI/CD and pipelines actually are and why anyone bothers — and then getting Jenkins running on your own laptop and building your first jobs. This afternoon we go deeper into writing pipelines as code, and then spend the back half building one real, end-to-end pipeline that tests an app, builds a Docker image, and pushes it to Docker Hub — completely automatically.

Jenkins runs on *your* machine today, so there's a bit of setup, and setup on real laptops is always where the gremlins live — shout early if something's not working and we'll sort it together. Lunch is at 1 for an hour, breaks mid-morning and mid-afternoon.

**Everyone make a working folder now, so we're all in the same place:**

*(Run from `~/`)*
- Run: `mkdir -p ~/jenkins-training` → **cd inside** with `cd ~/jenkins-training`
- Run: `docker --version` — confirm Docker responds. If it doesn't, flag it now, not at 10:15.

<br>
<br>

### 09:15–10:00 — What CI/CD Is, What Pipelines Are *For*, and Why

Before we touch Jenkins, I want to spend proper time on the *why*, because if the why lands, everything else today is just detail.

**Let's start with the pain.** Imagine a team with no pipeline. A developer finishes a feature. Now what has to happen before real users can use it? They need to pull the latest code, merge their work in, run the tests, build the app, package it up, copy it to a server, restart something, and check it didn't break. Every one of those steps is done by hand, by a human, usually the same one or two humans who "know how the deploy works".

**ASK** <br>
What do you think goes wrong, over time, with a team that releases software this way — entirely by hand? <br>
**ANSWER** <br>
Loads. Someone forgets a step. Someone does the steps in a different order. It "works on my machine" but not on the server because the environments differ. The one person who knows the deploy goes on holiday and nobody else dares release. Because releasing is scary and manual, they do it rarely — which means each release is huge, which makes it *more* likely to break, which makes them even more scared to release. It's a doom loop.

That doom loop is the problem CI/CD exists to break. Let's define the terms properly, because people throw them around loosely.

**Continuous Integration (CI)** means: every time someone pushes code, it's automatically **built and tested**, straight away. Not next week, not before a release — within minutes. The goal is to catch a problem *the moment it's introduced*, while it's small and while the person who caused it still remembers what they changed.

**Continuous Delivery (CD)** means: every change that passes CI is automatically **packaged and made ready to release** — so shipping it is a single, boring, low-risk button press whenever a human decides the time is right.

**Continuous Deployment (also CD)** is the stricter version: every change that passes all the checks goes **all the way to production automatically**, with no human in the loop at all.

**ASK** <br>
What's the actual difference between Continuous *Delivery* and Continuous *Deployment*? <br>
**ANSWER** <br>
Delivery keeps a human deciding *when* to release — the release itself is automated and easy, but a person still presses go. Deployment removes even that decision: every change that passes the checks is live automatically. Delivery is the far more common starting point for a team; full Deployment takes a lot of confidence in your tests.

**So what is a "pipeline", concretely?** A pipeline is the **automated assembly line** those ideas run on. It's an ordered series of **stages** — checkout the code, build it, test it, package it, deploy it — where each stage has to pass before the next one runs. If any stage fails, the line stops, and nothing broken makes it further down.

*REFER TO RESOURCE 1 - SLIDEE* <br>

Let me make it concrete with something you've already done. In the bash session, you wrote two scripts: one that scaffolded an app, and one — `containerise` — that ran `docker build` and pushed the image, and it deliberately checked exit codes so it would *stop* if the build failed. **That was a pipeline in miniature, run by hand.** A stage that builds, a stage that pushes, and a gate between them that halts on failure. All Jenkins does is run that assembly line *for you*, *automatically*, *every time the code changes*, and show you a nice picture of which stage passed or failed.

**ASK** <br>
Thinking back to the DevOps mantra from the Azure session — "you build it, you run it" — how does a pipeline actually support that? <br>
**ANSWER** <br>
It closes the gap between "I finished writing code" and "my code is running somewhere real", and it gives the person who wrote it fast, automatic feedback on whether it works. Instead of throwing code over a wall to a separate ops team and waiting, the same engineer pushes and, minutes later, knows whether they broke anything. The pipeline is what makes "you run it" practical rather than terrifying.

**Where does Jenkins fit?** Jenkins is an **automation server**. Its entire job is: *watch for an event, and when it happens, run some tasks.* The event is usually "someone pushed to Git". The tasks are whatever you tell it — run tests, build an image, deploy. Jenkins genuinely does not care what the tasks are; it's a very good, very configurable "run these steps when this happens" engine.

**ASK** <br>
Jenkins is far from the only tool that does this. What others might you have heard of? <br>
**ANSWER** <br>
GitHub Actions, GitLab CI, CircleCI, Azure Pipelines, Travis. Jenkins is one of the oldest and most widely used. The reason we learn it today is that it's **self-hosted** — you install and run every piece of it yourself, so nothing is a hidden black box. Once you understand Jenkins, the managed ones feel easy.

One last mental model before we install it. Jenkins has two conceptual parts, and it's worth knowing the words even though we'll run everything on one machine today:

- The **controller** runs the web interface, stores your configuration, and decides what work needs doing.
- **Agents** are separate machines or containers the controller hands the actual work to.
- An **executor** is a single "slot" for running one build at a time. A node with two executors can run two builds simultaneously.

**ASK** <br>
Why might a real team not want the controller itself running every build directly? <br>
**ANSWER** <br>
One badly-behaved build — an infinite loop, something that fills the disk — could take down the whole Jenkins server that every team depends on. Agents isolate that risk, and let you scale: busy? add more agents. For learning on our laptops today, we'll just let Jenkins use itself as the builder, which is fine for a classroom but not how you'd run it for real.

That's the whole conceptual foundation. Everything from here is making it real.

<br>
<br>

*(Take a 15 minute break here — and use it to make sure everyone's Docker Desktop is actually running before the install.)*

<br>
<br>

### 10:15–11:15 — Installing & Touring Jenkins on Your Laptop
*(Exercise + guided tour)*

Right — let's get a real Jenkins running on your own machine. We're going to run Jenkins itself as a **Docker container**, for three reasons: you already know Docker, it works *identically* on Windows and Mac (no OS-specific installer mess), and it keeps your laptop clean — if it all goes wrong, we delete a container and start again.

There's one wrinkle we're going to solve up front. Later today our pipeline needs to run `docker build` *from inside* Jenkins. The standard Jenkins image doesn't include the Docker command-line tool, so we'll build a tiny custom image that adds it, and give Jenkins access to your laptop's Docker. Doing this now means the afternoon capstone "just works".

**Step 1 — build a Jenkins image that can use Docker**

*(Run from `~/jenkins-training`)*
- Run: `touch Dockerfile`
- Then open it: `code Dockerfile`

```dockerfile
FROM jenkins/jenkins:lts-jdk17
USER root
RUN apt-get update && apt-get install -y docker.io && rm -rf /var/lib/apt/lists/*
USER jenkins
```

Read it line by line — this is exactly the Docker knowledge you already have:
- `FROM jenkins/jenkins:lts-jdk17` — start from the official Jenkins image. `lts` means Long Term Support, the stable release line.
- `USER root` — switch to the root user, because installing packages needs admin rights
- `RUN apt-get update && apt-get install -y docker.io ...` — install the Docker **command-line tool** (not a whole Docker engine). The `&&` is the one from your bash session: only install if the update succeeded. `rm -rf /var/lib/apt/lists/*` deletes the downloaded package index afterwards to keep the image small — a standard Dockerfile habit.
- `USER jenkins` — drop back to the normal jenkins user, so we're not running as root unnecessarily.

Build it:

*(Run from `~/jenkins-training`)*
```bash
docker build -t jenkins-docker .
```

`-t jenkins-docker` names ("tags") the image; the `.` at the end means "build using the Dockerfile in this current directory".

**Step 2 — run it**

On **Mac** (Terminal) or **Windows** (use **Git Bash**, so the `\` line-breaks work):

*(Run from `~/jenkins-training`)*
```bash
docker run -d --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -u root \
  jenkins-docker
```

If you'd rather paste a single line (works in any shell, including PowerShell):

*(Run from `~/jenkins-training`)*
```bash
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -u root jenkins-docker
```

What each flag does — go through these, they're all revision from the Docker session:
- `-d` — **detached**: run in the background and give you your terminal back
- `--name jenkins` — give the container a friendly name, so we can say `docker stop jenkins` rather than using a random ID
- `-p 8080:8080` — **port mapping**, `host:container`. Traffic to port 8080 on your laptop is forwarded to port 8080 inside the container, which is where the Jenkins web interface listens
- `-p 50000:50000` — the port Jenkins uses to talk to agents (unused today, but standard)
- `-v jenkins_home:/var/jenkins_home` — a **named volume**. Everything Jenkins saves (config, jobs, plugins, your user) lives at `/var/jenkins_home` *inside* the container; this maps it to storage managed by Docker on your laptop, which survives the container being deleted
- `-v /var/run/docker.sock:/var/run/docker.sock` — this one's different: it mounts your laptop's **Docker socket** into the container. The socket is the "phone line" the `docker` command uses to talk to the Docker engine. Handing it in means Jenkins can run `docker build` using your laptop's Docker. **This is the bit that makes the capstone possible.**
- `-u root` — run as root, to sidestep Docker permission faff. Fine for training; **not** how you'd do it in production
- `jenkins-docker` — the image to run (the one we just built)

**NOTE FOR TRAINERS** <br>
On both Docker Desktop for Mac and Docker Desktop for Windows (WSL2 backend), `/var/run/docker.sock` is exposed and the socket mount works as written — this is the reliable cross-platform path. Running `-u root` is a deliberate simplification to avoid the docker-group permission dance in a classroom; call it out honestly as a training shortcut so nobody copies it into a real setup. If a student's install refuses the socket mount, they can still do the *entire* day except the final `docker build`/`docker push` stages, so don't let one broken socket block someone — pair them up. <br>
**END OF NOTE**

Check it's actually running:

*(Run from `~/jenkins-training`)*
```bash
docker ps
```

You should see a container named `jenkins` with a status of "Up". If it's not there, run `docker ps -a` to see stopped containers, and `docker logs jenkins` to see why it fell over.

**Step 3 — unlock and set up**

Jenkins takes a minute or two to start. Then fetch the one-time admin password from inside the container:

*(Run from `~/jenkins-training`)*
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

`docker exec jenkins <command>` means "run this command inside the already-running container called jenkins". Here we're `cat`-ing a file that only exists inside it.

*(In your browser)*
- Go to **`http://localhost:8080`**
- Paste the password in
- Choose **Install suggested plugins** — this pulls in the Git, Pipeline and Credentials plugins we'll use all day. Give it a few minutes.
- Create your first **admin user** when prompted — **write the username and password down**, you'll log in with it all day
- Accept the default Jenkins URL

**ASK** <br>
We deliberately mounted a named volume at `/var/jenkins_home`. Based on the Docker session, what would happen to your admin user, your plugins, and every job you build today if we *hadn't*? <br>
**ANSWER** <br>
They'd all vanish the moment the container was removed and recreated, because a container's own filesystem is disposable. The named volume stores Jenkins' data on the host, outside the container's lifecycle, so it persists. This is the same reason a database container needs a volume — state that must survive belongs outside the container.

**A quick tour of the interface**

*(In the Jenkins UI — Dashboard, at `http://localhost:8080`)*

Point these out on your own screen as you describe them:

- **Dashboard** — the home page; every job you create shows here with its recent status (blue/green = passing, red = failing)
- **New Item** (top left) — how you create a new job. "Item" is just Jenkins' word for a job
- **Manage Jenkins** (left sidebar) — the settings hub. Three areas we care about:
  - **Plugins** — almost everything Jenkins can do comes from plugins. The "suggested" set you just installed is only a starter kit
  - **Credentials** — a secure vault for secrets (Git tokens, Docker Hub passwords). We'll use this properly this afternoon
  - **Nodes** — where build machines are listed
- **Build History** (left sidebar) — a running log of every build across all jobs

**HANDS ON (remaining time)** <br>
1. Get Jenkins running via the steps above and log in with your new admin user.
2. *(In the Jenkins UI)* Go to **Manage Jenkins → Plugins → Installed plugins**. Scroll the list and pick **three plugins you don't recognise** — search their names and jot down, in a sentence each, what they do.
3. *(In the Jenkins UI)* Go to **Manage Jenkins → Nodes**. You'll see one node, "Built-In Node". This is the controller acting as its own builder — exactly the "fine for learning, not for production" setup we described. Note how many **executors** it has.
**END OF NOTE**

**Solution**

The full install sequence, start to finish:

*(Run from `~/`)*
```bash
mkdir -p ~/jenkins-training && cd ~/jenkins-training
```

*(Run from `~/jenkins-training`)*
```bash
# 1. Create the Dockerfile (contents as above), then build it
docker build -t jenkins-docker .

# 2. Run it
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -u root jenkins-docker

# 3. Confirm it started
docker ps

# 4. Get the unlock password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Then *(in your browser)* `http://localhost:8080` → paste password → **Install suggested plugins** → create admin user.

Step 3 answer: the Built-In Node has **2 executors** by default, meaning Jenkins can run two builds at the same time before queuing.

Useful troubleshooting commands to have on hand:

*(Run from `~/jenkins-training`)*
```bash
docker logs jenkins          # see why it won't start
docker stop jenkins          # stop it (data survives — it's in the volume)
docker start jenkins         # start it again
docker rm -f jenkins         # remove the container entirely (data STILL survives)
docker volume ls             # confirm jenkins_home volume exists
```

<br>
<br>

### 11:15–12:15 — Your First Job: Watching Jenkins Build Something
*(Exercise)*

We're going to make Jenkins actually *do* something now, using the simplest possible job type. It's not how we'll write pipelines for real, but it makes the core mechanic — "Jenkins runs your steps and shows you the output" — completely visible before we add any syntax on top.

**A Freestyle job**

A **Freestyle project** is a job configured entirely by clicking in the web UI. No code at all.

*(In the Jenkins UI — Dashboard)*
- Click **New Item** (top left)
- **Enter an item name**: `hello-jenkins`
- Select **Freestyle project** → click **OK**
- You land on the job's configuration page. Scroll down to **Build Steps**
- Click **Add build step** → **Execute shell**
- In the box that appears, enter:

```bash
echo "Hello from Jenkins!"
echo "This build is running as: $(whoami)"
echo "The time is: $(date)"
```

*(Note: this is plain bash — exactly what you wrote in the last session. `$(whoami)` and `$(date)` are the command substitution you already know.)*

- Click **Save** at the bottom
- On the job's page, click **Build Now** in the left sidebar
- A build appears under **Build History** (bottom left) numbered `#1`. Click **#1**, then click **Console Output**

There it is — the exact output of your shell commands, captured and shown in the browser. That's the entire heart of Jenkins: it ran your steps somewhere, and kept the log.

**ASK** <br>
Where in an *earlier* session did we see almost exactly this pattern — a set of shell commands running unattended, where we could only see what happened by looking at a log afterwards? <br>
**ANSWER** <br>
`cron` in the automation session — a scheduled, unattended script whose output you only see by checking a log file. Jenkins' Console Output is doing the same job, just with a proper interface and a lot more built around it. If you understood why cron logs matter, you already understand why Jenkins keeps build logs.

**The workspace and built-in variables**

Every build runs inside a **workspace** — a folder on disk where Jenkins does its work, one per job. And Jenkins automatically injects useful **environment variables** into every build. Let's use one.

*(In the Jenkins UI — the `hello-jenkins` job)*
- Click **Configure** in the left sidebar
- Scroll to your **Execute shell** box and replace its contents with:

```bash
echo "This is build number $BUILD_NUMBER"
echo "The job is called $JOB_NAME"
echo "My workspace is $WORKSPACE"
echo "Build $BUILD_NUMBER of job $JOB_NAME" > result.txt
cat result.txt
```

The variables Jenkins gives you for free include:

| Variable | What it holds |
|---|---|
| `$BUILD_NUMBER` | The number of this build — 1, 2, 3... increments every run |
| `$JOB_NAME` | The name of the job (`hello-jenkins`) |
| `$WORKSPACE` | The full path to the folder this build is running in |
| `$BUILD_URL` | The web address of this specific build |

- **Save** → **Build Now**, twice. Open the **Console Output** each time.

**ASK** <br>
If you run this job five times, what will `result.txt` say each time, and why? <br>
**ANSWER** <br>
A different number each run — `$BUILD_NUMBER` increases by one every build. Jenkins gives every build a unique, ever-incrementing number, which becomes really useful later for tagging things (like Docker images) so you can trace exactly which build produced which artifact.

**Keeping the output — artifacts**

Right now `result.txt` is buried in a workspace folder. An **artifact** is a file a build produces that you want to keep and download afterwards. Let's publish it.

*(In the Jenkins UI — the `hello-jenkins` job)*
- **Configure** → scroll to **Post-build Actions**
- Click **Add post-build action** → **Archive the artifacts**
- **Files to archive**: `result.txt`
- **Save** → **Build Now**
- On the completed build's page, `result.txt` now appears as a downloadable link

**Making a job trigger itself — cron syntax**

*(In the Jenkins UI — the `hello-jenkins` job)*
- **Configure** → **Build Triggers** → tick **Build periodically**
- In the **Schedule** box enter: `H/5 * * * *`

That's the cron syntax from the automation session. Five fields, in order:

```
 minute  hour  day-of-month  month  day-of-week
   */5     *         *         *         *
```

`*` means "every". So `*/5 * * * *` means "every 5 minutes, every hour, every day".

Jenkins adds one twist: **`H` means "hash"** — it spreads the load. `H/5` still means "every 5 minutes", but Jenkins picks *which* minute within each 5-minute window based on the job name, so 200 jobs don't all fire at exactly the same second. Use `H` wherever you'd otherwise use `*` in the minute field — it's the Jenkins convention.

- **Save**, and watch it trigger itself over the next few minutes. Then **turn it off again** so it isn't running all day.

**The catch with Freestyle — and why we're about to leave it**

We've got a working job. But let's poke at it the way we poke at everything in this course.

**ASK** <br>
All of this configuration — the shell commands, the archive setting, the trigger schedule — where does it actually *live*? <br>
**ANSWER** <br>
Inside Jenkins itself, in its own internal config (in that `jenkins_home` volume). **Not** in your Git repository.

**ASK** <br>
Given everything this course has hammered about wanting things "as code" — reviewable, versioned, repeatable — what problems does that cause? <br>
**ANSWER** <br>
No version history of how the build process changed. No code review before someone alters it. No easy way to copy it to another project or run different behaviour on different branches. If someone clicks the wrong thing, there's no diff and no undo. It's ClickOps all over again — the exact problem we called out with the Azure Portal, now wearing a Jenkins costume.

That gap is what **Pipelines** fix, and that's where we're going next.

**HANDS ON (remaining time)** <br>
1. Build the `hello-jenkins` job, including the `$BUILD_NUMBER` version and the archived artifact.
2. Add a **Build periodically** trigger of `H/5 * * * *`, watch it fire on its own, then turn it off.
3. Add a second shell step that deliberately fails (`exit 1`) *before* the working one, rebuild, and look at what the build status and Console Output show you.
4. In pairs, discuss: if three people on your team disagreed about what the build steps should do, how would you even resolve that disagreement with a Freestyle job? (Hint: there's nothing to review, comment on, or merge.)
**END OF NOTE**

**Solution**

**Step 1 —** *(In the Jenkins UI — Dashboard)* **New Item** → `hello-jenkins` → **Freestyle project** → **OK** → **Build Steps** → **Add build step** → **Execute shell**:

```bash
echo "This is build number $BUILD_NUMBER"
echo "The job is called $JOB_NAME"
echo "Build $BUILD_NUMBER of job $JOB_NAME" > result.txt
cat result.txt
```

Then **Post-build Actions** → **Archive the artifacts** → **Files to archive**: `result.txt` → **Save** → **Build Now**.

**Step 2 —** **Configure** → **Build Triggers** → tick **Build periodically** → Schedule: `H/5 * * * *` → **Save**. Untick it again afterwards.

**Step 3 —** Add this as a build step *above* the working one:

```bash
echo "About to fail on purpose"
exit 1
```

The build goes **red**, the Console Output ends at `exit 1`, and **the second build step never runs** — Jenkins stopped the moment a step returned a non-zero exit code. That's the same exit-code contract from the bash session, now being enforced by a tool. Remove the step to go green again.

**Step 4 discussion answer —** You couldn't resolve it properly. There's no diff to look at, no pull request to comment on, no history showing who changed what and why. Somebody just clicks, and the change is live for everyone immediately. That's precisely the argument for pipeline-as-code.

<br>
<br>

### 12:15–13:00 — Pipeline as Code: Your First `Jenkinsfile`
*(Exercise + Challenge)*

Here's the big shift. Instead of configuring a job by clicking, we describe the whole pipeline in a **text file** called a `Jenkinsfile`, and we keep that file **in Git, alongside the code it builds**.

**ASK** <br>
Where have we seen this exact philosophy already — describing something as a text file in version control instead of clicking it into existence? <br>
**ANSWER** <br>
Everywhere this course goes: the bash scripts you wrote, the Dockerfile that defines an image, and — coming up soon — Terraform's `.tf` files. Same instinct every time: the desired thing described as reviewable, versioned, repeatable code, not clicked together by hand and forgotten.

**A word on the syntax before we write any**

A `Jenkinsfile` is written in a language called **Groovy**. You don't need to learn Groovy — we're using a restricted, structured subset called **Declarative Pipeline**, which is really just a set of nested blocks. But three bits of syntax will look unfamiliar, so let's name them now:

**1. Everything is a block, marked by `{ }`.** A block is a named container holding other things. They nest inside each other like Russian dolls:

```groovy
pipeline {        // outermost block
    stages {      // a block inside it
        stage('Build') {   // a block inside that
            steps {        // and one more
                echo 'hello'
            }
        }
    }
}
```

There are **no semicolons** at the end of lines, and indentation is for humans (Groovy doesn't care), but keep it tidy or the nesting becomes impossible to read.

**2. `sh` runs a shell command.** `sh 'npm install'` is exactly like typing `npm install` in a terminal. It's how a pipeline actually *does* anything. (There's also `bat` for Windows batch commands — we won't need it, because our Jenkins runs in a Linux container.)

**3. Single quotes and double quotes behave differently.** This one genuinely catches people out:

| Written as | Behaviour |
|---|---|
| `'Hello ${NAME}'` | **single quotes** — literal. Groovy leaves `${NAME}` exactly as typed |
| `"Hello ${NAME}"` | **double quotes** — Groovy substitutes the variable's value in |

So `echo "Building ${APP_NAME}"` prints the value; `echo 'Building ${APP_NAME}'` prints the literal text. **If you want a variable filled in by Groovy, you need double quotes.** (Confusingly, `sh 'echo $APP_NAME'` also works — because there the *shell* does the substitution, not Groovy. Both routes get you there; just be aware there are two different mechanisms.)

**The simplest possible pipeline**

```groovy
pipeline {
    agent any

    stages {
        stage('Say Hello') {
            steps {
                echo 'Hello from a real pipeline!'
            }
        }
    }
}
```

Reading it top to bottom:
- `pipeline { }` — every declarative pipeline is wrapped in this. Always. It's the outermost container
- `agent any` — "run this on any available builder/executor". Today that's our controller
- `stages { }` — the container holding the ordered list of phases. **This is the assembly line**
- `stage('Say Hello') { }` — one phase. The text in quotes is its name, and it's what shows up in the UI
- `steps { }` — the container for the actual things to do in that stage
- `echo` — a built-in step that just prints a message

**Run it**

For now we'll paste it straight into Jenkins (we'll move it into Git this afternoon).

*(In the Jenkins UI — Dashboard)*
- **New Item** → name it `hello-pipeline` → select **Pipeline** → **OK**
- Scroll right down to the **Pipeline** section at the bottom
- Leave **Definition** as **Pipeline script**
- Paste the pipeline above into the **Script** box
- **Save** → **Build Now**

Look at the build page — Jenkins now shows a **Stage View**: a box labelled "Say Hello" with a green tick. As pipelines grow, that visual lets you see *at a glance* exactly which stage passed and which broke.

**Add a second stage so you can see the assembly line**

*(In the Jenkins UI — the `hello-pipeline` job)*
- **Configure** → replace the script with:

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'echo "pretend build step running"'
            }
        }
        stage('Test') {
            steps {
                echo 'Running the tests...'
                sh 'echo "pretend tests passing"'
            }
        }
    }
}
```

- **Save** → **Build Now**, and watch the Stage View show **two** boxes, Build then Test, lighting up green in order.

**ASK** <br>
Why is it genuinely useful that each stage shows up separately in the UI, rather than the whole build just being "passed" or "failed"? <br>
**ANSWER** <br>
When something breaks, you instantly see *where* — "Test failed" tells you far more than "the build failed". On a real pipeline with checkout, build, test, package and deploy stages, that pinpointing saves a lot of time, and it's why we split work into named stages even when we technically could cram it all into one.

**HANDS ON (20 min)** <br>
*(All in the Jenkins UI — the `hello-pipeline` job)*
1. Create `hello-pipeline` and get the two-stage version above running with a green Stage View.
2. Add a **third** stage called `Package` that echoes a message.
3. Make one of the `sh` steps deliberately fail — change it to `sh 'exit 1'` — rebuild, and watch that stage go **red** and the stages *after* it **not run at all**. This is the single most important thing a pipeline does: **stop the line when something's wrong, before the broken thing goes any further.** Then fix it back to green.
**END OF NOTE**

**Solution**

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'echo "pretend build step running"'
            }
        }
        stage('Test') {
            steps {
                echo 'Running the tests...'
                sh 'exit 1'          // step 3: deliberately fail here
            }
        }
        stage('Package') {           // step 2: the third stage
            steps {
                echo 'Packaging the application...'
            }
        }
    }
}
```

With `sh 'exit 1'` in Test, the Stage View shows: **Build = green**, **Test = red**, **Package = grey/skipped** — it never ran. Change `sh 'exit 1'` back to `sh 'echo "pretend tests passing"'` and all three go green.

---

**Challenge**

*Direct* students, **in pairs**, to write a declarative pipeline that does the following:

* Has **three** stages, named `Checkout`, `Test` and `Deploy`
* Each stage prints a message saying what it's doing
* The `Deploy` stage additionally prints **which build number** it is deploying, using Jenkins' built-in `$BUILD_NUMBER`
* The `Test` stage must run a real shell command (using `sh`), not just an `echo`
* **OPTIONAL** — make the pipeline fail in `Test` on **even-numbered builds only**, so you can watch it alternate between green and red on successive runs

*Provide* this example Console Output for a successful third build as an aid:

```
Checking out the code...
Running the tests...
tests passed
Deploying build number 3
```

*Grant* students ~10 minutes.

Hints to offer if they're stuck: remember **double quotes** are needed for `${BUILD_NUMBER}` to be substituted by Groovy. For the optional part, the shell has a modulo operator — `$((BUILD_NUMBER % 2))` gives 0 for even numbers.

**SOLUTION**

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out the code...'
            }
        }

        stage('Test') {
            steps {
                echo 'Running the tests...'
                sh 'echo "tests passed"'
            }
        }

        stage('Deploy') {
            steps {
                // Double quotes so Groovy substitutes the build number in
                echo "Deploying build number ${BUILD_NUMBER}"
            }
        }
    }
}
```

**SOLUTION (optional extension — fails on even builds)**

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out the code...'
            }
        }

        stage('Test') {
            steps {
                echo 'Running the tests...'
                // $((BUILD_NUMBER % 2)) is 0 on even builds.
                // The shell 'if' and 'exit 1' are exactly what you wrote in the bash session.
                sh '''
                    if [ $((BUILD_NUMBER % 2)) -eq 0 ]; then
                      echo "Even build - failing on purpose"
                      exit 1
                    else
                      echo "tests passed"
                    fi
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying build number ${BUILD_NUMBER}"
            }
        }
    }
}
```

Two things to draw out when revealing this:
* The **triple single quotes** `'''...'''` let you write a multi-line shell script inside a single `sh` step. Very common in real pipelines.
* The `if`, the `[ ]` test brackets, `-eq`, and `exit 1` are all **exactly** what students wrote in the bash session — nothing new. A pipeline is mostly a wrapper around the shell skills they already have.

Run it three or four times and watch it alternate red, green, red, green — and notice `Deploy` is skipped every time `Test` goes red.

<br>
<br>

*(Take a 1 hour lunch break here.)*

<br>
<br>
### 14:00–14:45 — Anatomy of a Pipeline: Stages, Environment, Post, Parallel
*(Talk + short exercise)*

Welcome back. This morning you saw the *shape* of a pipeline. Now let's fill out the toolkit — the handful of building blocks that show up in almost every real `Jenkinsfile`. I'll show each one; you'll add them to your `hello-pipeline` as we go.

**Environment variables — name a value once, reuse it**

```groovy
pipeline {
    agent any

    environment {
        APP_NAME = 'countries-api'
        GREETING = 'Building'
    }

    stages {
        stage('Build') {
            steps {
                echo "${GREETING} ${APP_NAME}..."
            }
        }
    }
}
```

Unpacking the new bits:
- `environment { }` — a block that defines variables available to **every** stage in the pipeline
- `APP_NAME = 'countries-api'` — assignment. Note Groovy **does** use spaces around `=`, unlike bash
- `echo "${GREETING} ${APP_NAME}..."` — **double quotes**, because we want the values substituted in. Single quotes here would print the literal text `${GREETING} ${APP_NAME}`

Change `APP_NAME` in one place and it updates everywhere it's used.

**ASK** <br>
Where have you already used this exact idea — declaring a value once at the top and referring to it by name throughout? <br>
**ANSWER** <br>
Bash variables in your scripts, and (coming up) Terraform `variable` blocks. Same idea, slightly different punctuation each time. Once you see it's the *same* concept, each new tool's version is trivial.

**Parameters — let a human feed input at build time**

```groovy
pipeline {
    agent any

    parameters {
        string(name: 'ENVIRONMENT', defaultValue: 'dev', description: 'Which environment to target')
    }

    stages {
        stage('Deploy') {
            steps {
                echo "Deploying to ${params.ENVIRONMENT}..."
            }
        }
    }
}
```

- `parameters { }` — declares inputs the user supplies when starting a build
- `string(name: ..., defaultValue: ..., description: ...)` — this is a **function call with named arguments**. The `name:` bits are labels, so the order doesn't matter and it's self-documenting. Other types exist too: `booleanParam`, `choice`
- `${params.ENVIRONMENT}` — parameters live under `params.`, so you must write the prefix

Once a pipeline has `parameters`, the job's button changes from **Build Now** to **Build with Parameters**, and Jenkins prompts you for a value each time.

**NOTE FOR TRAINERS** <br>
Flag this gotcha before students hit it: the **first** build after adding a `parameters` block still runs with defaults and shows a plain "Build Now" — Jenkins has to run the pipeline once to *discover* the parameters exist. From the second build onwards you get the prompt. Students reliably think they've done it wrong. <br>
**END OF NOTE**

**The `post` section — do something after, based on the outcome**

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
    }

    post {
        success {
            echo 'It worked!'
        }
        failure {
            echo 'It broke — someone needs to look at this.'
        }
        always {
            echo 'This runs every time, pass or fail.'
        }
    }
}
```

The `post` block sits **outside** `stages` and runs *after* all of them finish. Inside it:
- `success { }` — runs only if everything passed
- `failure { }` — runs only if something broke
- `always { }` — runs no matter what (good for cleanup, or publishing test reports)
- (`unstable` and `changed` also exist — `changed` fires only when the result differs from the previous build)

**ASK** <br>
In a real team, what would you actually want in that `failure` block instead of an `echo`? <br>
**ANSWER** <br>
A notification — a Slack message to the team channel, an email, a page to whoever's on call, maybe auto-opening a ticket. The whole point of CI is *fast feedback*, and a failure nobody's told about is slow feedback. Jenkins has plugins that plug straight into this `failure` block for exactly that.

**Parallel stages — run independent work at the same time**

```groovy
stages {
    stage('Checks') {
        parallel {
            stage('Unit Tests') {
                steps { echo 'Running unit tests...' }
            }
            stage('Linting') {
                steps { echo 'Running the linter...' }
            }
        }
    }
}
```

The `parallel { }` block replaces the `steps { }` block of a stage, and contains **nested stages** that all run at once. The outer stage isn't finished until every branch inside it completes.

**ASK** <br>
Why would we run unit tests and linting in parallel rather than one after the other? <br>
**ANSWER** <br>
They don't depend on each other — the linter's result doesn't change whether the tests should run, and vice versa. Running them at the same time gives faster overall feedback with no change to the outcome. Faster feedback is the entire game in CI, so anything independent is a candidate for parallel.

**One more idea to *name*, not build: Multibranch**

There's a job type called a **Multibranch Pipeline** that scans a whole repo and automatically creates a pipeline for *every branch and every open Pull Request* that has a `Jenkinsfile` — and cleans them up when the branch is deleted.

**ASK** <br>
Why is that a perfect fit for the Pull Request workflow we discussed in the Azure session? <br>
**ANSWER** <br>
Because every feature branch and every open PR automatically gets its own build and test run, with zero manual job-creation. That's exactly the mechanism behind "run the tests on every PR before we allow a merge" — and later, "run `terraform plan` on every PR so a reviewer can see what would change." We won't wire it up today, but know the term.

**HANDS ON (20 min)** <br>
*(All in the Jenkins UI — the `hello-pipeline` job → **Configure**)*

Take your `hello-pipeline` and, one at a time, add:
1. An `environment` block with an `APP_NAME`, and use it inside a stage's `echo`. Make sure you use **double quotes**.
2. A `post` section with `success`, `failure` and `always` blocks. Then make a step fail (`sh 'exit 1'`) and confirm `failure` and `always` run but `success` does not. Fix it and confirm the opposite.
3. A `parameters` block with a `string` parameter, and echo `params.YOURPARAM` in a stage. Build **twice**, and notice the button change to **Build with Parameters** on the second run.
4. (Stretch) Convert your `Test` stage into a `parallel` block containing two nested stages.
**END OF NOTE**

**Solution**

All four combined into one pipeline:

```groovy
pipeline {
    agent any

    // 1. Variables available to every stage
    environment {
        APP_NAME = 'countries-api'
    }

    // 3. Input supplied when the build starts
    parameters {
        string(name: 'ENVIRONMENT', defaultValue: 'dev', description: 'Which environment to target')
    }

    stages {
        stage('Build') {
            steps {
                echo "Building ${APP_NAME}..."          // double quotes = value substituted
            }
        }

        // 4. Stretch: two checks running at the same time
        stage('Checks') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        echo 'Running unit tests...'
                        sh 'echo "unit tests passed"'
                    }
                }
                stage('Linting') {
                    steps {
                        echo 'Running the linter...'
                        sh 'echo "no lint errors"'
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying ${APP_NAME} to ${params.ENVIRONMENT}"
            }
        }
    }

    // 2. Runs after all stages, based on the outcome
    post {
        success {
            echo "${APP_NAME} build ${BUILD_NUMBER} succeeded"
        }
        failure {
            echo "${APP_NAME} build ${BUILD_NUMBER} FAILED"
        }
        always {
            echo 'Pipeline finished — pass or fail.'
        }
    }
}
```

To test step 2, temporarily change one step to `sh 'exit 1'` and rebuild. In the Console Output you'll see the **`failure`** and **`always`** messages, but **not** `success`. Change it back and you'll see `success` and `always`, but not `failure`.

<br>
<br>

### 14:45–15:00 — Connecting Jenkins to Git, and Triggering Builds
*(Talk + demo)*

Our `Jenkinsfile` is still pasted into the Jenkins UI — which has the *exact same* "not really code" problem as the Freestyle job. Let's fix the concept now, so the capstone can do it for real.

**Pulling the `Jenkinsfile` from Git**

Instead of pasting the script, we point Jenkins at a Git repo and tell it "the `Jenkinsfile` is in there".

*(In the Jenkins UI — a Pipeline job → **Configure**)*
- Scroll to the **Pipeline** section
- Change **Definition** from `Pipeline script` to **Pipeline script from SCM**

*(SCM stands for "Source Control Management" — Jenkins' generic term for Git and its older cousins.)*

- **SCM**: `Git`
- **Repository URL**: your fork of the starter repo
- **Branch Specifier**: `*/main` — the `*/` prefix is Jenkins shorthand meaning "the `main` branch from whichever remote", so `*/main` matches `origin/main`
- **Script Path**: `Jenkinsfile` — the path to the file within the repo. The default assumes it's at the repo root

Now the pipeline definition lives in Git: versioned, reviewable in a Pull Request, and identical for anyone who runs it. This is the whole reason pipeline-as-code beats clicking.

**Handling secrets — the Credentials store**

To pull a private repo (or push to Docker Hub later), Jenkins needs secrets. We never paste those into a config box or, worse, into the `Jenkinsfile` in Git. We use Jenkins' **Credentials** store.

*(In the Jenkins UI — Dashboard)*
- **Manage Jenkins → Credentials → System → Global credentials (unrestricted) → Add Credentials**
- We'll set one up properly together in the capstone

**ASK** <br>
Why store a token in the Credentials vault instead of just typing it into the repository URL field, or the `Jenkinsfile`? <br>
**ANSWER** <br>
The vault keeps the secret out of both the job config *and* out of Git, it automatically **masks** the value if it would otherwise show up in a build log, and it lets you rotate the secret in one place without editing anything else. A token committed to Git is a genuine security incident — the vault exists so that never has to happen.

**Triggering builds automatically**

The magic of CI is that nobody clicks "Build". A change to Git triggers it. Two ways:

- **Poll SCM** — Jenkins checks the repo on a schedule (cron syntax again) and builds only if something changed. Simple, works everywhere, but there's a delay and it's a bit wasteful
- **Webhook** — GitHub *tells* Jenkins the instant a push happens. Instant, efficient, and how real setups do it

**ASK** <br>
Between polling every minute and a webhook, which is better, and why? <br>
**ANSWER** <br>
The webhook. Polling means Jenkins repeatedly asks "anything changed yet?" when the answer is almost always no — wasted checks, plus a delay of up to the polling interval. A webhook means GitHub notifies Jenkins the moment something actually happens: no waste, no delay.

**NOTE FOR TRAINERS** <br>
Here's the local-laptop reality: a webhook from GitHub **cannot** reach `http://localhost:8080` on a student's machine, because GitHub is on the public internet and their Jenkins isn't. The proper fix is a tunnel like [ngrok](https://ngrok.com/), which is a great optional stretch. For the guaranteed-to-work classroom path, we use **Poll SCM** in the capstone — it needs no networking magic and demonstrates the same "builds trigger themselves from Git" principle. Teach the webhook as the concept and the goal; use polling as the reliable hands-on. <br>
**END OF NOTE**

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>
### 15:15–16:45 — Capstone: A Real Test → Build → Push Pipeline
*(Exercise — 1 hour 30 minutes)*

This is the main event, and it ties the whole day — and the whole course so far — together. You're going to build one pipeline that, automatically, on a push to Git: **checks out a Node app, installs it and runs its tests, builds a Docker image, and pushes that image to Docker Hub.** This is the genuine shape of how real teams ship software.

Work individually or in pairs. Get the core pipeline green first; the stretch goals are there if you race ahead. Take your time — this is meant to fill the session.

---

#### Part 0 (≈10 min) — Get the app and a Docker Hub token ready

**1. Clone your fork.**

*(Run from `~/jenkins-training`)*
```bash
git clone https://github.com/<your-username>/<your-fork>.git
cd <your-fork>
```

*(Run from `~/jenkins-training/<your-fork>`)*
```bash
ls
cat package.json
cat Dockerfile
```

You have a small Node app with a `package.json`, a trivial test (`npm test` passes), and a `Dockerfile`. Have a proper look at those three files so you know what the pipeline will be acting on. *(If you'd rather, use the `countries` app you scaffolded in the bash session — it already has a `Dockerfile`.)*

**2. Get a Docker Hub access token.**

*(In your browser — [hub.docker.com](https://hub.docker.com))*
- **Account Settings → Security → New Access Token**
- Give it a description, set permissions to **Read & Write**, click **Generate**
- **Copy it now** — Docker Hub only shows it once

This token is what the pipeline pushes with — **never** your account password. A token can be revoked on its own without changing your login.

#### Part 1 (≈15 min) — Store your Docker Hub credentials in Jenkins

*(In the Jenkins UI — Dashboard)*
- **Manage Jenkins → Credentials → System → Global credentials (unrestricted) → Add Credentials**
- **Kind**: `Username with password`
- **Username**: your Docker Hub username
- **Password**: the access token you just generated
- **ID**: `dockerhub-credentials` ← **this exact string**, because the `Jenkinsfile` refers to it by ID
- **Description**: anything helpful, e.g. "Docker Hub push token"
- Click **Create**

**Checkpoint question — ask yourself:** why did we put the token *here* and not in the `Jenkinsfile` we're about to write and push to Git?

#### Part 2 (≈35 min) — Write the pipeline

*(Run from `~/jenkins-training/<your-fork>`)*
- Run: `touch Jenkinsfile`
- Then open it: `code Jenkinsfile`

Paste this in, replacing `your-dockerhub-username`:

```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME = "your-dockerhub-username/hello-pipeline-app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Building ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Install & Test') {
            agent {
                docker { image 'node:20' }
            }
            steps {
                sh 'npm install'
                sh 'npm test'
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
                }
            }
        }
    }

    post {
        success {
            echo "Done! Pushed ${IMAGE_NAME}:${IMAGE_TAG} to Docker Hub."
        }
        failure {
            echo 'Pipeline failed — check which stage went red in the Stage View.'
        }
    }
}
```

Let me walk through the new pieces, because each connects to something you already know:

**`checkout scm`** — a built-in step meaning "check out the same repository this `Jenkinsfile` came from". You don't have to specify the URL again; Jenkins already knows it. `scm` here is a variable Jenkins provides holding your source-control config.

**A stage with its own `agent`:**
```groovy
stage('Install & Test') {
    agent {
        docker { image 'node:20' }
    }
```
The pipeline's top-level `agent any` sets the default, but **an individual stage can override it**. Here, this stage runs *inside a freshly-started `node:20` container*. Jenkins pulls the image, starts it, mounts the workspace in, runs the steps, then throws the container away. That's huge: the stage gets a clean, correct Node environment every time, and you never have to install Node on the Jenkins host. It's the "disposable, identical environment per build" idea from the Docker session, applied to CI.

**Tagging with the build number:**
```groovy
IMAGE_TAG  = "${BUILD_NUMBER}"
```
Every image gets tagged with the build number that produced it, so you can trace any running container back to an exact build. That's the `$BUILD_NUMBER` you met this morning, now doing real work.

**Note the two quoting styles in play.** In `environment`, we use Groovy double quotes so `${BUILD_NUMBER}` is substituted. But in `sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'` we use **single** quotes with `$VAR` — here the *shell* does the substitution, because Jenkins exports everything in `environment` as real shell environment variables. Both work; single quotes in `sh` steps are actually safer, because the value never gets baked into the command Groovy builds (which matters for secrets — see below).

**`withCredentials` — the vault, used properly:**
```groovy
withCredentials([usernamePassword(
    credentialsId: 'dockerhub-credentials',
    usernameVariable: 'DOCKER_USER',
    passwordVariable: 'DOCKER_PASS'
)]) {
    // secrets exist ONLY inside this block
}
```
- `credentialsId:` — matches the **ID** you set in Part 1
- `usernameVariable:` / `passwordVariable:` — the names the values get bound to
- Everything inside the `{ }` can use `$DOCKER_USER` and `$DOCKER_PASS`; **outside the block they don't exist**
- Jenkins automatically **masks** these values in the console log — you'll see `****` instead of the token

**`--password-stdin`:**
```bash
echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
```
That's the pipe from your bash session. Rather than putting the password on the command line (where it could appear in process listings or logs), we pipe it into `docker login` via standard input. Standard security practice.

Now commit and push:

*(Run from `~/jenkins-training/<your-fork>`)*
```bash
git add Jenkinsfile
git commit -m "Add build and push pipeline"
git push origin main
```

#### Part 3 (≈20 min) — Point Jenkins at it and run it

*(In the Jenkins UI — Dashboard)*
1. **New Item** → name it `build-and-push` → select **Pipeline** → **OK**
2. Scroll to the **Pipeline** section:
   - **Definition**: `Pipeline script from SCM`
   - **SCM**: `Git`
   - **Repository URL**: your fork's HTTPS URL
   - **Branch Specifier**: `*/main`
   - **Script Path**: `Jenkinsfile`
3. Scroll up to **Build Triggers** → tick **Poll SCM** → **Schedule**: `H/2 * * * *` (checks every couple of minutes)
4. **Save**, then **Build Now** for the first run
5. Watch the **Stage View** march through Checkout → Install & Test → Build Image → Push Image

*(In your browser — [hub.docker.com](https://hub.docker.com))*
6. When it's green, go to your Docker Hub repositories and confirm the image is there, tagged with the build number

Now the moment that matters:

*(Run from `~/jenkins-training/<your-fork>`)*
```bash
echo "A trivial change" >> README.md
git add README.md
git commit -m "Trigger the pipeline"
git push origin main
```

Then **wait**. Within a couple of minutes, Poll SCM kicks off a build **on its own**. That — a build you didn't start, triggered purely by a Git push — is CI working. Sit with it; that's the whole point of the day.

**ASK** *(mid-capstone checkpoint, ~16:15)* <br>
Look at the order of your stages: Test comes *before* Build and Push. Why does that order matter — what does putting Test first actually protect you from? <br>
**ANSWER** <br>
If the tests fail, the pipeline stops right there and **never builds or pushes the image**. A broken version can't reach Docker Hub, because the gate caught it first. That's the "stop the line before the bad thing ships" principle — the same reason your bash `containerise` script checked exit codes. Ordering stages so the cheap checks fail fast, before the expensive or irreversible steps, is a core pipeline design habit.

#### Part 4 — Stretch goals

If you've got a green, self-triggering pipeline, level it up:

1. **Prove the gate works.** Add a stage *before* Build Image that deliberately fails (`sh 'exit 1'`), push it, and confirm nothing gets built or pushed. Then remove it. Worth doing — *seeing* the pipeline refuse to ship broken code is more convincing than being told it will.
2. **Add a `latest` tag.** In the Push stage, also tag and push `$IMAGE_NAME:latest` alongside the numbered tag, so there's always a "newest" image.
3. **Parallelise the checks.** Add a linting step and run it in `parallel` with the tests.
4. **Notify on failure.** Enrich the `failure` block to print the image name and the build URL, then read about the Slack/email plugins that would do this for real.
5. **Webhook instead of polling** (advanced): install `ngrok`, expose your local Jenkins, and set up a GitHub webhook so pushes trigger builds *instantly*.

**Solution**

**Stretch 1 — proving the gate.** Insert this stage between `Install & Test` and `Build Image`:

```groovy
stage('Deliberate Failure') {
    steps {
        echo 'About to fail on purpose...'
        sh 'exit 1'
    }
}
```

Push it, and the Stage View shows Checkout ✅ → Install & Test ✅ → Deliberate Failure ❌ → **Build Image and Push Image never run**. Check Docker Hub: no new tag. Delete the stage and push again to go green.

**Stretch 2 — a `latest` tag as well.** Replace the Push Image stage's steps:

```groovy
stage('Push Image') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'dockerhub-credentials',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )]) {
            sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
            sh 'docker push $IMAGE_NAME:$IMAGE_TAG'

            // Point 'latest' at this same image, then push that tag too
            sh 'docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest'
            sh 'docker push $IMAGE_NAME:latest'
        }
    }
}
```

**Stretch 3 — parallel checks.** Replace the `Install & Test` stage with:

```groovy
stage('Checks') {
    agent {
        docker { image 'node:20' }
    }
    steps {
        sh 'npm install'
    }
}

stage('Test & Lint') {
    agent {
        docker { image 'node:20' }
    }
    parallel {
        stage('Unit Tests') {
            steps {
                sh 'npm test'
            }
        }
        stage('Lint') {
            steps {
                sh 'echo "linting..." && npm run lint || echo "no lint script configured"'
            }
        }
    }
}
```

*(Note the `|| echo ...` — the `||` from your bash session, used here so a missing `lint` script doesn't fail the build while you're experimenting.)*

**Stretch 4 — a more useful failure message:**

```groovy
post {
    success {
        echo "Done! Pushed ${IMAGE_NAME}:${IMAGE_TAG} to Docker Hub."
    }
    failure {
        echo "FAILED building ${IMAGE_NAME}:${IMAGE_TAG}"
        echo "Full log: ${BUILD_URL}console"
    }
    always {
        echo "Build ${BUILD_NUMBER} finished with status: ${currentBuild.currentResult}"
    }
}
```

`currentBuild.currentResult` is a built-in Jenkins object holding this build's outcome — `SUCCESS`, `FAILURE`, or `UNSTABLE`.

**Stretch 5 — ngrok webhook (outline).**

*(Run from `~/jenkins-training`)*
```bash
ngrok http 8080
```
Copy the `https://....ngrok-free.app` address it prints. Then *(in your browser, on GitHub)*: **your fork → Settings → Webhooks → Add webhook**, with **Payload URL** = `https://....ngrok-free.app/github-webhook/` (the trailing slash matters), **Content type** = `application/json`, **Just the push event**. Finally *(in the Jenkins UI)*: **Configure** the job → **Build Triggers** → untick Poll SCM, tick **GitHub hook trigger for GITScm polling**. Push a commit and the build starts within a second or two.

#### What to show at 16:45

Be ready to demonstrate: push a change, and (via Poll SCM) watch a build start on its own, go green through all stages, and result in a freshly-tagged image in your Docker Hub. And be ready to explain, in your own words, *why* the Test stage comes before the Push stage.

<br>
<br>

### 16:45–17:00 — Wrap-up & Q&A

Let's pull the day together.

You started this morning with a definition — CI/CD is about replacing scary, manual, occasional releases with automatic, safe, frequent ones — and a mental model: a pipeline is an assembly line of stages, where each one has to pass before the next runs, and the line stops the moment something breaks.

Then you made it real. You installed Jenkins on your own laptop, ran your first job, and watched the crucial shift from clicking a job together in the UI to describing it as a `Jenkinsfile` in Git — reviewable, versioned, repeatable, the same "as code" principle behind every tool in this course. And in the capstone you built the genuine article: a pipeline that, on a git push, tests an app, builds an image, and ships it to a registry, all on its own — refusing to push anything if the tests fail.

And notice how much of today was actually just **bash skills wearing a Jenkins hat**. Every `sh` step, every `exit 1`, every `||`, every pipe into `docker login` — that was last session's material. The pipeline is the wrapper; the shell is still doing the work.

**ASK** <br>
Look at the shape of that capstone pipeline: checkout, then a stage that tests, then a stage that changes something real, then a stage that ships it — with credentials pulled from the vault and a `post` block reporting the outcome. If we wanted this *same* pipeline to run **Terraform against Azure** instead of building a Docker image, how much would actually have to change? <br>
**ANSWER** <br>
Structurally, almost nothing. The credentials pattern is identical — you'd store an Azure Service Principal's details in the vault instead of a Docker Hub token. The `sh` steps would run `terraform init`, `terraform plan`, `terraform apply` instead of `docker build`/`docker push`. The stages, the gating, the `post` block — all the same. That's not a coincidence: it's exactly the next session, and everything you built today is the groundwork it stands on.

Where this sits in the course:
- **Today** — you can explain and build a pipeline, and run Jenkins yourself
- **Terraform & IaC on Azure** — the same pipeline shape, now running Terraform to provision real infrastructure, with a human approval gate before anything is applied
- **Kubernetes** — running and scaling the images your pipeline now builds
- **Integration** — everything woven together: a push to Git flowing through test, build, provision and deploy, automatically and safely

**Q&A** — take remaining questions.

**Before everyone leaves** — remind them how to stop and restart Jenkins without losing anything:

*(Run from anywhere)*
```bash
docker stop jenkins     # frees up your laptop's resources
docker start jenkins    # everything is still there, thanks to the named volume
```

<br>
<br>

### Exercise (take-home / reinforcement)

Before the next session, individually or in pairs:

1. From scratch, in a brand new folder, get Jenkins running and build a two-stage pipeline **from memory** — no notes. Rebuilding is the fastest way to make it stick.
2. Take the capstone pipeline and add a **`Package` stage** between Test and Build Image that prints the version being built — practising adding a stage cleanly.
3. Add a `parameters` block letting you pass in the image tag manually, defaulting to the build number, and use it in the Build and Push stages.
4. Write a short paragraph, in your own words, explaining the difference between Continuous Delivery and Continuous Deployment, with an example of a team that would prefer each.
5. (Stretch) Set up `ngrok` and a GitHub webhook so a push triggers your pipeline **instantly**, and compare the feel against Poll SCM.
6. (Stretch) Read the [Jenkins Pipeline syntax docs](https://www.jenkins.io/doc/book/pipeline/syntax/) and find one directive we didn't cover today (e.g. `when`, `options`, `tools`) — work out what it does and where you'd use it.

**Solution** *(for the guided ones — 2, 3, 6)*

**Take-home 2** — the `Package` stage, inserted between Test and Build Image:

```groovy
stage('Package') {
    steps {
        echo "Packaging ${IMAGE_NAME} version ${IMAGE_TAG}"
        sh 'echo "Contents about to be packaged:" && ls -la'
    }
}
```

**Take-home 3** — a parameterised image tag. Note the `?:` operator, Groovy's "Elvis operator": it means "use the left value, **or** the right one if the left is empty" — the same idea as bash's `${2:-default}`.

```groovy
pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_TAG_OVERRIDE', defaultValue: '', description: 'Leave blank to use the build number')
    }

    environment {
        IMAGE_NAME = "your-dockerhub-username/hello-pipeline-app"
        IMAGE_TAG  = "${params.IMAGE_TAG_OVERRIDE ?: BUILD_NUMBER}"
    }

    stages {
        stage('Build Image') {
            steps {
                echo "Building ${IMAGE_NAME}:${IMAGE_TAG}"
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }
    }
}
```

**Take-home 6** — the most useful directive we didn't cover is `when`, which makes a stage **conditional**:

```groovy
stage('Deploy to Production') {
    when {
        branch 'main'      // only run this stage on the main branch
    }
    steps {
        echo 'Deploying to production...'
    }
}
```

You'd use it so that feature branches get built and tested, but only `main` actually deploys — a very common real-world pattern.

<br>
<br>

### Conclusion

- **Inform** students this marks the end of the Jenkins & CI/CD Pipelines session
- **Reinforce** that every mechanism the next session needs — credentials, stages, gating, `post` blocks, pipeline-as-code — already exists in what they built today; only the commands inside the stages change
- **Preview** the next session directly: the same pipeline shape, running **Terraform against Azure**, with a manual approval gate before `apply`
- **Direct** students to the take-home exercises and to the [Jenkins Pipeline documentation](https://www.jenkins.io/doc/book/pipeline/) for anyone wanting to go deeper

---

[Back](./README.md)

---
