# Jenkins & CI/CD Pipelines — Trainer Script

A full day taking entry-level trainees from "I've heard the word pipeline" to "I've built one that tests, builds and ships a Docker image automatically". The emphasis throughout is on **what a pipeline actually is, what it's for, and why teams rely on them** — the mechanics of Jenkins are the vehicle, not the point. This is written as a script — read it out, adapt where it feels natural, but the intent is that you could deliver this cold from the page.

## Organisation

### Audience

Entry-level DevOps trainees who have already covered **Azure cloud fundamentals**, **Docker**, and **bash scripting & automation**. Assume they can build and run a Docker image, write a small shell script, and use Git at a basic level (clone, commit, push). Assume **no** prior exposure to Jenkins or any CI/CD tool. Jenkins runs on **each student's own laptop** — Windows or Mac — so a chunk of today is making that work reliably.

### Duration & schedule

Full day, **09:00–17:00**, with a one-hour lunch and two 15-minute breaks. Roughly **4 hours of instruction and 3 hours of hands-on**, weighted towards an end-of-day build-and-push capstone.

| Time | Section | Shape |
|---|---|---|
| 09:00–09:15 | Welcome & recap: where we are | Talk |
| 09:15–10:00 | What CI/CD is, what pipelines are *for*, and why | Talk + discussion |
| **10:00–10:15** | **Break** | |
| 10:15–11:15 | Installing & touring Jenkins on your laptop | **Exercise (install) + tour** |
| 11:15–12:15 | Your first job: watching Jenkins build something | **Exercise** |
| 12:15–13:00 | Pipeline as code: your first `Jenkinsfile` | **Exercise** |
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
- **Write** a declarative `Jenkinsfile` with stages, environment variables, parameters, `post` actions, and parallel stages
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

In your cloned starter repo there's a `Dockerfile` for this (or create one). It's four lines:

```dockerfile
FROM jenkins/jenkins:lts-jdk17
USER root
RUN apt-get update && apt-get install -y docker.io && rm -rf /var/lib/apt/lists/*
USER jenkins
```

Read it — this is exactly the Docker knowledge you already have: start from the official Jenkins image, become root, install the Docker CLI, drop back to the jenkins user. Build it:

```bash
docker build -t jenkins-docker .
```

**Step 2 — run it**

On **Mac** (Terminal) or **Windows** (use **Git Bash**, so the `\` line-breaks work):

```bash
docker run -d --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -u root \
  jenkins-docker
```

If you'd rather paste a single line (works in any shell, including PowerShell):

```bash
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -u root jenkins-docker
```

What each part does:
- `-d` — run in the background (detached)
- `-p 8080:8080` — the Jenkins web interface, which we'll open in a browser
- `-p 50000:50000` — the port Jenkins uses to talk to agents (unused today, but standard)
- `-v jenkins_home:/var/jenkins_home` — a **named volume** so all your Jenkins config survives if the container is recreated. You met volumes in the Docker session — this is why they matter: without it, restart Jenkins and everything you built today is gone.
- `-v /var/run/docker.sock:/var/run/docker.sock` — this hands the container access to your laptop's Docker, so pipelines can build images. This is the bit that makes the capstone possible.
- `-u root` — run as root so we sidestep Docker permission faff. Fine for training; **not** how you'd do it in production.

**NOTE FOR TRAINERS** <br>
On both Docker Desktop for Mac and Docker Desktop for Windows (WSL2 backend), `/var/run/docker.sock` is exposed and the socket mount works as written — this is the reliable cross-platform path. Running `-u root` is a deliberate simplification to avoid the docker-group permission dance in a classroom; call it out honestly as a training shortcut so nobody copies it into a real setup. If a student's install refuses the socket mount, they can still do the *entire* day except the final `docker build`/`docker push` stages, so don't let one broken socket block someone — pair them up. <br>
**END OF NOTE**

**Step 3 — unlock and set up**

Jenkins takes a minute or two to start. Then:

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Copy that password, open **`http://localhost:8080`** in your browser, and paste it in. Then:
- Choose **Install suggested plugins** — this pulls in the Git, Pipeline and Credentials plugins we'll use all day. Give it a few minutes.
- Create your first **admin user** when prompted (remember the username and password — you'll log in with it all day).
- Accept the default Jenkins URL.

**ASK** <br>
We deliberately mounted a named volume at `/var/jenkins_home`. Based on the Docker session, what would happen to your admin user, your plugins, and every job you build today if we *hadn't*? <br>
**ANSWER** <br>
They'd all vanish the moment the container was removed and recreated, because a container's own filesystem is disposable. The named volume stores Jenkins' data on the host, outside the container's lifecycle, so it persists. This is the same reason a database container needs a volume — state that must survive belongs outside the container.

**A quick tour of the interface**

Now let's get oriented. Point these out on your own screen as I describe them:

- **Dashboard** — the home page; every job you create shows here with its recent status (blue/green = passing, red = failing).
- **New Item** — how you create a new job. "Item" is just Jenkins' word for a job.
- **Manage Jenkins** — the settings hub. Two areas we care about:
  - **Plugins** — almost everything Jenkins can do comes from plugins. The "suggested" set you just installed is only a starter kit.
  - **Credentials** — a secure vault for secrets (Git tokens, Docker Hub passwords). We'll use this properly this afternoon, and it's a genuinely important habit.
- **Build History** — a running log of every build across all jobs.

**HANDS ON (remaining time)** <br>
1. Get Jenkins running via the steps above and log in with your new admin user.
2. Go to **Manage Jenkins > Plugins > Installed plugins**. Scroll the list and pick **three plugins you don't recognise** — search their names and jot down, in a sentence each, what they do.
3. Go to **Manage Jenkins > Nodes**. You'll see one node ("Built-In Node"). This is the controller acting as its own builder — exactly the "fine for learning, not for production" setup we described. Note how many executors it has.
**END OF NOTE**

<br>
<br>

### 11:15–12:15 — Your First Job: Watching Jenkins Build Something
*(Exercise)*

We're going to make Jenkins actually *do* something now, using the simplest possible job type. It's not how we'll write pipelines for real, but it makes the core mechanic — "Jenkins runs your steps and shows you the output" — completely visible before we add any syntax on top.

**A Freestyle job**

A **Freestyle project** is a job configured entirely by clicking in the web UI. No code. Let's make one.

- **New Item** > name it `hello-jenkins` > choose **Freestyle project** > **OK**
- Scroll to **Build Steps** > **Add build step** > **Execute shell**
- In the box, enter:

```bash
echo "Hello from Jenkins!"
echo "This build is running as: $(whoami)"
echo "The time is: $(date)"
```

- **Save**, then click **Build Now** on the left.
- A build appears under **Build History** (bottom left) with a number, `#1`. Click it, then click **Console Output**.

There it is — the exact output of your shell commands, captured and shown in the browser. That's the entire heart of Jenkins: it ran your steps somewhere, and kept the log.

**ASK** <br>
Where in an *earlier* session did we see almost exactly this pattern — a set of shell commands running unattended, where we could only see what happened by looking at a log afterwards? <br>
**ANSWER** <br>
`cron` in the automation session — a scheduled, unattended script whose output you only see by checking a log file. Jenkins' Console Output is doing the same job, just with a proper interface and a lot more built around it. If you understood why cron logs matter, you already understand why Jenkins keeps build logs.

**The workspace and built-in variables**

Every build runs inside a **workspace** — a folder on disk where Jenkins does its work. And Jenkins injects useful **environment variables** into every build automatically. Let's use one.

- **Configure** the job again, and change the shell step to:

```bash
echo "This is build number $BUILD_NUMBER"
echo "Build $BUILD_NUMBER of job $JOB_NAME" > result.txt
cat result.txt
```

- **Save** and **Build Now** a couple of times. Open the Console Output each time.

**ASK** <br>
If you run this job five times, what will `result.txt` say each time, and why? <br>
**ANSWER** <br>
A different number each run — `$BUILD_NUMBER` increases by one every build. Jenkins gives every build a unique, ever-incrementing number, which becomes really useful later for tagging things (like Docker images) so you can trace exactly which build produced which artifact.

**Keeping the output**

Right now `result.txt` is buried in a workspace folder. We can publish it so it's downloadable from the build page:

- **Configure** > **Post-build Actions** > **Add** > **Archive the artifacts**
- **Files to archive**: `result.txt`
- **Save**, **Build Now**, and notice the archived `result.txt` now appears as a link on the build's page.

**The catch with Freestyle — and why we're about to leave it**

We've got a working job. But let's poke at it the way we poke at everything in this course.

**ASK** <br>
All of this configuration — the shell commands, the archive setting, the build number logic — where does it actually *live*? <br>
**ANSWER** <br>
Inside Jenkins itself, in its own internal config. **Not** in your Git repository.

**ASK** <br>
Given everything this course has hammered about wanting things "as code" — reviewable, versioned, repeatable — what problems does that cause? <br>
**ANSWER** <br>
No version history of how the build process changed. No code review before someone alters it. No easy way to copy it to another project or run different behaviour on different branches. If someone clicks the wrong thing, there's no diff and no undo. It's ClickOps all over again — the exact problem we called out with the Azure Portal, now wearing a Jenkins costume.

That gap is what **Pipelines** fix, and that's where we're going next.

**HANDS ON (remaining time)** <br>
1. Build the `hello-jenkins` job above, including the `$BUILD_NUMBER` version and the archived artifact.
2. **Configure** the job > **Build Triggers** > tick **Build periodically** and enter `H/5 * * * *` (that's cron syntax from the automation session — roughly every 5 minutes). Watch it trigger itself on its own. Then **turn it off again** so it's not running all day.
3. In pairs, discuss: if three people on your team disagreed about what the build steps should do, how would you even resolve that disagreement with a Freestyle job? (Hint: there's nothing to review, comment on, or merge.)
**END OF NOTE**

<br>
<br>

### 12:15–13:00 — Pipeline as Code: Your First `Jenkinsfile`
*(Exercise)*

Here's the big shift. Instead of configuring a job by clicking, we describe the whole pipeline in a **text file** called a `Jenkinsfile`, and we keep that file **in Git, alongside the code it builds**.

**ASK** <br>
Where have we seen this exact philosophy already — describing something as a text file in version control instead of clicking it into existence? <br>
**ANSWER** <br>
Everywhere this course goes: the bash scripts you wrote, the Dockerfile that defines an image, and — coming up soon — Terraform's `.tf` files. Same instinct every time: the desired thing described as reviewable, versioned, repeatable code, not clicked together by hand and forgotten.

A `Jenkinsfile` can be written two ways — **Scripted** (full programming language, very flexible, harder to read) and **Declarative** (a cleaner, more structured style). We'll use **Declarative** the whole day; it's the recommended starting point and covers the vast majority of real pipelines.

**The simplest possible pipeline**

Let me show you the shape, then you'll build it:

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
- `pipeline { }` — every declarative pipeline is wrapped in this. Always.
- `agent any` — "run this on any available builder." Today that's our controller.
- `stages { }` — the ordered list of phases. This is the assembly line.
- `stage('Say Hello') { }` — one phase, with a name that'll show up in the UI.
- `steps { }` — the actual things to do in that stage. `echo` just prints.

**Run it**

For now we'll paste it straight into Jenkins (we'll move it into Git this afternoon):

- **New Item** > name it `hello-pipeline` > choose **Pipeline** > **OK**
- Scroll to the **Pipeline** section. Leave **Definition** as **Pipeline script**.
- Paste the pipeline above into the **Script** box.
- **Save**, then **Build Now**.

Look at the build — Jenkins now shows a little **Stage View**: a box for "Say Hello" with a green tick. That visual is a real benefit: as pipelines grow, you can see *at a glance* exactly which stage passed and which one broke.

**Add a second stage so you can see the assembly line**

- **Configure** and change the pipeline to:

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

Note `sh` — that runs an actual shell command, exactly like a step in a bash script. (This is why we run Jenkins in a Linux container: `sh` behaves identically for everyone, whether you're on Windows or Mac.)

- **Save**, **Build Now**, and watch the Stage View now show **two** boxes, Build then Test, lighting up green in order.

**ASK** <br>
Why is it genuinely useful that each stage shows up separately in the UI, rather than the whole build just being "passed" or "failed"? <br>
**ANSWER** <br>
When something breaks, you instantly see *where* — "Test failed" tells you far more than "the build failed". On a real pipeline with checkout, build, test, package and deploy stages, that pinpointing saves a lot of time, and it's why we split work into named stages even when we technically could cram it into one.

**HANDS ON (remaining time)** <br>
1. Create `hello-pipeline` and get the two-stage version running with a green Stage View.
2. Add a **third** stage called `Package` that just echoes a message.
3. Make one of the `sh` steps deliberately fail — change it to `sh 'exit 1'` — rebuild, and watch that stage go **red** and the stages *after* it **not run at all**. This is the single most important thing a pipeline does: **stop the line when something's wrong, before the broken thing goes any further.** Then fix it back to green.
**END OF NOTE**

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

The `environment { }` block defines variables available to every stage. Change `APP_NAME` in one place and it updates everywhere.

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

Once a pipeline has `parameters`, the job's button changes from **Build Now** to **Build with Parameters**, and Jenkins prompts for a value each time. Handy for "deploy to dev vs prod".

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
            echo 'It worked! 🎉'
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

The `post` block runs *after* all your stages. `success` only runs if everything passed, `failure` only if something broke, `always` runs no matter what.

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

**HANDS ON (remaining time)** <br>
Take your `hello-pipeline` and, one at a time, add:
1. An `environment` block with an `APP_NAME`, and use it inside a stage's `echo`.
2. A `post` section with `success`, `failure` and `always` blocks. Then make a step fail (`sh 'exit 1'`) and confirm `failure` and `always` run but `success` does not. Fix it and confirm the opposite.
3. A `parameters` block with a `string` parameter, and echo `params.YOURPARAM` in a stage. Notice the button change to **Build with Parameters**.
**END OF NOTE**

<br>
<br>

### 14:45–15:00 — Connecting Jenkins to Git, and Triggering Builds
*(Talk + demo)*

Our `Jenkinsfile` is still pasted into the Jenkins UI — which has the *exact same* "not really code" problem as the Freestyle job. Let's fix the concept now, so the capstone can do it for real.

**Pulling the `Jenkinsfile` from Git**

Instead of pasting the script, we point Jenkins at a Git repo and tell it "the `Jenkinsfile` is in there":

- On a Pipeline job: **Configure** > **Pipeline** section > change **Definition** to **Pipeline script from SCM**
- **SCM**: Git
- **Repository URL**: your fork of the starter repo
- **Branch Specifier**: `*/main`
- **Script Path**: `Jenkinsfile` (the default — the file at the repo root)

Now the pipeline definition lives in Git: versioned, reviewable in a Pull Request, and identical for anyone who runs it. This is the whole reason pipeline-as-code beats clicking.

**Handling secrets — the Credentials store**

To pull a private repo (or push to Docker Hub later), Jenkins needs secrets. We never paste those into a config box or, worse, into the `Jenkinsfile` in Git. We use Jenkins' **Credentials** store.

- **Manage Jenkins > Credentials > System > Global credentials > Add Credentials**
- We'll set one up properly together in the capstone.

**ASK** <br>
Why store a token in the Credentials vault instead of just typing it into the repository URL field, or the `Jenkinsfile`? <br>
**ANSWER** <br>
The vault keeps the secret out of both the job config *and* out of Git, it automatically **masks** the value if it would otherwise show up in a build log, and it lets you rotate the secret in one place without editing anything else. A token committed to Git is a genuine security incident — the vault exists so that never has to happen.

**Triggering builds automatically**

The magic of CI is that nobody clicks "Build". A change to Git triggers it. Two ways:

- **Poll SCM** — Jenkins checks the repo on a schedule (cron syntax again) and builds only if something changed. Simple, works everywhere, but there's a delay and it's a bit wasteful.
- **Webhook** — GitHub *tells* Jenkins the instant a push happens. Instant, efficient, and how real setups do it.

**ASK** <br>
Between polling every minute and a webhook, which is better, and why? <br>
**ANSWER** <br>
The webhook. Polling means Jenkins repeatedly asks "anything changed yet?" when the answer is almost always no — wasted checks, plus a delay of up to the polling interval. A webhook means GitHub notifies Jenkins the moment something actually happens: no waste, no delay.

**NOTE FOR TRAINERS** <br>
Here's the local-laptop reality: a webhook from GitHub can't reach `http://localhost:8080` on a student's machine, because GitHub is on the public internet and their Jenkins isn't. The proper fix is a tunnel like [ngrok](https://ngrok.com/), which is a great optional stretch. For the guaranteed-to-work classroom path, we use **Poll SCM** in the capstone — it needs no networking magic and demonstrates the same "builds trigger themselves from Git" principle. Teach the webhook as the concept and the goal; use polling as the reliable hands-on. <br>
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

1. In your forked starter repo you have a small Node app with a `package.json`, a trivial test (`npm test` passes), and a `Dockerfile`. Clone it locally if you haven't, and have a look at those three files so you know what the pipeline will be acting on. (If you'd rather, use the `countries` app you scaffolded in the bash session — it already has a `Dockerfile`.)
2. On **Docker Hub**: go to **Account Settings > Security > New Access Token**, create one with **Read & Write**, and copy it somewhere safe for a minute. This is what the pipeline pushes with — never your account password.

#### Part 1 (≈15 min) — Store your Docker Hub credentials in Jenkins

1. **Manage Jenkins > Credentials > System > Global credentials > Add Credentials**
2. **Kind**: Username with password
3. **Username**: your Docker Hub username
4. **Password**: the access token you just created
5. **ID**: `dockerhub-credentials` (we'll refer to this exact ID from the `Jenkinsfile`)
6. Save.

**Checkpoint question — ask yourself:** why did we put the token *here* and not in the `Jenkinsfile` we're about to write and push to Git?

#### Part 2 (≈35 min) — Write the pipeline

Create a file called `Jenkinsfile` at the root of your repo with this content, replacing `your-dockerhub-username`:

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

- **`checkout scm`** — checks out the same repo the `Jenkinsfile` came from. When the pipeline runs from Git, this pulls the code automatically.
- **the `Install & Test` stage has its own `agent { docker { image 'node:20' } }`** — this stage runs *inside a fresh `node:20` container*. That's huge: the stage gets a clean, correct Node environment every time, with no need to install Node on the Jenkins host. It's the "disposable, identical environment per build" idea from the Docker session, applied to CI.
- **`$IMAGE_TAG` = `$BUILD_NUMBER`** — every image is tagged with the build number, so you can trace exactly which build produced which image. That's the `$BUILD_NUMBER` you met this morning, now doing real work.
- **`withCredentials`** — pulls your Docker Hub secret out of the vault *only* for the steps inside it, binds it to `DOCKER_USER`/`DOCKER_PASS`, and masks it in the logs. `--password-stdin` avoids the password ever appearing on a command line.

Commit and push this `Jenkinsfile` to your fork.

#### Part 3 (≈20 min) — Point Jenkins at it and run it

1. **New Item** > `build-and-push` > **Pipeline** > OK.
2. **Pipeline** section > **Definition**: **Pipeline script from SCM** > **Git** > your fork's URL > branch `*/main` > **Script Path**: `Jenkinsfile`.
3. **Build Triggers** > tick **Poll SCM** > schedule `H/2 * * * *` (checks every couple of minutes).
4. **Save**, then **Build Now** for the first run.
5. Watch the Stage View march through Checkout → Install & Test → Build Image → Push Image. When it's green, go to your **Docker Hub** account and confirm the image is there, tagged with the build number.
6. Now make a trivial change in your repo (edit the README), commit and push, and **wait** — within a couple of minutes Poll SCM should kick off a build **on its own**. That moment — a build you didn't start, triggered purely by a Git push — is CI working. Sit with it; that's the whole point of the day.

**ASK** *(mid-capstone checkpoint, ~16:15)* <br>
Look at the order of your stages: Test comes *before* Build and Push. Why does that order matter — what does putting Test first actually protect you from? <br>
**ANSWER** <br>
If the tests fail, the pipeline stops right there and **never builds or pushes the image**. A broken version can't reach Docker Hub, because the gate caught it first. That's the "stop the line before the bad thing ships" principle — the same reason your bash `containerise` script checked exit codes. Ordering stages so the cheap checks fail fast, before the expensive/irreversible steps, is a core pipeline design habit.

#### Part 4 — Stretch goals

If you've got a green, self-triggering pipeline, level it up:

1. **Prove the gate works.** Add a stage *before* Build that deliberately fails (`sh 'exit 1'`), push it, and confirm nothing gets built or pushed — the pipeline stops at the red stage. Then remove it. This is worth doing: *seeing* the pipeline refuse to ship broken code is more convincing than being told it will.
2. **Add a `latest` tag.** In the Push stage, also tag and push `$IMAGE_NAME:latest` alongside the numbered tag, so there's always a "newest" image.
3. **Parallelise the checks.** If you add a linting step, run it in `parallel` with the tests.
4. **Notify on failure.** Even just a richer `echo` in the `failure` block that prints `${IMAGE_NAME}` and the build URL — then read about the Slack/email plugins that would do this for real.
5. **Webhook instead of polling** (advanced): install `ngrok`, expose your local Jenkins, and set up a GitHub webhook so pushes trigger builds *instantly* instead of on a 2-minute poll.

#### What to show at 16:45

Be ready to demonstrate: push a change, and (via Poll SCM) watch a build start on its own, go green through all stages, and result in a freshly-tagged image in your Docker Hub. And be ready to explain, in your own words, *why* the Test stage comes before the Push stage.

<br>
<br>

### 16:45–17:00 — Wrap-up & Q&A

Let's pull the day together.

You started this morning with a definition — CI/CD is about replacing scary, manual, occasional releases with automatic, safe, frequent ones — and a mental model: a pipeline is an assembly line of stages, where each one has to pass before the next runs, and the line stops the moment something breaks.

Then you made it real. You installed Jenkins on your own laptop, ran your first job, and watched the crucial shift from clicking a job together in the UI to describing it as a `Jenkinsfile` in Git — reviewable, versioned, repeatable, the same "as code" principle behind every tool in this course. And in the capstone you built the genuine article: a pipeline that, on a git push, tests an app, builds an image, and ships it to a registry, all on its own — refusing to push anything if the tests fail.

**ASK** <br>
Look at the shape of that capstone pipeline: checkout, then a stage that tests, then a stage that changes something real, then a stage that ships it — with credentials pulled from the vault and a `post` block reporting the outcome. If we wanted this *same* pipeline to run **Terraform against Azure** instead of building a Docker image, how much would actually have to change? <br>
**ANSWER** <br>
Structurally, almost nothing. The credentials pattern is identical — you'd store an Azure Service Principal's details in the vault instead of a Docker Hub token. The `sh` steps would run `terraform init`, `terraform plan`, `terraform apply` instead of `docker build`/`docker push`. The stages, the gating, the `post` block — all the same. That's not a coincidence: it's exactly the next session, and everything you built today is the groundwork it stands on.

Where this sits in the course:
- **Today** — you can explain and build a pipeline, and run Jenkins yourself
- **Terraform & IaC on Azure** — the same pipeline shape, now running Terraform to provision real infrastructure, with a human approval gate before anything is applied
- **Kubernetes** — running and scaling the images your pipeline now builds
- **Integration** — everything woven together: a push to Git flowing through test, build, provision and deploy, automatically and safely

**Q&A** — take remaining questions. And remind everyone to `docker stop jenkins` when they're done if they want their laptop's resources back — the named volume means it'll all still be there next time with `docker start jenkins`.

<br>
<br>

### Exercise (take-home / reinforcement)

Before the next session, individually or in pairs:

1. From scratch, in a brand new folder, get Jenkins running and build a two-stage pipeline from memory — no notes. Rebuilding is the fastest way to make it stick.
2. Take the capstone pipeline and add a **`Package` stage** between Test and Build that just prints the version being built — practising adding a stage cleanly.
3. Add a `parameters` block letting you pass in the image tag manually, defaulting to the build number, and use it in the Build and Push stages.
4. Write a short paragraph, in your own words, explaining the difference between Continuous Delivery and Continuous Deployment, with an example of a team that would prefer each.
5. (Stretch) Set up `ngrok` and a GitHub webhook so a push triggers your pipeline **instantly**, and compare the feel against Poll SCM.
6. (Stretch) Read the [Jenkins Pipeline syntax docs](https://www.jenkins.io/doc/book/pipeline/syntax/) and find one directive we didn't cover today (e.g. `when`, `options`, `tools`) — work out what it does and where you'd use it.

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
