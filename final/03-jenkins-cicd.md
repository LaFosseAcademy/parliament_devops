# Session 3 — Jenkins & CI/CD Pipelines — Trainer Script

A full day taking trainees from "I've heard the word pipeline" to "I've built one that tests, builds and ships a Docker image automatically". The emphasis throughout is on **what a pipeline actually is, what it's for, and why teams rely on them** — the mechanics of Jenkins are the vehicle, not the point. This is written as a script — read it out, adapt where it feels natural, but the intent is that you could deliver this cold from the page.

---

## 📦 STARTER CODE — put this in the repo before training

Everything here goes into **`/jenkins/intro-to-jenkins/starter-code`** before the session.

Two principles: **ship anything that's pure transcription**, and **withhold anything that's the learning**. The Node app and the Jenkins `Dockerfile` are setup — students gain nothing from typing them. The `Jenkinsfile` is the entire point of the day, so it lives in `completed-code/` as catch-up only.

<br>

**`README.md`**
```markdown
# Jenkins & CI/CD — Starter Code

## What's here

- **jenkins-image/Dockerfile** — a Jenkins image that can also run Docker.
  We build this first thing; you don't need to write it.

- **app/** — a tiny Node app with a passing test and its own Dockerfile.
  This is what the pipeline will test, build and push.

- **pipeline-glossary.md** — you fill this in. Every term we meet today,
  in your own words.

- **completed-code/** — the finished Jenkinsfiles.
  Catch-up only. Writing these yourself IS the session.

## Before we start

    docker --version      # must respond
    docker run hello-world # must succeed

If either fails, tell a trainer before 09:15.
```

<br>

**`jenkins-image/Dockerfile`**
```dockerfile
# A Jenkins image that can also run Docker commands.
# The stock jenkins/jenkins:lts image has no docker CLI inside it,
# which is why we build our own.

FROM jenkins/jenkins:lts-jdk17

USER root

RUN apt-get update \
    && apt-get install -y docker.io \
    && rm -rf /var/lib/apt/lists/*

USER jenkins
```

<br>

**`app/package.json`**
```json
{
  "name": "hello-pipeline-app",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "test": "node test.js"
  },
  "dependencies": {
    "express": "^4.19.2"
  }
}
```

<br>

**`app/index.js`**
```javascript
const express = require("express");

const app = express();
const PORT = process.env.PORT || 3000;

app.get("/", (req, res) => {
  res.json({ message: "Hello from the pipeline", version: "1.0.0" });
});

app.get("/health", (req, res) => {
  res.json({ status: "ok" });
});

if (require.main === module) {
  app.listen(PORT, () => console.log(`Listening on port ${PORT}`));
}

module.exports = app;
```

<br>

**`app/test.js`** *(deliberately dependency-free, so `npm test` works with nothing installed but Express)*
```javascript
// A deliberately tiny test. No test framework, so there's nothing
// extra to install and nothing to go wrong in the pipeline.
// In a real project this would be Jest, Vitest or Mocha.

const assert = require("assert");
const app = require("./index");

let failures = 0;

function check(name, fn) {
  try {
    fn();
    console.log(`  PASS  ${name}`);
  } catch (err) {
    console.log(`  FAIL  ${name}: ${err.message}`);
    failures++;
  }
}

console.log("Running tests...");

check("app is exported", () => {
  assert.ok(app, "app should be defined");
});

check("app has routes registered", () => {
  assert.ok(app._router, "app should have a router");
});

if (failures > 0) {
  console.log(`\n${failures} test(s) failed`);
  process.exit(1);        // NON-ZERO exit code = the pipeline will fail
}

console.log("\nAll tests passed");
process.exit(0);          // ZERO = success
```

<br>

**`app/Dockerfile`**
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

<br>

**`app/.dockerignore`**
```
node_modules
.git
```

<br>

**`pipeline-glossary.md`** *(students fill this in — it's one of today's activities)*
```markdown
# My Pipeline Glossary

Fill this in as we go. **Your own words** — if you can't explain it
simply, you haven't understood it yet.

| Term | What it means (in MY words) |
|---|---|
| Continuous Integration | |
| Continuous Delivery | |
| Continuous Deployment | |
| Pipeline | |
| Stage | |
| Step | |
| Agent | |
| Executor | |
| Workspace | |
| Artifact | |
| Freestyle job | |
| Pipeline job | |
| Jenkinsfile | |
| Declarative pipeline | |
| Credentials store | |
| Poll SCM | |
| Webhook | |
| Multibranch pipeline | |

## The one-sentence version

If a colleague asked "what's a pipeline?", I'd say:

> _________________________________________________
```

<br>

**`completed-code/`** — put the finished `Jenkinsfile` versions here (they appear in the Solution blocks below). Mark clearly as catch-up only.

<br>

---

## 💬 SLACK SNIPPETS — queue these before you start

Have these in a draft message. **Post reactively**, at the point in the session where they're needed. Long Groovy blocks especially — transcribing a `Jenkinsfile` by hand teaches nothing except typo tolerance, and a single missing brace costs a student twenty minutes.

| # | When | What to post |
|---|---|---|
| 1 | 09:05 | The pre-flight Docker check |
| 2 | 10:15 | The `docker build` + `docker run` commands for Jenkins |
| 3 | 10:35 | Jenkins troubleshooting commands |
| 4 | 11:15 | The Freestyle job's shell steps |
| 5 | 12:15 | The first two-stage `Jenkinsfile` |
| 6 | 14:00 | The `environment` / `parameters` / `post` / `parallel` blocks |
| 7 | 15:30 | **The full capstone `Jenkinsfile`** — post this, don't make them type it |
| 8 | 16:45 | The stop/start commands |

Each is marked **💬 SLACK** in the body below.

---

## Organisation

### Audience

Trainees who have completed **Session 1 (Azure fundamentals)** and **Session 2 (bash scripting & automation)**, and who have covered **Docker**.

They are experienced developers: **Node, Express, REST, MVC, SQL, Git, GitHub, Docker, frontend, unit and integration testing**. Three of those matter enormously today and should be leaned on constantly:

- **They already write tests.** CI is not a new idea to them — it's their existing test suite, run by someone other than them, automatically. Frame it that way and half the session lands instantly
- **They already use pull requests.** The "review before it's live" instinct is fully formed; today just extends it to the build process
- **They already build Docker images.** The capstone is their `docker build` and `docker push`, with a trigger on the front

What's genuinely new: **Jenkins itself**, **Groovy syntax** (assume zero exposure), and — most importantly — the *organisational* idea that releasing software should be boring and frequent rather than rare and terrifying.

**NOTE FOR TRAINERS** <br>
The risk with this room is that the *concepts* land in ten minutes and they get bored — "yes, obviously you should run tests automatically, we know". Don't linger on selling CI/CD; they'll buy it immediately. Spend the reclaimed time on the two things they genuinely can't guess: **Groovy's quoting rules** and **why the credentials store exists rather than environment variables**. Those are where the day actually goes wrong for people. <br>
**END OF NOTE**

### How this document is laid out

Today you switch constantly between **a terminal** and **a browser**, which is exactly where trainees get lost. Every instruction block is labelled:

- *(Run from `~/jenkins-training`)* — a **terminal** command, in that folder
- *(In the Jenkins UI — Dashboard)* — a **browser** action, starting from that screen
- Navigation is written as breadcrumbs: **Manage Jenkins → Credentials → Add Credentials**

The folders we use today:

| Folder | What it's for |
|---|---|
| `~/jenkins-training` | Where we build the custom Jenkins image and run Docker commands |
| `~/jenkins-training/<your-fork>` | Your cloned fork — where the `Jenkinsfile` lives |

**Every activity has a `**Solution**` block** immediately afterwards.

### Duration & schedule

Full day, **09:00–17:00**, with a one-hour lunch and two 15-minute breaks.

**Hands-on time today: ~3 hours 35 minutes** across seven activities, every one with a solution in this document.

| Time | Section | Activity time |
|---|---|---|
| 09:00–09:15 | Welcome & recap | |
| 09:15–10:00 | What CI/CD is, what pipelines are *for*, and why | |
| **10:00–10:15** | **Break** | |
| 10:15–11:15 | Installing & touring Jenkins on your laptop | **35 min** |
| 11:15–12:15 | Your first job: watching Jenkins build something | **40 min** |
| 12:15–13:00 | Pipeline as code: your first `Jenkinsfile` | **20 min + challenge** |
| **13:00–14:00** | **Lunch** | |
| 14:00–14:45 | Anatomy of a pipeline: stages, environment, post, parallel | **20 min** |
| 14:45–15:00 | Connecting Jenkins to Git, and triggering builds | |
| **15:00–15:15** | **Break** | |
| 15:15–16:45 | Capstone: a real test → build → push pipeline | **90 min** |
| 16:45–17:00 | Wrap-up & Q&A | |

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
  - **Docker Desktop** installed and **running** — we run Jenkins itself as a container, so this is non-negotiable
  - On **Windows**: Docker Desktop using the **WSL2 backend** (its default), and **Git Bash** installed so the multi-line commands paste cleanly
  - A **GitHub account**, with the starter repo **forked** and cloned locally
  - A free **[Docker Hub](https://hub.docker.com)** account — needed for the afternoon capstone
  - At least a few GB of free disk — Jenkins plus a couple of images adds up
  - No Terraform or Azure needed today — that's Session 5, which today sets up perfectly

**NOTE FOR TRAINERS — the one thing that will eat your morning** <br>
The single biggest time-sink in a room of local laptops is Docker Desktop not installed or not running. Send a setup checklist the day before and have students confirm `docker run hello-world` works **before** they arrive. Ten minutes of prep saves an hour of firefighting. <br>
Second-biggest: disk space. Jenkins, the `node:20` image and a couple of builds will consume several GB. Someone with a nearly-full laptop will fail mysteriously at 15:40. <br>
**END OF NOTE**

## Learning objectives

- **Explain** what CI/CD means, what a pipeline is, and *why* teams use them instead of releasing by hand
- **Install** and run a working Jenkins instance locally on Windows or Mac
- **Create and run** a Freestyle job and watch a build execute
- **Read and write** declarative `Jenkinsfile` syntax — blocks, stages, steps, string interpolation, `sh`
- **Use** environment variables, parameters, `post` actions and parallel stages
- **Explain** the difference between pipeline-in-the-UI and pipeline-as-code, and why the second wins
- **Connect** Jenkins to a Git repository and trigger builds automatically
- **Build** a complete pipeline that tests an app, builds a Docker image, and pushes it
- **Compare** Jenkins against the other CI/CD tools they'll meet in the wild

<br>

## Sequence

### 09:00–09:15 — Welcome & Recap: Where We Are

Morning. Let's place today in the story, because it slots in precisely.

**Session 1** gave you Azure, and the idea that "ClickOps is fragile" — building by hand in a portal isn't repeatable, has no review, no audit trail.

**Session 2** gave you the shell. You wrote a script called `containerise` that ran `docker build`, checked its exit code, and refused to push if the build failed. At the very end I asked: *what's the missing piece that would make these scripts run automatically, every time someone pushes code, with nobody typing anything?*

**Today is that missing piece.** It's called a **pipeline**, and the tool is **Jenkins**.

Here's the shape. This morning is mostly the *idea* — what CI/CD is and why anyone bothers — then getting Jenkins running on your laptop and building your first jobs. This afternoon goes deeper into pipelines as code, then the back half is one real, end-to-end pipeline that tests an app, builds a Docker image and pushes it to Docker Hub, completely automatically.

One thing worth saying up front: **you already do most of this manually.** You write tests. You run them. You build images. You review each other's code before it merges. Today isn't teaching you new *activities* — it's teaching you to make the activities you already do happen **automatically, in a fixed order, with a gate that stops broken things moving forward.**

Jenkins runs on *your* machine today, so there's setup, and setup on real laptops is where we have issues.

**Everyone make a working folder now:**

*(Run from `~/`)*
- Run: `mkdir -p ~/jenkins-training` → **cd inside** with `cd ~/jenkins-training`

**💬 SLACK — snippet 1**, post at 09:05:
```bash
mkdir -p ~/jenkins-training && cd ~/jenkins-training

docker --version         # must respond
docker ps                # must respond (empty list is fine)
docker run hello-world   # must succeed
df -h .                  # check you have a few GB free
```

Anything failing there gets fixed **now**.

<br>
<br>

### 09:15–10:00 — What CI/CD Is, What Pipelines Are *For*, and Why

Before we touch Jenkins, proper time on the *why* — because if the why lands, everything else is detail.

**Let's start with the pain.** Imagine a team with no pipeline. You finish a feature. What has to happen before real users can use it? Pull the latest code, merge, run the tests, build the app, package it, copy it to a server, restart something, check it didn't break. Every step by hand, by a human, usually the same one or two humans who "know how the deploy works".

**ASK** <br>
What goes wrong, over time, with a team that releases entirely by hand? <br>
**ANSWER** <br>
Someone forgets a step. Someone does them in a different order. It "works on my machine" but not on the server. The one person who knows the deploy goes on holiday and nobody dares release. And because releasing is scary and manual, they do it **rarely** — which means each release is huge, which makes it *more* likely to break, which makes them even more scared to release. It's a doom loop, and the way out is to **release more often, not less.**

**ASK** <br>
You already write tests. Who runs them, and when? <br>
**ANSWER** <br>
Usually the author, on their own machine, when they remember. Which means: tests get skipped when someone's in a hurry, they run on one person's machine with one person's node version and one person's leftover state, and **nobody finds out something broke until the next person to run them happens to run them**. CI (Continuous Integration) isn't new — it's your existing test suite, run **by something that never forgets, on a clean machine, within minutes of every change.**

Let's define the terms properly, because people throw them around loosely.

- `SLIDE ACROSS`

**Continuous Integration (CI)** — every time someone pushes code, it's automatically **built and tested**, straight away. Not next week, not before a release. The goal is catching a problem *the moment it's introduced*, while it's small and while the person who caused it still remembers what they changed.

When we say **built**, that could be compiling the code or bundling the code. Essentually, building is getting code ready to run, testing, packaging or deployment. 

- `SLIDE ACROSS`

**Continuous Delivery (CD)** — every change that passes CI is automatically **packaged and made ready to release**, so shipping is a single, boring, low-risk button press whenever a human decides. Building and pushing a Docker Image could be part of this stage.

- `SLIDE ACROSS`

**Continuous Deployment (also CD)** — the stricter version: every change that passes all checks goes **all the way to production automatically**, no human in the loop.

**ASK** <br>
What's the actual difference between Continuous *Delivery* and Continuous *Deployment*? <br>
**ANSWER** <br>
Delivery keeps a human deciding **when** to release — the release itself is automated and easy, but a person presses go. Deployment removes even that decision. Delivery is the far more common starting point; full Deployment takes serious confidence in your tests, because your test suite becomes the *only* thing standing between a commit and production.

**So what is a "pipeline", concretely?** The **automated assembly line** those ideas run on. An ordered series of **stages** — checkout, build, test, package, deploy — where each must pass before the next runs. If any stage fails, the line stops and nothing broken moves further down.


Let me make it concrete with something you've already built. In Session 2 you wrote `containerise`: it ran `docker build`, checked the exit code, and `exit 1`'d if the build failed so the push never happened. **That was a pipeline in miniature, run by hand.** A build stage, a push stage, and a gate between them. All Jenkins does is run that assembly line *for you*, *automatically*, *every time the code changes*, and draw you a picture of which stage passed.

**ASK** <br>
Thinking back to "you build it, you run it" from Session 1 — how does a pipeline support that? <br>
**ANSWER** <br>
It closes the gap between "I finished writing code" and "my code is running somewhere real", and gives the person who wrote it **fast, automatic feedback**. Instead of throwing code over a wall and waiting, the same engineer pushes and knows within minutes whether they broke anything. The pipeline is what makes "you run it" practical rather than terrifying.

**Where does Jenkins fit?** Jenkins is an **automation server**. Its entire job: *watch for an event, and when it happens, run some tasks.* The event is usually "someone pushed to Git". The tasks are whatever you tell it. Jenkins genuinely does not care what they are — it's a very good, very configurable "run these steps when this happens" engine that reads exit codes.


Given Jenkins just runs shell commands and reads exit codes — why did we spend so long on exit codes last session? <br>

Because exit codes are the **entire contract** between your scripts and Jenkins. A stage passes if the last command exits `0` and fails otherwise. That's it. A script that swallows an error and exits `0` anyway makes a broken pipeline look green and ships broken software. Everything Jenkins does about safety rests on your commands being **honest about failure**.

**ASK** <br>
Jenkins is far from the only tool that does this. What others have you heard of? <br>
**ANSWER** <br>
GitHub Actions, GitLab CI, CircleCI, Azure Pipelines, Travis. Jenkins is one of the oldest and most widely deployed. We learn it because it's **self-hosted** — you install and run every piece yourself, so nothing is a hidden black box. Once you understand Jenkins, the managed ones feel easy. 

One last mental model before we install. Jenkins has two conceptual parts — worth knowing the words even though we run everything on one machine today:

- `SLIDE ACROSS`

- The **controller** runs the web interface, stores configuration, and decides what work needs doing
- **Agents** are separate machines or containers the controller hands actual work to
- An **executor** is a single "slot" for running one build. A node with two executors runs two builds at once

All can run on our own machines or external machines in the cloud. In our use case we'll have containers doing the work. 


Real teams might not want the controller running every build directly? <br>

One badly-behaved build — an infinite loop, something filling the disk — could take down the Jenkins server every team depends on. Agents isolate that risk and let you scale horizontally: busy, add more agents. For a classroom we let Jenkins use itself as the builder, which is fine for learning and not how you'd run it for real.

**Start your glossary now.** Open `03-pipeline-glossary.md` and fill in the first eight rows before the break — CI, Delivery, Deployment, pipeline, stage, step, agent, executor. **Your own words.**

<br>
<br>

*(Take a 15 minute break here — and use it to check everyone's Docker Desktop is actually running before the install.)*

<br>
<br>

### 10:15–11:15 — Installing & Touring Jenkins on Your Laptop
*(Activity: 35 min)*

Let's get a real Jenkins running on your own machine. We run Jenkins itself as a **Docker container**, for three reasons: you already know Docker, it behaves *identically* on Windows and Mac (no OS-specific installer mess), and it keeps your laptop clean — if it all goes wrong, delete a container and start again.

There's one wrinkle we solve up front. Later today the pipeline needs to run `docker build` *from inside* Jenkins, and the standard Jenkins image doesn't include the Docker command-line tool. So we build a small custom image that adds it, and give Jenkins access to your laptop's Docker. Doing this now means the afternoon "just works".

**Step 1 — build a Jenkins image that can use Docker**

The `Dockerfile` is in your starter repo at `jenkins-image/Dockerfile` — no need to type it. Read it through:

```dockerfile
FROM jenkins/jenkins:lts-jdk17
USER root
RUN apt-get update \
    && apt-get install -y docker.io \
    && rm -rf /var/lib/apt/lists/*
USER jenkins
```

Line by line — all Docker knowledge you already have:
- `FROM jenkins/jenkins:lts-jdk17` — start from the official Jenkins image. `lts` is Long Term Support, the stable release line
- `USER root` — switch to root, because installing packages needs admin rights
- `RUN apt-get update && apt-get install -y docker.io` — install the Docker **command-line tool** (not a whole Docker engine). The `&&` is from Session 2: only install if the update succeeded. The `-y` is the "no interactive prompts" flag we met in the same session — **essential in a Dockerfile, because there's nobody there to answer**
- `rm -rf /var/lib/apt/lists/*` — delete the downloaded package index to keep the image smaller. Chained into the same `RUN` deliberately, so the cleanup happens in the same layer as the download
- `USER jenkins` — drop back to the unprivileged user

*(Run from `~/jenkins-training/<your-fork>/jenkins-image`)*
```bash
docker build -t jenkins-docker .
```

`-t jenkins-docker` names ("tags") the image; the `.` means "build using the Dockerfile in this directory".

**Step 2 — run it**

On **Mac** (Terminal) or **Windows** (**Git Bash**, so the `\` line-breaks work):

*(Run from `~/jenkins-training`)*
```bash
docker run -d --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -u root \
  jenkins-docker
```

Single-line version for any shell, including PowerShell:

*(Run from `~/jenkins-training`)*
```bash
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -u root jenkins-docker
```

Every flag, and they're all revision:
- `-d` — **detached**: run in the background, give you your terminal back
- `--name jenkins` — a friendly name, so you can say `docker stop jenkins` rather than using a random ID
- `-p 8080:8080` — **port mapping**, `host:container`. Traffic to port 8080 on your laptop forwards to 8080 inside the container, where the Jenkins web interface listens
- `-p 50000:50000` — the port Jenkins uses to talk to agents. Unused today, but standard
- `-v jenkins_home:/var/jenkins_home` — a **named volume**. Everything Jenkins saves (config, jobs, plugins, your user) lives at `/var/jenkins_home` inside the container; this maps it to storage Docker manages on your laptop, which survives the container being deleted
- `-v /var/run/docker.sock:/var/run/docker.sock` — this one's different. It mounts your laptop's **Docker socket** into the container. The socket is the "phone line" the `docker` command uses to reach the Docker engine. Handing it in means Jenkins can run `docker build` using **your laptop's** Docker. **This is what makes the capstone possible**
- `-u root` — run as root to sidestep Docker permission faff. Fine for training; **not** production practice
- `jenkins-docker` — the image we just built

**ASK** <br>
Look at that socket mount. Jenkins is running *inside* a container, but it's about to build Docker images. Is it running Docker inside Docker? <br>
**ANSWER** <br>
**No** — and this catches people out. Jenkins has only the docker *client* inside it. The socket mount lets that client talk to the Docker **engine running on your laptop**, outside the container. So when the pipeline builds an image, that image appears in *your* `docker images` list, not in some nested world. It's often called "Docker-out-of-Docker". The practical consequence: **your laptop's disk fills up with images your pipeline builds**, which is worth knowing at 16:00 when someone's build fails on space.

**NOTE FOR TRAINERS** <br>
On Docker Desktop for Mac and for Windows (WSL2 backend), `/var/run/docker.sock` is exposed and the socket mount works as written — the reliable cross-platform path. `-u root` is a deliberate simplification avoiding the docker-group permission dance; call it out honestly as a training shortcut so nobody copies it into a real setup. <br>
If a student's socket mount refuses, they can do the **entire day** except the final `docker build`/`docker push` stages — don't let one broken socket block someone, pair them up. <br>
**END OF NOTE**

Check it's running:

*(Run from `~/jenkins-training`)*
```bash
docker ps
```

You should see a container named `jenkins`, status "Up". If not: `docker ps -a` shows stopped containers, `docker logs jenkins` says why it fell over.

**Step 3 — unlock and set up**

Jenkins takes a minute or two to start. Then fetch the one-time admin password from inside the container:

*(Run from `~/jenkins-training`)*
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

`docker exec jenkins <command>` means "run this command inside the already-running container called jenkins". Here we're `cat`-ing a file that only exists in there.

*(In your browser)*
- Go to **`http://localhost:8080`**
- Paste the password
- Choose **Install suggested plugins** — pulls in Git, Pipeline and Credentials, which we use all day. Give it a few minutes
- Create your **admin user** — **write the username and password down**
- Accept the default Jenkins URL

**ASK** <br>
We deliberately mounted a named volume at `/var/jenkins_home`. From the Docker session — what would happen to your admin user, your plugins and every job you build today if we hadn't? <br>
**ANSWER** <br>
They'd all vanish the moment the container was removed and recreated, because a container's own filesystem is disposable. The named volume stores Jenkins' data on the host, outside the container's lifecycle. Same reason a database container needs a volume: **state that must survive belongs outside the container.**

**A quick tour of the interface**

*(In the Jenkins UI — Dashboard, at `http://localhost:8080`)*

Point these out on your own screen:

- **Dashboard** — home; every job shows here with recent status (blue/green passing, red failing)
- **New Item** (top left) — create a job. "Item" is just Jenkins' word for a job
- **Manage Jenkins** (left sidebar) — the settings hub. Three areas matter:
  - **Plugins** — almost everything Jenkins can do comes from plugins. The "suggested" set is only a starter kit
  - **Credentials** — a secure vault for secrets. We use this properly this afternoon
  - **Nodes** — where build machines are listed
- **Build History** (left sidebar) — every build across all jobs

**HANDS ON (35 min)** <br>

Part A *(20 min)* — get it running.
1. *(Run from the starter repo's `jenkins-image` folder)* Build the image as `jenkins-docker`
2. *(Run from `~/jenkins-training`)* Run the container with all the flags shown. Confirm with `docker ps`
3. Get the unlock password, complete setup in the browser, install suggested plugins, create your admin user

Part B *(10 min)* — explore.
4. *(In the Jenkins UI)* **Manage Jenkins → Plugins → Installed plugins**. Pick **three you don't recognise**, search their names, and write one sentence each on what they do
5. *(In the Jenkins UI)* **Manage Jenkins → Nodes**. You'll see one node, "Built-In Node" — the controller acting as its own builder. Note how many **executors** it has, and add "executor" to your glossary

Part C *(5 min)* — research.
6. Pick **one** of GitHub Actions, GitLab CI or Azure Pipelines. Spend five minutes reading its docs and note **two differences** from what you've just set up. We'll compare notes as a room
**END OF NOTE**

**💬 SLACK — snippet 2**, post at the start:
```bash
# 1. Build the Jenkins image (from the starter repo's jenkins-image folder)
docker build -t jenkins-docker .

# 2. Run it  (single line — works in any shell)
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -u root jenkins-docker

# 3. Confirm
docker ps

# 4. Get the unlock password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# 5. Browser: http://localhost:8080
```

**💬 SLACK — snippet 3**, post around 10:35 when the first person hits trouble:
```bash
docker logs jenkins          # why won't it start?
docker ps -a                 # is it there but stopped?
docker stop jenkins          # stop it (data survives — it's in the volume)
docker start jenkins         # start it again
docker rm -f jenkins         # remove the container (data STILL survives)
docker volume ls             # confirm jenkins_home exists
```

**Solution**

*(Run from `~/jenkins-training/<your-fork>/jenkins-image`)*
```bash
docker build -t jenkins-docker .
```

*(Run from `~/jenkins-training`)*
```bash
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -u root jenkins-docker
docker ps
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Then *(in your browser)* `http://localhost:8080` → paste password → **Install suggested plugins** → create admin user.

**Step 5 answer:** the Built-In Node has **2 executors** by default — Jenkins can run two builds simultaneously before queuing.

**Step 6 — the comparison answers worth surfacing.** Take two or three from the room:

| | Jenkins | GitHub Actions / GitLab CI / Azure Pipelines |
|---|---|---|
| **Where it runs** | A server **you** install, patch and maintain | Managed for you |
| **Config file** | `Jenkinsfile` (Groovy) | `.yml` (YAML) |
| **Setup cost** | What we just did — an hour | Commit a file to your repo. Minutes |
| **Cost** | Free software; you pay for the machine | Free tier, then per build-minute |
| **Ecosystem** | ~1,800 plugins, some unmaintained | Curated marketplace |
| **Control** | Total. Runs anywhere, including on-prem with no internet | Whatever the vendor allows |

The honest summary: **for a greenfield project on GitHub, Actions is usually the pragmatic choice.** Jenkins earns its place where you need on-prem, unusual environments, or where a decade of existing pipelines already exist. And a very large number of enterprises are in exactly that position — which is why you'll meet it.

**Common install failures:**

| Symptom | Cause |
|---|---|
| `port is already allocated` | Something else on 8080. Use `-p 8081:8080` and browse to `:8081` |
| `Cannot connect to the Docker daemon` | Docker Desktop isn't running |
| Container exits immediately | `docker logs jenkins` — usually a typo in the run command |
| Browser shows nothing at 8080 | Jenkins takes 1–2 minutes to start. Wait, then `docker logs jenkins` |

<br>
<br>

### 11:15–12:15 — Your First Job: Watching Jenkins Build Something
*(Activity: 40 min)*

Let's make Jenkins actually *do* something, using the simplest job type. It's not how we write pipelines for real, but it makes the core mechanic — "Jenkins runs your steps and shows you the output" — completely visible before adding syntax on top.

**A Freestyle job**

A **Freestyle project** is configured entirely by clicking in the web UI. No code at all.

*(In the Jenkins UI — Dashboard)*
- Click **New Item** (top left)
- **Enter an item name**: `hello-jenkins`
- Select **Freestyle project** → **OK**
- You land on the configuration page. Scroll to **Build Steps**
- **Add build step** → **Execute shell**
- In the box:

```bash
echo "Hello from Jenkins!"
echo "This build is running as: $(whoami)"
echo "The time is: $(date)"
```

*(That's plain bash — exactly Session 2. `$(whoami)` and `$(date)` are the command substitution you already know.)*

- **Save** → **Build Now** (left sidebar)
- A build appears under **Build History** numbered `#1`. Click **#1** → **Console Output**

There it is — the exact output of your shell commands, captured and shown in a browser. That's the entire heart of Jenkins: it ran your steps somewhere, and kept the log.

**ASK** <br>
Where in an *earlier* session did we see almost exactly this — shell commands running unattended, where you only see what happened by reading a log afterwards? <br>
**ANSWER** <br>
`cron`, from Session 2. A scheduled, unattended script whose output you only see by checking a log file. Jenkins' Console Output does the same job with a proper interface and a lot more built around it. If you understood why cron needs logging, you already understand why Jenkins keeps build logs.

**The workspace and built-in variables**

Every build runs inside a **workspace** — a folder on disk where Jenkins works, one per job. And Jenkins injects useful **environment variables** into every build.

*(In the Jenkins UI — the `hello-jenkins` job)*
- **Configure** → replace the Execute shell contents with:

```bash
echo "This is build number $BUILD_NUMBER"
echo "The job is called $JOB_NAME"
echo "My workspace is $WORKSPACE"
echo "Build $BUILD_NUMBER of job $JOB_NAME" > result.txt
cat result.txt
```

The variables you get for free include:

| Variable | Holds |
|---|---|
| `$BUILD_NUMBER` | This build's number — 1, 2, 3... increments every run |
| `$JOB_NAME` | The job's name (`hello-jenkins`) |
| `$WORKSPACE` | Full path to the folder this build runs in |
| `$BUILD_URL` | Web address of this specific build |

- **Save** → **Build Now**, twice. Read the Console Output each time.

**ASK** <br>
If you run this five times, what will `result.txt` say each time, and why? <br>
**ANSWER** <br>
A different number each run — `$BUILD_NUMBER` increases by one every build. Jenkins gives every build a unique, ever-incrementing number, which becomes genuinely useful later for **tagging Docker images**, so you can trace any running container back to the exact build that produced it.

**Keeping the output — artifacts**

Right now `result.txt` is buried in a workspace folder. An **artifact** is a file a build produces that you want to keep and download.

*(In the Jenkins UI — the `hello-jenkins` job)*
- **Configure** → **Post-build Actions** → **Add post-build action** → **Archive the artifacts**
- **Files to archive**: `result.txt`
- **Save** → **Build Now**
- On the completed build's page, `result.txt` is now a downloadable link

**Making a job trigger itself — cron syntax**

*(In the Jenkins UI — the `hello-jenkins` job)*
- **Configure** → **Build Triggers** → tick **Build periodically**
- **Schedule**: `H/5 * * * *`

That's the cron syntax from Session 2. Five fields:

```
 minute  hour  day-of-month  month  day-of-week
   */5     *         *         *         *
```

`*` means "every", so `*/5 * * * *` is "every 5 minutes, every hour, every day".

Jenkins adds a twist: **`H` means "hash"** — it spreads load. `H/5` still means every 5 minutes, but Jenkins picks *which* minute within each window based on the job name, so 200 jobs don't all fire on the same second. Use `H` wherever you'd otherwise use `*` in the minute field. It's the Jenkins convention.

- **Save**, watch it fire on its own over the next few minutes, then **turn it off again**.

**The catch with Freestyle — and why we're about to leave it**

We have a working job. Let's poke at it the way we poke at everything on this course.

**ASK** <br>
All that configuration — the shell commands, the archive setting, the trigger schedule — where does it actually *live*? <br>
**ANSWER** <br>
Inside Jenkins, in its own internal config (in that `jenkins_home` volume). **Not** in your Git repository.

**ASK** <br>
You review every line of application code through a pull request. Would you accept a colleague changing the **build process** with no review, no diff and no history? <br>
**ANSWER** <br>
Obviously not — and yet that's exactly what a Freestyle job allows. Someone clicks, and it's live for everyone immediately. No version history of how the build changed, no review, no way to copy it to another project, no way to run different behaviour per branch, and no undo. **It's ClickOps all over again** — the exact problem we called out with the Azure Portal, wearing a Jenkins costume.

That gap is what **Pipelines** fix, and it's where we go next.

**HANDS ON (40 min)** <br>

Part A *(20 min)* — build the job.
1. Create `hello-jenkins` as a **Freestyle project** with the `$BUILD_NUMBER` version of the shell step
2. Add the archived artifact and confirm you can download `result.txt` from a completed build
3. Add a **Build periodically** trigger of `H/5 * * * *`, watch it fire on its own, then turn it off

Part B *(10 min)* — break it on purpose.
4. Add a **second** shell step that deliberately fails, **above** the working one:
   ```bash
   echo "About to fail on purpose"
   exit 1
   ```
   Rebuild. What colour is the build? Did the second step run? What does the Console Output end with?
5. Remove it and confirm green again

Part C *(10 min)* — discuss and record.
6. In pairs: if three people on your team disagreed about what the build steps should do, how would you resolve it with a Freestyle job? Compare that to how you'd resolve a disagreement about application code
7. Add these to your glossary: **workspace**, **artifact**, **Freestyle job**
**END OF NOTE**

**💬 SLACK — snippet 4**:
```bash
# First version
echo "Hello from Jenkins!"
echo "This build is running as: $(whoami)"
echo "The time is: $(date)"

# Second version — using Jenkins' built-in variables
echo "This is build number $BUILD_NUMBER"
echo "The job is called $JOB_NAME"
echo "My workspace is $WORKSPACE"
echo "Build $BUILD_NUMBER of job $JOB_NAME" > result.txt
cat result.txt

# Part B — the deliberate failure
echo "About to fail on purpose"
exit 1
```

**Solution**

**Part A** — *(In the Jenkins UI — Dashboard)* **New Item** → `hello-jenkins` → **Freestyle project** → **OK** → **Build Steps** → **Add build step** → **Execute shell**:

```bash
echo "This is build number $BUILD_NUMBER"
echo "The job is called $JOB_NAME"
echo "Build $BUILD_NUMBER of job $JOB_NAME" > result.txt
cat result.txt
```

Then **Post-build Actions** → **Archive the artifacts** → `result.txt` → **Save** → **Build Now**.

Trigger: **Configure** → **Build Triggers** → **Build periodically** → `H/5 * * * *` → **Save**. Untick afterwards.

**Part B answers** — this is the important bit, so draw it out:
- The build goes **red**
- The Console Output **ends at `exit 1`**
- **The second build step never runs**

Jenkins stopped the moment a step returned a non-zero exit code. That's the **exact same exit-code contract** from Session 2, now being enforced by a tool rather than by your own `if` statement. Nothing new is happening — Jenkins is just reading `$?` for you and deciding whether to continue.

**Part C discussion answer:** You couldn't resolve it properly. There's no diff, no pull request to comment on, no history showing who changed what and why. Somebody clicks and it's live. Compare that to application code, where the same disagreement gets a PR, line comments, an approval and a permanent record. **The build process deserves the same treatment as the code it builds** — that's the entire argument for the next section.

<br>
<br>
### 12:15–13:00 — Pipeline as Code: Your First `Jenkinsfile`
*(Activity: 20 min + challenge)*

Here's the big shift. Instead of configuring a job by clicking, we describe the whole pipeline in a **text file** called a `Jenkinsfile`, and keep it **in Git, alongside the code it builds**.

**ASK** <br>
Where have we seen this exact philosophy already? <br>
**ANSWER** <br>
Everywhere this course goes: the bash scripts from Session 2, the Dockerfile that defines an image, and — coming in Session 5 — Terraform's `.tf` files. Same instinct every time: the desired thing described as **reviewable, versioned, repeatable code**, not clicked together and forgotten.

#### A word on the syntax before we write any

A `Jenkinsfile` is written in **Groovy**. You don't need to learn Groovy — we use a restricted, structured subset called **Declarative Pipeline**, which is really just nested blocks. But three bits will look unfamiliar, so let's name them.

**1. Everything is a block, marked by `{ }`.** A named container holding other things, nested like Russian dolls:

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

**No semicolons** at line ends. Indentation is for humans — Groovy doesn't care, but keep it tidy or the nesting becomes unreadable.

**2. `sh` runs a shell command.** `sh 'npm install'` is exactly like typing `npm install` in a terminal. It's how a pipeline actually *does* anything. (There's also `bat` for Windows batch — we won't need it, because our Jenkins runs in a Linux container.)

**3. Single quotes and double quotes behave differently.** This is the one that genuinely catches people out, so it's worth doing carefully:

| Written as | Behaviour |
|---|---|
| `'Hello ${NAME}'` | **single quotes** — literal. Groovy leaves `${NAME}` exactly as typed |
| `"Hello ${NAME}"` | **double quotes** — Groovy substitutes the variable's value in |

So `echo "Building ${APP_NAME}"` prints the value; `echo 'Building ${APP_NAME}'` prints the literal text.

**ASK** <br>
Where did you meet an almost identical rule last session? <br>
**ANSWER** <br>
**Heredocs.** `<< 'EOF'` kept everything literal; `<< EOF` substituted your variables. Identical concept, different syntax — quote it to keep it literal, leave it unquoted to substitute. Recognising that these are the same idea in a different costume is most of what makes a second language easy.

One extra wrinkle: `sh 'echo $APP_NAME'` **also works** — because there the *shell* does the substitution, not Groovy. Two different mechanisms reaching the same result. Single quotes in `sh` steps are actually the safer default, and this afternoon we'll see a security reason why.

#### The simplest possible pipeline

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

Top to bottom:
- `pipeline { }` — every declarative pipeline is wrapped in this. Always. The outermost container
- `agent any` — "run this on any available builder/executor". Today that's our controller
- `stages { }` — the container holding the ordered list of phases. **This is the assembly line**
- `stage('Say Hello') { }` — one phase. The quoted text is its name, and it shows up in the UI
- `steps { }` — the container for the things to actually do
- `echo` — a built-in step that prints a message

**Run it**

For now we paste it into Jenkins; we move it into Git this afternoon.

*(In the Jenkins UI — Dashboard)*
- **New Item** → `hello-pipeline` → select **Pipeline** → **OK**
- Scroll right down to the **Pipeline** section
- Leave **Definition** as **Pipeline script**
- Paste the pipeline into the **Script** box
- **Save** → **Build Now**

The build page now shows a **Stage View** — a box labelled "Say Hello" with a green tick. As pipelines grow, that visual lets you see at a glance which stage passed and which broke.

**Add a second stage so you can see the assembly line**

*(In the Jenkins UI — the `hello-pipeline` job → **Configure**)*

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

- **Save** → **Build Now**, and watch the Stage View show **two** boxes lighting up green in order.

**ASK** <br>
Why is it useful that each stage shows separately, rather than the whole build being just "passed" or "failed"? <br>
**ANSWER** <br>
When something breaks you instantly see **where**. "Test failed" tells you far more than "the build failed". On a real pipeline with checkout, build, test, package and deploy, that pinpointing saves real time — and it's why we split work into named stages even when we could cram it into one. It's the same reason you write focused unit tests rather than one giant test called "everything works".

**HANDS ON (20 min)** <br>
*(All in the Jenkins UI — the `hello-pipeline` job)*
1. Create `hello-pipeline` and get the two-stage version running with a green Stage View
2. Add a **third** stage called `Package` that echoes a message
3. Make one `sh` step deliberately fail — `sh 'exit 1'` — rebuild, and watch that stage go **red** and the stages *after* it **not run at all**. Then fix it back to green
4. Prove the quoting rule to yourself: add two lines to a stage, `echo "Build ${BUILD_NUMBER}"` and `echo 'Build ${BUILD_NUMBER}'`, and compare the output
5. Add to your glossary: **Jenkinsfile**, **declarative pipeline**, **stage**, **step**
**END OF NOTE**

**💬 SLACK — snippet 5**:
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

**Solution**

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'echo "pretend build step running"'

                // Step 4 — the quoting comparison
                echo "Double quotes: build ${BUILD_NUMBER}"   // -> Double quotes: build 7
                echo 'Single quotes: build ${BUILD_NUMBER}'   // -> Single quotes: build ${BUILD_NUMBER}
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

With `sh 'exit 1'` in Test, the Stage View shows **Build = green**, **Test = red**, **Package = grey/skipped** — it never ran. Change it back to `sh 'echo "pretend tests passing"'` and all three go green.

**Step 4 is worth pausing on as a room.** The output makes the rule undeniable:
```
Double quotes: build 7
Single quotes: build ${BUILD_NUMBER}
```
Nobody forgets it after seeing it once.

---

**Challenge**

*Direct* students, **in pairs**, to write a declarative pipeline that:

* Has **three** stages named `Checkout`, `Test` and `Deploy`
* Each stage prints a message saying what it's doing
* `Deploy` additionally prints **which build number** it's deploying, using `$BUILD_NUMBER`
* `Test` must run a real shell command using `sh`, not just an `echo`
* **OPTIONAL** — make the pipeline fail in `Test` on **even-numbered builds only**, so it alternates green and red on successive runs

*Provide* this example Console Output for a successful third build:

```
Checking out the code...
Running the tests...
tests passed
Deploying build number 3
```

*Grant* ~10 minutes.

Hints if stuck: remember **double quotes** for `${BUILD_NUMBER}` to be substituted by Groovy. For the optional part, the shell has a modulo operator — `$((BUILD_NUMBER % 2))` gives `0` for even numbers.

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
                // Double quotes so Groovy substitutes the build number
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
                // The if, the [ ] brackets, -eq and exit 1 are ALL
                // exactly what you wrote in Session 2. Nothing new.
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

Two things to draw out when revealing:

- **Triple single quotes `'''...'''`** let you write a multi-line shell script inside one `sh` step. Very common in real pipelines
- The `if`, the `[ ]` test brackets, `-eq` and `exit 1` are **exactly** Session 2 material. **A pipeline is mostly a wrapper around shell skills you already have** — which is the single most reassuring thing to tell a room that finds Groovy intimidating

Run it three or four times and watch it alternate red, green, red, green — and notice `Deploy` is skipped every time `Test` goes red.

<br>
<br>

*(Take a 1 hour lunch break here.)*

<br>
<br>

### 14:00–14:45 — Anatomy of a Pipeline: Stages, Environment, Post, Parallel
*(Activity: 20 min)*

This morning you saw the *shape* of a pipeline. Now the toolkit — the handful of blocks that appear in almost every real `Jenkinsfile`.

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

- `environment { }` — variables available to **every** stage
- `APP_NAME = 'countries-api'` — assignment. Note Groovy **does** use spaces around `=`, unlike bash. Small difference, endless confusion
- `echo "${GREETING} ${APP_NAME}..."` — **double quotes**, because we want values substituted

**ASK** <br>
Where have you already used this exact idea? <br>
**ANSWER** <br>
Bash variables in Session 2, and Terraform `variable` blocks in Session 5. Same concept, three different punctuations. Once you see it's the *same* idea, each new tool's version takes thirty seconds to learn rather than an hour.

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

- `parameters { }` — inputs supplied when a build starts
- `string(name: ..., defaultValue: ..., description: ...)` — a **function call with named arguments**. The labels mean order doesn't matter and it's self-documenting. Other types exist: `booleanParam`, `choice`
- `${params.ENVIRONMENT}` — parameters live under `params.`, so the prefix is required

Once a pipeline has `parameters`, the button changes from **Build Now** to **Build with Parameters**.

**NOTE FOR TRAINERS** <br>
Flag this gotcha *before* students hit it: the **first** build after adding a `parameters` block still runs with defaults and shows a plain "Build Now" — Jenkins must run the pipeline once to *discover* the parameters exist. From the second build you get the prompt. Students reliably conclude they've done it wrong. <br>
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

`post` sits **outside** `stages` and runs after all of them:
- `success { }` — only if everything passed
- `failure { }` — only if something broke
- `always { }` — no matter what. Good for cleanup, or publishing test reports
- `unstable` and `changed` also exist — `changed` fires only when the result **differs from the previous build**, which is how you avoid spamming a channel every night with "still broken"

**ASK** <br>
In a real team, what would you actually want in that `failure` block instead of an `echo`? <br>
**ANSWER** <br>
A notification — a Slack message to the team channel, an email, a page to whoever's on call, maybe auto-opening a ticket. The whole point of CI is **fast feedback**, and a failure nobody's told about is slow feedback. A pipeline that goes red in a browser tab nobody has open is barely better than no pipeline.

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

`parallel { }` replaces the `steps { }` block of a stage and contains **nested stages** that all run at once. The outer stage isn't done until every branch completes.

**ASK** <br>
Why run unit tests and linting in parallel rather than one after the other? <br>
**ANSWER** <br>
They don't depend on each other — the linter's result doesn't change whether tests should run, or vice versa. Running them together gives faster feedback with no change to the outcome. **Faster feedback is the entire game in CI**, so anything independent is a candidate. The limit is executors: two parallel branches need two free executors, and our Built-In Node has exactly two.

**One more idea to *name*, not build: Multibranch**

There's a job type called a **Multibranch Pipeline** that scans a whole repo and automatically creates a pipeline for **every branch and every open Pull Request** containing a `Jenkinsfile` — deleting them when the branch goes.

**ASK** <br>
Why is that a perfect fit for the Pull Request workflow you already use? <br>
**ANSWER** <br>
Because every feature branch and every open PR automatically gets its own build and test run, with zero manual job creation. That's exactly the mechanism behind "the tests must pass before we allow a merge" — the green tick you've all seen on a GitHub PR. And in Session 5 it becomes "run `terraform plan` on every PR so a reviewer can see what would change to real infrastructure". We won't wire it up today, but know the term.

**HANDS ON (20 min)** <br>
*(All in the Jenkins UI — the `hello-pipeline` job → **Configure**)*

Add these one at a time, rebuilding after each:
1. An `environment` block with an `APP_NAME`, used inside a stage's `echo`. **Double quotes**
2. A `post` section with `success`, `failure` and `always`. Then make a step fail (`sh 'exit 1'`) and confirm `failure` and `always` run but `success` does **not**. Fix it and confirm the opposite
3. A `parameters` block with a `string` parameter, echoing `params.YOURPARAM` in a stage. Build **twice** and notice the button change on the second run
4. **(Stretch)** Convert your `Test` stage into a `parallel` block with two nested stages
5. Add to your glossary: **post block**, **parameters**, **parallel**, **multibranch pipeline**
**END OF NOTE**

**💬 SLACK — snippet 6**:
```groovy
    environment {
        APP_NAME = 'countries-api'
    }

    parameters {
        string(name: 'ENVIRONMENT', defaultValue: 'dev', description: 'Which environment to target')
    }

    post {
        success { echo 'It worked!' }
        failure { echo 'It broke.' }
        always  { echo 'Runs every time.' }
    }

    // parallel — replaces steps { } inside a stage
    stage('Checks') {
        parallel {
            stage('Unit Tests') { steps { sh 'echo "unit tests passed"' } }
            stage('Linting')    { steps { sh 'echo "no lint errors"' } }
        }
    }
```

**Solution**

All four combined:

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

        // 4. Stretch: two checks at the same time
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

    // 2. Runs after all stages, based on outcome
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

To test step 2, temporarily change a step to `sh 'exit 1'` and rebuild. The Console Output shows the **`failure`** and **`always`** messages but **not** `success`. Change it back and you get `success` and `always`, not `failure`.

<br>
<br>

### 14:45–15:00 — Connecting Jenkins to Git, and Triggering Builds

Our `Jenkinsfile` is still pasted into the Jenkins UI — the *exact same* "not really code" problem as the Freestyle job. Let's fix the concept now so the capstone can do it for real.

**Pulling the `Jenkinsfile` from Git**

*(In the Jenkins UI — a Pipeline job → **Configure**)*
- Scroll to the **Pipeline** section
- Change **Definition** from `Pipeline script` to **Pipeline script from SCM**

*(SCM = "Source Control Management", Jenkins' generic term for Git and its older cousins.)*

- **SCM**: `Git`
- **Repository URL**: your fork
- **Branch Specifier**: `*/main` — the `*/` prefix means "the `main` branch from whichever remote", so it matches `origin/main`
- **Script Path**: `Jenkinsfile` — path within the repo. The default assumes the repo root

Now the pipeline definition lives in Git: versioned, reviewable in a PR, identical for anyone who runs it.

**Handling secrets — the Credentials store**

To pull a private repo, or push to Docker Hub later, Jenkins needs secrets. We never paste those into a config box or into the `Jenkinsfile`.

*(In the Jenkins UI — Dashboard)*
- **Manage Jenkins → Credentials → System → Global credentials (unrestricted) → Add Credentials**
- We set one up properly in the capstone

**ASK** <br>
Why store a token in the vault rather than typing it into the repository URL field, or the `Jenkinsfile`? <br>
**ANSWER** <br>
Three reasons, and the third is the one people miss. **(1)** The vault keeps the secret out of Git — and **Git history is permanent**, so deleting the line later doesn't remove it. **(2)** It lets you rotate the secret in one place without editing code. **(3)** Jenkins **automatically masks** the value in build logs. Without that, any `sh` step that happened to echo the variable would print your live credential into a log that everyone in the team can read. That third one is why an environment variable isn't good enough.

**Triggering builds automatically**

The magic of CI is that **nobody clicks Build**. A change to Git triggers it. Two ways:

- **Poll SCM** — Jenkins checks the repo on a schedule (cron syntax again) and builds only if something changed. Simple, works anywhere, but there's a delay and it's a bit wasteful
- **Webhook** — GitHub *tells* Jenkins the instant a push happens. Instant, efficient, how real setups do it

**ASK** <br>
Between polling every minute and a webhook, which is better and why? <br>
**ANSWER** <br>
The webhook. Polling means repeatedly asking "anything changed yet?" when the answer is almost always no — wasted requests, plus a delay of up to the polling interval. A webhook is push rather than pull: GitHub notifies Jenkins the moment something happens. It's the same argument as polling an API versus subscribing to events, which you've all met in frontend work.

**NOTE FOR TRAINERS** <br>
The local-laptop reality: a webhook from GitHub **cannot** reach `http://localhost:8080` on a student's machine, because GitHub is on the public internet and their Jenkins isn't. The proper fix is a tunnel like [ngrok](https://ngrok.com/), which is a good optional stretch. For the guaranteed-to-work classroom path we use **Poll SCM** in the capstone — no networking magic, same principle demonstrated. Teach the webhook as the goal; use polling as the hands-on. <br>
**END OF NOTE**

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>
### 15:15–16:45 — Capstone: A Real Test → Build → Push Pipeline
*(Activity: 90 min)*

The main event, tying the day — and the course so far — together. You're building one pipeline that, automatically on a push to Git: **checks out a Node app, installs it and runs its tests, builds a Docker image, and pushes that image to Docker Hub.** That's the genuine shape of how real teams ship software.

Work individually or in pairs. Get the core pipeline green first; stretch goals if you race ahead.

**NOTE FOR TRAINERS** <br>
Tell them the split explicitly at the start: **Parts 0–1 are setup, Part 2 is reading, Part 3 is where it gets real.** The `Jenkinsfile` gets pasted from Slack — there's no learning in transcribing Groovy, and one missing brace costs twenty minutes. Protect time for Part 3, because the moment a build triggers itself from a `git push` is the moment the day lands. <br>
**END OF NOTE**

---

#### Part 0 (≈10 min) — Get the app and a Docker Hub token ready

**1. Clone your fork.**

*(Run from `~/jenkins-training`)*
```bash
git clone https://github.com/<your-username>/<your-fork>.git
cd <your-fork>/app
```

*(Run from `~/jenkins-training/<your-fork>/app`)*
```bash
ls
cat package.json
cat test.js
cat Dockerfile
```

A small Express app with a passing test and a Dockerfile. **Read `test.js` properly** — it's deliberately simple, and note the last few lines:

```javascript
if (failures > 0) {
  process.exit(1);        // NON-ZERO = the pipeline will fail
}
process.exit(0);          // ZERO = success
```

**ASK** <br>
Your usual test runner is Jest or Vitest. Do those do the same thing? <br>
**ANSWER** <br>
**Yes** — every test runner exits non-zero when tests fail. You've never needed to care, because you read the red output on your own screen. In a pipeline **nobody is reading the screen**, so that exit code is the only signal that reaches Jenkins. It's the Session 2 contract again, and it's why `npm test` works as a pipeline stage with no extra wiring.

**2. Get a Docker Hub access token.**

*(In your browser — [hub.docker.com](https://hub.docker.com))*
- **Account Settings → Security → New Access Token**
- Description, permissions **Read & Write**, **Generate**
- **Copy it now** — Docker Hub shows it once

This token is what the pipeline pushes with — **never** your account password. A token can be revoked independently without changing your login.

---

#### Part 1 (≈10 min) — Store your Docker Hub credentials in Jenkins

*(In the Jenkins UI — Dashboard)*
- **Manage Jenkins → Credentials → System → Global credentials (unrestricted) → Add Credentials**
- **Kind**: `Username with password`
- **Username**: your Docker Hub username
- **Password**: the access token
- **ID**: `dockerhub-credentials` ← **this exact string**, because the `Jenkinsfile` refers to it by ID
- **Description**: e.g. "Docker Hub push token"
- **Create**

**Checkpoint question:** why did we put the token *here* and not in the `Jenkinsfile` we're about to commit to Git?

---

#### Part 2 (≈25 min) — Write the pipeline

*(Run from `~/jenkins-training/<your-fork>`)*
- Run: `touch Jenkinsfile`
- Then: `code Jenkinsfile`

Paste from Slack, replacing `your-dockerhub-username`:

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
                dir('app') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }

        stage('Build Image') {
            steps {
                dir('app') {
                    sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
                }
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

**Read it together before running it.** Each new piece connects to something you know:

**`checkout scm`** — checks out the same repository this `Jenkinsfile` came from. You don't specify the URL again; Jenkins already knows it. `scm` is a variable Jenkins provides holding your source-control config.

**`dir('app')`** — runs the enclosed steps **inside that subfolder**. Our app lives in `app/`, not the repo root. It's `cd app` for a block of steps, and it puts you back afterwards.

**A stage with its own `agent`:**
```groovy
stage('Install & Test') {
    agent {
        docker { image 'node:20' }
    }
```
The top-level `agent any` sets the default, but **an individual stage can override it**. This stage runs *inside a freshly-started `node:20` container* — Jenkins pulls the image, starts it, mounts the workspace in, runs the steps, throws it away.

**ASK** <br>
Why bother? Jenkins could just have Node installed. <br>
**ANSWER** <br>
Because then **every project on that Jenkins shares one Node version**, and upgrading it for one team breaks another. With a per-stage Docker agent, each project declares the exact environment it needs, in its own `Jenkinsfile`, and gets a clean one every build. It's the "works on my machine" problem solved at the CI layer — the same argument that made you containerise your apps in the first place, applied one level up.

**Tagging with the build number:**
```groovy
IMAGE_TAG  = "${BUILD_NUMBER}"
```
Every image is tagged with the build that produced it, so you can trace any running container back to an exact build, and to the exact commit that build checked out. The `$BUILD_NUMBER` you met this morning, doing real work.

**The two quoting styles, side by side.** In `environment` we use Groovy double quotes so `${BUILD_NUMBER}` is substituted. In `sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'` we use **single** quotes with `$VAR` — there the *shell* substitutes, because Jenkins exports everything in `environment` as real shell environment variables. Both work. Single quotes in `sh` steps are safer, for the reason immediately below.

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
- `credentialsId:` — matches the **ID** from Part 1
- `usernameVariable:` / `passwordVariable:` — names the values get bound to
- Inside the `{ }` you can use `$DOCKER_USER` and `$DOCKER_PASS`; **outside they don't exist**
- Jenkins **masks** these in the console log — you see `****`

**ASK** <br>
Given Jenkins masks the values anyway, why do the docs insist on **single** quotes in `sh` steps that use secrets? <br>
**ANSWER** <br>
Because with **double** quotes, Groovy substitutes the secret's value into the command string *before* handing it to the shell — so the literal secret can end up in a process listing, or in a stack trace if the step errors. With single quotes, Groovy passes `$DOCKER_PASS` through untouched and the **shell** resolves it from the environment, so the value never appears in the command Groovy built. Masking catches most leaks; single quotes prevent one it can't.

**`--password-stdin`:**
```bash
echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
```
The pipe from Session 2. Rather than putting the password on the command line — where it appears in process listings — we pipe it in via standard input.

Now commit and push:

*(Run from `~/jenkins-training/<your-fork>`)*
```bash
git add Jenkinsfile
git commit -m "Add build and push pipeline"
git push origin main
```

**💬 SLACK — snippet 7.** Post the whole `Jenkinsfile` here. Don't make them type it.

---

#### Part 3 (≈35 min) — Point Jenkins at it and run it

*(In the Jenkins UI — Dashboard)*
1. **New Item** → `build-and-push` → **Pipeline** → **OK**
2. **Pipeline** section:
   - **Definition**: `Pipeline script from SCM`
   - **SCM**: `Git`
   - **Repository URL**: your fork's HTTPS URL
   - **Branch Specifier**: `*/main`
   - **Script Path**: `Jenkinsfile`
3. Scroll up to **Build Triggers** → tick **Poll SCM** → **Schedule**: `H/2 * * * *`
4. **Save** → **Build Now** for the first run
5. Watch the **Stage View** march through Checkout → Install & Test → Build Image → Push Image

*(In your browser — [hub.docker.com](https://hub.docker.com))*
6. When green, confirm the image is in your Docker Hub repositories, tagged with the build number

**Now the moment that matters:**

*(Run from `~/jenkins-training/<your-fork>`)*
```bash
echo "A trivial change" >> README.md
git add README.md
git commit -m "Trigger the pipeline"
git push origin main
```

Then **wait**. Within a couple of minutes Poll SCM kicks off a build **on its own**. A build you didn't start, triggered purely by a Git push. Sit with that — it's the whole point of the day.

**ASK** *(mid-capstone checkpoint, ~16:15)* <br>
Look at your stage order: Test comes *before* Build and Push. What does that ordering actually protect you from? <br>
**ANSWER** <br>
If the tests fail, the pipeline stops there and **never builds or pushes the image**. A broken version physically cannot reach Docker Hub, because the gate caught it. Same principle as your `containerise` script checking exit codes. And there's a second reason: **the tests are the cheapest stage.** Running them first means a broken commit fails in thirty seconds rather than after a three-minute image build. Cheapest checks first is a design habit that scales all the way up.

---

#### Part 4 — Stretch goals

1. **Prove the gate works.** Add a stage *before* Build Image that deliberately fails (`sh 'exit 1'`), push it, and confirm nothing gets built or pushed. Then remove it. Worth doing — *seeing* the pipeline refuse to ship broken code is more convincing than being told it will
2. **Break the test instead.** Edit `app/test.js` so an assertion genuinely fails, push, and watch the pipeline stop at Install & Test. This is the realistic version of stretch 1
3. **Add a `latest` tag.** In the Push stage, also tag and push `$IMAGE_NAME:latest` alongside the numbered tag
4. **Parallelise the checks.** Add a lint step and run it in `parallel` with the tests
5. **Notify on failure.** Enrich the `failure` block to print the image name and the build URL, then read about the Slack/email plugins that do this for real
6. **Webhook instead of polling** (advanced): install `ngrok`, expose your local Jenkins, set up a GitHub webhook so pushes trigger builds *instantly*

**Solution**

**Stretch 1 — proving the gate.** Insert between `Install & Test` and `Build Image`:

```groovy
stage('Deliberate Failure') {
    steps {
        echo 'About to fail on purpose...'
        sh 'exit 1'
    }
}
```

Stage View: Checkout ✅ → Install & Test ✅ → Deliberate Failure ❌ → **Build Image and Push Image never run**. Check Docker Hub — no new tag. Delete the stage and push to go green.

**Stretch 2 — break the test properly.** In `app/test.js`:

```javascript
check("app is exported", () => {
  assert.ok(false, "deliberately failing");   // was: assert.ok(app, ...)
});
```

Push it. The pipeline fails at **Install & Test**, and the Console Output shows your actual test failure message. **This is the more realistic demo** — it's what a genuine broken commit looks like, and it proves the whole chain: failing assertion → non-zero exit → red stage → nothing shipped. Revert it afterwards.

**Stretch 3 — a `latest` tag as well:**

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

**Stretch 4 — parallel checks:**

```groovy
stage('Install') {
    agent { docker { image 'node:20' } }
    steps {
        dir('app') { sh 'npm install' }
    }
}

stage('Test & Lint') {
    agent { docker { image 'node:20' } }
    parallel {
        stage('Unit Tests') {
            steps { dir('app') { sh 'npm test' } }
        }
        stage('Lint') {
            steps {
                dir('app') {
                    sh 'npm run lint || echo "no lint script configured — skipping"'
                }
            }
        }
    }
}
```

*(The `|| echo ...` is Session 2's `||` — so a missing `lint` script doesn't fail the build while experimenting. In a real project you'd want it to fail.)*

**Stretch 5 — a more useful failure message:**

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

`currentBuild.currentResult` is a built-in object holding this build's outcome — `SUCCESS`, `FAILURE` or `UNSTABLE`.

**Stretch 6 — ngrok webhook (outline).**

*(Run from `~/jenkins-training`)*
```bash
ngrok http 8080
```
Copy the `https://....ngrok-free.app` address. Then *(on GitHub)*: **your fork → Settings → Webhooks → Add webhook**, **Payload URL** = `https://....ngrok-free.app/github-webhook/` (**the trailing slash matters**), **Content type** = `application/json`, **Just the push event**. Then *(in the Jenkins UI)*: **Configure** the job → **Build Triggers** → untick Poll SCM, tick **GitHub hook trigger for GITScm polling**. Push a commit; the build starts within a second or two.

**Common capstone failures:**

| Symptom | Cause |
|---|---|
| `docker: not found` in Build Image | Using the stock Jenkins image, not `jenkins-docker` |
| `permission denied` on the docker socket | `-u root` omitted from the `docker run` |
| `npm: not found` | The `agent { docker { image 'node:20' } }` block is missing or misplaced |
| `denied: requested access to the resource is denied` | `IMAGE_NAME` doesn't start with **your** Docker Hub username |
| Push works but nothing on Docker Hub | Look under the right account — check you're logged into the same one |
| Poll SCM never triggers | Job was saved before the first manual build. Run **Build Now** once first |

#### What to show at 16:45

Push a change, and via Poll SCM watch a build start on its own, go green through all stages, and produce a freshly-tagged image in your Docker Hub. Then explain, in your own words, **why the Test stage comes before the Push stage**.

<br>
<br>

### 16:45–17:00 — Wrap-up & Q&A

Let's pull the day together.

You started with a definition — CI/CD replaces scary, manual, occasional releases with automatic, safe, frequent ones — and a mental model: **a pipeline is an assembly line of stages, where each must pass before the next runs, and the line stops the moment something breaks.**

Then you made it real. You installed Jenkins on your own laptop, ran your first job, and watched the shift from clicking a job together in a UI to describing it as a `Jenkinsfile` in Git — reviewable, versioned, repeatable. And in the capstone you built the genuine article: a pipeline that, on a git push, tests an app, builds an image and ships it to a registry, on its own — refusing to push anything if the tests fail.

And notice how much of today was **bash skills wearing a Jenkins hat**. Every `sh` step, every `exit 1`, every `||`, every pipe into `docker login` — that was Session 2. The pipeline is the wrapper; **the shell is still doing the work.**

**ASK** <br>
Look at the shape of that capstone: checkout, a stage that tests, a stage that changes something real, a stage that ships it — with credentials from the vault and a `post` block reporting the outcome. If we wanted this *same* pipeline to run **Terraform against Azure** instead of building a Docker image, how much would change? <br>
**ANSWER** <br>
Structurally, **almost nothing**. The credentials pattern is identical — you'd store an Azure Service Principal's details in the vault instead of a Docker Hub token, and you already made one of those in Session 1. The `sh` steps would run `terraform init`, `plan`, `apply` instead of `docker build`/`push`. The stages, the gating, the `post` block — all the same. That's not a coincidence: **it's Session 5**, and everything you built today is the ground it stands on.

Where this sits:
- **Today** — you can explain and build a pipeline, and run Jenkins yourself
- **Session 5, Terraform & IaC** — the same shape, running Terraform to provision real infrastructure, with a human approval gate before `apply`
- **Session 7, Kubernetes** — running and scaling the images your pipeline now builds
- **Session 8, Integration** — all of it woven together: a push to Git flowing through test, build, provision and deploy

**Before you go:**
1. Your `pipeline-glossary.md` is complete and committed
2. Your Docker Hub token is somewhere safe — **not** in a Git repo
3. You know how to stop and restart Jenkins without losing anything

**💬 SLACK — snippet 8**:
```bash
docker stop jenkins     # frees your laptop's resources
docker start jenkins    # everything's still there, thanks to the named volume

docker images           # see what today's builds left behind
docker image prune      # tidy up dangling images if you're short on disk
```

**Q&A** — take remaining questions.

<br>
<br>

### Exercise (take-home / reinforcement)

Before the next session, individually or in pairs. **Do at least one practical and one research task.**

**Practical**

1. From scratch, in a brand new folder, get Jenkins running and build a two-stage pipeline **from memory** — no notes. Rebuilding is the fastest way to make it stick
2. Add a **`Package` stage** between Test and Build Image that prints the version being built — practising adding a stage cleanly
3. Add a `parameters` block letting you pass the image tag manually, defaulting to the build number, used in Build and Push
4. Deliberately break `app/test.js`, push, and screenshot the red pipeline. Then fix it and screenshot the green one. Put both in your notes — it's the clearest possible demonstration of what CI is for

**Research**

5. **Finish your glossary**, including the one-sentence "what's a pipeline?" answer at the bottom. Commit it
6. **Compare CI/CD tools.** Pick **two** of GitHub Actions, GitLab CI, CircleCI and Azure Pipelines. Write half a page: what would make you choose each over Jenkins, and what would make you choose Jenkins over them?
7. Write a short paragraph, in your own words, explaining the difference between Continuous **Delivery** and Continuous **Deployment**, with an example of a team that would prefer each
8. **(Stretch)** Read the [Jenkins Pipeline syntax docs](https://www.jenkins.io/doc/book/pipeline/syntax/) and find one directive we didn't cover (`when`, `options`, `tools`, `triggers`). Work out what it does and where you'd use it
9. **(Stretch)** Set up `ngrok` and a GitHub webhook so a push triggers your pipeline **instantly**, and compare the feel against Poll SCM

**Solutions** *(for the guided ones — 2, 3, 8)*

**Take-home 2** — the `Package` stage:
```groovy
stage('Package') {
    steps {
        echo "Packaging ${IMAGE_NAME} version ${IMAGE_TAG}"
        dir('app') {
            sh 'echo "Contents about to be packaged:" && ls -la'
        }
    }
}
```

**Take-home 3** — a parameterised image tag. Note `?:`, Groovy's **Elvis operator**: "use the left value, or the right if the left is empty" — the same idea as bash's `${2:-default}` from Session 2.

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
                dir('app') {
                    sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }
    }
}
```

**Take-home 8** — the most useful directive we didn't cover is `when`, making a stage **conditional**:

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

You'd use it so feature branches get built and tested but only `main` actually deploys — a very common real-world pattern, and exactly what Session 8 builds properly.

Also worth finding:
- **`options`** — e.g. `timeout(time: 30, unit: 'MINUTES')` so a hung build doesn't occupy an executor forever
- **`triggers`** — declaring `pollSCM('H/2 * * * *')` in the `Jenkinsfile` rather than clicking it in the UI, so even the trigger is version-controlled

<br>
<br>

### Conclusion

- **Inform** students this marks the end of the Jenkins & CI/CD Pipelines session
- **Confirm** everyone's `pipeline-glossary.md` is complete and committed
- **Reinforce** that every mechanism Session 5 needs — credentials, stages, gating, `post` blocks, pipeline-as-code — already exists in what they built today; **only the commands inside the stages change**
- **Remind** them to `docker stop jenkins` if they want their laptop's resources back, and that the named volume means nothing is lost
- **Preview** the next session directly: the same pipeline shape, running **Terraform against Azure**, with a manual approval gate before `apply`
- **Direct** students to the take-home exercises and the [Jenkins Pipeline documentation](https://www.jenkins.io/doc/book/pipeline/)

---

[Back](./README.md)

---
