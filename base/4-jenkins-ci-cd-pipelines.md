# Introduction to Jenkins

A full day building up real Jenkins fundamentals — CI/CD concepts, installation, jobs, pipelines, and integrating with Git and Docker — so that wiring Terraform and Azure into a pipeline in the next session feels like a natural next step, not a cold start.

## Organisation

### Duration

Full day, **09:30 - 17:00**, split into four sessions with a mid-morning break, lunch, and a mid-afternoon break.

| Time | Session |
|---|---|
| 09:30 - 11:00 | Session 1: CI/CD Fundamentals & Installing Jenkins |
| 11:00 - 11:15 | Break |
| 11:15 - 12:45 | Session 2: Freestyle Jobs and How Jenkins Builds Things |
| 12:45 - 13:30 | Lunch |
| 13:30 - 15:00 | Session 3: Pipelines and the Jenkinsfile |
| 15:00 - 15:15 | Break |
| 15:15 - 16:45 | Session 4: Git, Agents, and a Real Build-and-Push Pipeline |
| 16:45 - 17:00 | Wrap-up & Q&A |

### Set-up

#### For Trainers

**For the slidee presentation** <br>
**Open** the [Slidee](https://github.com/mklilley/slidee) presentation:

- Run `npx slidee` from within the `curriculum` folder to see the available slide decks (those with extension `.slides.md`)
- Click on `See your presentations`
- Open `Weeks X Y > CDO > Introduction to Jenkins`
- Hit `s` to open the presenter notes
- Resource to be kept open on a tab throughout the day to refer to

**Check out** [FAQs](/FAQs/README.md) for help with using Slidee

#### For Students
- **Direct** students to the starter code for this module
  - [Student Facing Repo](https://github.com/LaFosseAcademy/cloud-devops-student-repos/tree/main)
- **Use**
  - **/jenkins/intro-to-jenkins/starter-code**
- **Make sure**
  - Docker is installed and running
  - Students have a GitHub account and have forked the starter repo
  - Students have a free [Docker Hub](https://hub.docker.com) account for Session 4
  - No Terraform or Azure needed today — that's next session

## Learning objectives

- **Understand** what CI/CD means in practice, and where Jenkins fits among the tools that provide it
- **Install and configure** a working Jenkins instance
- **Create and run** both Freestyle jobs and Pipeline jobs
- **Write** a `Jenkinsfile` using declarative pipeline syntax, including stages, environment variables, and post actions
- **Understand** the difference between Jenkins' controller and its build agents
- **Connect** Jenkins to a Git repository and trigger builds automatically
- **Build** a complete pipeline that tests, builds, and pushes a Docker image — laying the exact groundwork the next session builds on with Terraform and Azure

<br>

---

## Session 1: CI/CD Fundamentals & Installing Jenkins
### 09:30 - 11:00

### Sequence

#### What does CI/CD actually mean?

We've used the word "pipeline" loosely in earlier sessions. Let's properly unpack it.

* **Continuous Integration (CI)** — every time a developer pushes code, it's automatically built and tested. The goal: catch problems within minutes of them being introduced, not weeks later.
* **Continuous Delivery (CD)** — every change that passes CI is automatically prepared for release (packaged, versioned) and ready to deploy at the push of a button.
* **Continuous Deployment (CD)** — the stricter version: every change that passes CI is automatically deployed to production, with no human in the loop at all.

**ASK** <br>
What's the actual difference between Continuous *Delivery* and Continuous *Deployment*? <br>
**ANSWER** <br>
Delivery still has a human decide *when* to release (even if the release itself is a single click); Deployment removes that decision entirely — every passing change goes live automatically.

*REFER TO RESOURCE 1 - SLIDEE* <br>
![intro-to-jenkins-1](./resources/intro-to-jenkins-1.png)

**ASK** <br>
Thinking back to the DevOps mantra "you build it, you run it" — how does CI/CD support that? <br>
**ANSWER** <br>
It removes the human bottleneck between "code is finished" and "code is running somewhere real" — the same engineers who write the code get fast, automatic feedback on whether it actually works, rather than throwing it over a wall and waiting.

#### Where Jenkins fits

**Jenkins** is an open-source **automation server** — its job is to run tasks automatically in response to events, most commonly a change in a Git repository. It doesn't know or care what those tasks are: running tests, building a Docker image, deploying to a server, or — as we'll build towards over the next two sessions — running Terraform.

**ASK** <br>
Jenkins isn't the only tool in this space. What else might do this job? <br>
**ANSWER** <br>
GitHub Actions, GitLab CI, CircleCI, Azure Pipelines. Jenkins is one of the oldest and most widely adopted — and critically for today, it's **self-hosted**, meaning every part of it is something we install, configure, and can inspect directly, rather than a managed black box.

#### Jenkins' architecture: controller and agents

*REFER TO RESOURCE 2 - SLIDEE* <br>
![intro-to-jenkins-2](./resources/intro-to-jenkins-2.png)

Jenkins has two conceptual pieces:

* The **controller** (historically called the "master") — runs the Jenkins web UI, stores configuration, schedules work, but ideally doesn't run the actual build steps itself
* **Agents** (historically "slaves" or "nodes") — separate machines (or containers) that the controller hands work off to, actually running the `sh` steps, compiling code, and so on

**ASK** <br>
Why might we not want the controller itself running every build directly? <br>
**ANSWER** <br>
A single build with a runaway process, or one that fills up disk space, could take down the entire Jenkins server for every team relying on it — agents isolate that risk, and let us scale out horizontally by adding more agents as build demand grows.

For today, we'll run everything on a single machine without dedicated agents — Jenkins defaults to using the controller itself as a build executor when nothing else is configured, which is fine for learning, but not how a production setup should look.

#### Installing Jenkins

The fastest way to get a working Jenkins instance for training purposes is via Docker.

* Run:
```bash
docker run -d --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

* `-p 8080:8080` — the Jenkins web UI
* `-p 50000:50000` — used for communication with any agents we attach
* `-v jenkins_home:/var/jenkins_home` — a named Docker volume, so our configuration survives if the container is removed and recreated

**NOTE FOR TRAINERS** <br>
Worth mentioning briefly that this is one of several installation routes — Jenkins also ships as a native package for Linux distributions, a WAR file runnable with any Java installation, and there are managed/semi-managed offerings on most clouds too. We're using Docker purely because it's the fastest, most consistent route to a working instance across everyone's machines in a training room. <br>
**END OF NOTE**

Jenkins takes a minute or two to start. Once it's up:

* Grab the initial admin password:
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

* Visit `http://localhost:8080`, paste in the password
* Choose **Install suggested plugins** — this pulls in the common Git, Pipeline, and credentials plugins we'll rely on all day
* Create your first admin user when prompted

#### A tour of the Jenkins UI

* **Dashboard** — every job ("Item") we've created, with its recent build status
* **New Item** — create a new job
* **Manage Jenkins** — the umbrella for system configuration:
  * **Plugins** — Jenkins' functionality is almost entirely plugin-driven; the "suggested plugins" we just installed are themselves just a starter set
  * **Credentials** — a secure store for secrets Jenkins needs (we'll use this properly in Session 4)
  * **Nodes** — where agents are registered and managed
* **Build History** — a running log of every build across a job, with status and duration

*REFER TO RESOURCE 3 - SLIDEE* <br>
![intro-to-jenkins-3](./resources/intro-to-jenkins-3.png)

#### Exercise
Students:
1. Get Jenkins running via Docker on their own machine and log in
2. Explore **Manage Jenkins > Plugins > Installed plugins** and identify three plugins they don't recognise — look up what they do
3. Explore **Manage Jenkins > Nodes** and note what's listed as the default executor

<br>

---

## Session 2: Freestyle Jobs and How Jenkins Builds Things
### 11:15 - 12:45

### Sequence

#### Our first job: Freestyle

Jenkins supports several job types. The oldest and simplest is a **Freestyle project** — entirely configured through the web UI, no code involved.

**NOTE FOR TRAINERS** <br>
We're deliberately starting with Freestyle, even though we'll move away from it later today, because it makes the *mechanics* of "Jenkins runs a job and shows you the output" completely visible before we add the extra layer of Pipeline/Groovy syntax on top. <br>
**END OF NOTE**

* **New Item** > name it `hello-world-freestyle` > choose **Freestyle project** > **OK**
* Under **Build Steps** > **Add build step** > **Execute shell**
* Enter:
```bash
echo "Hello from Jenkins!"
date
whoami
```

* **Save**, then **Build Now**
* Click into the build that appears under **Build History**, then **Console Output**

**ASK** <br>
Where else in this course have we seen almost this exact pattern — a shell script running unattended, with its output only visible in a log afterwards? <br>
**ANSWER** <br>
`cron`, from the Linux and Automation session — a scheduled, unattended script whose output we only see by checking a log. Jenkins' Console Output is effectively doing the same job, just with a proper UI wrapped around it.

#### Build triggers

Right now, our job only runs when we click **Build Now**. Let's look at other ways to trigger it.

* **Configure** our job > **Build Triggers**:
  * **Build periodically** — uses the exact same cron syntax from the Linux and Automation session (`minute hour day month weekday`)
  * **Poll SCM** — Jenkins checks the repository on a schedule (also cron syntax) and only builds if something's actually changed
  * **GitHub hook trigger for GITScm polling** — the repository notifies Jenkins the instant something changes, via a webhook (we'll wire this up properly in Session 4)

**ASK** <br>
Between "Poll SCM every minute" and "trigger via webhook", which is more efficient, and why? <br>
**ANSWER** <br>
Webhooks — polling means Jenkins is repeatedly asking "has anything changed yet?" even when the answer is almost always no, wasting requests and introducing a delay of up to the polling interval; a webhook means GitHub tells Jenkins the instant something actually happens, with no wasted checks and no delay.

#### Build steps and the workspace

Every build runs inside a dedicated **workspace** — a directory on disk where Jenkins checks out code and runs our steps.

* Let's add a build step that creates a file:
```bash
echo "build number: $BUILD_NUMBER" > result.txt
cat result.txt
```

* `$BUILD_NUMBER` is one of several environment variables Jenkins automatically injects into every build

**ASK** <br>
If we run this job five times, what would we expect `result.txt` to contain each time? <br>
**ANSWER** <br>
A different build number each time — `$BUILD_NUMBER` increments with every run, and by default each build gets a fresh workspace state for its steps (though the workspace directory itself is often reused between builds unless explicitly cleaned).

We can publish files out of the workspace so they're accessible from the Jenkins UI after the build finishes:

* **Post-build Actions** > **Archive the artifacts** > pattern: `result.txt`
* **Build Now**, then look for the archived artifact link on the completed build's page

#### The limits of Freestyle

We've now built a small, working Freestyle job — but let's stress-test the idea the same way we did with bash scripts back in the Linux and Automation session.

**ASK** <br>
Our Freestyle job's configuration — the shell commands, the triggers, the archived artifact pattern — where does all of that actually live? <br>
**ANSWER** <br>
Inside Jenkins itself, in its internal configuration, not in our Git repository.

**ASK** <br>
What problems does that cause, given everything we've said this course about wanting infrastructure (and now pipeline) *as code*? <br>
**ANSWER** <br>
No version history of pipeline changes, no code review before a change to the build process goes live, no way to reuse or fork a pipeline definition easily, and no way to have different pipeline behaviour per branch without manually duplicating jobs.

This is exactly the gap **Pipeline** jobs close — and it's where we're headed for the rest of the day.

#### Exercise
Students:
1. Create a Freestyle job that checks out a small public Git repository (use **Source Code Management > Git** and any public repo URL) and runs a shell step against it
2. Configure it to poll that repository every 2 minutes using cron syntax
3. Discuss in pairs: if three people on a team all had slightly different opinions about what the build steps should do, how would that disagreement even get resolved with a Freestyle job?

<br>

---

## Session 3: Pipelines and the Jenkinsfile
### 13:30 - 15:00

### Sequence

#### Pipeline as Code

A **Pipeline** job is defined in a text file, checked into Git alongside our application or infrastructure code, called a `Jenkinsfile`. 

**ASK** <br>
Where have we seen this exact philosophy before, just applied to a different kind of file? <br>
**ANSWER** <br>
Terraform's `.tf` files — the desired state described as code, reviewable, versioned, and repeatable, rather than configured by hand through a UI.

There are two syntax styles: **Scripted Pipeline** (full Groovy, very flexible, more verbose) and **Declarative Pipeline** (a more structured, opinionated syntax, easier to read and lint). We'll be using **Declarative** — it covers the vast majority of real-world use cases and is the recommended starting point.

#### Anatomy of a declarative Jenkinsfile

Let's build one up piece by piece.

**Jenkinsfile**
```groovy
// NEW CONFIG
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello from a Pipeline job!'
            }
        }
    }
}
```

* `pipeline { }` — the outer block, every declarative pipeline starts here
* `agent any` — run this pipeline on any available executor (we'll look at more specific agent configuration in Session 4)
* `stages { }` — a named sequence of steps; each `stage` shows up individually in the Jenkins UI, which makes it easy to see exactly which part of a build failed
* `steps { }` — the actual commands run within a stage

Let's create a Pipeline job to run this:

* **New Item** > name it `hello-world-pipeline` > choose **Pipeline** > **OK**
* Under **Pipeline**, leave **Definition** as `Pipeline script`, and paste our Jenkinsfile content directly into the box for now (we'll switch to pulling it from Git shortly)
* **Save**, then **Build Now**

Notice the UI now shows each `stage` as a distinct segment — this becomes much more useful as pipelines grow.

#### Adding more stages, environment variables, and parameters

**Jenkinsfile**
```groovy
pipeline {
    agent any

    // NEW CONFIG
    environment {
        GREETING = 'Hello'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Pretending to check out code...'
            }
        }

        stage('Build') {
            steps {
                echo "${GREETING}, building the application..."
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
            }
        }
    }
}
```

**ASK** <br>
Where have we already used this exact idea — naming a value once, then referencing it by name throughout a config file? <br>
**ANSWER** <br>
Terraform `variable` blocks, and bash script variables from the Linux and Automation session — same underlying idea, different syntax each time.

We can also make a pipeline accept input at build time:

**Jenkinsfile**
```groovy
pipeline {
    agent any

    // NEW CONFIG
    parameters {
        string(name: 'ENVIRONMENT', defaultValue: 'dev', description: 'Which environment to target')
    }

    environment {
        GREETING = 'Hello'
    }

    stages {
        stage('Build') {
            steps {
                echo "${GREETING}, building for ${params.ENVIRONMENT}..."
            }
        }
    }
}
```

* **Build with Parameters** now appears on the job instead of a plain **Build Now**, letting us type in a value each time

#### The `post` section

We often want something to happen *after* all our stages run, regardless of — or specifically because of — the outcome.

**Jenkinsfile**
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

    // NEW CONFIG
    post {
        success {
            echo 'Build succeeded!'
        }
        failure {
            echo 'Build failed — someone should look at this.'
        }
        always {
            echo 'This runs no matter what happened.'
        }
    }
}
```

**ASK** <br>
In a real team setup, what might we want to happen in the `failure` block, instead of just an `echo`? <br>
**ANSWER** <br>
Sending a Slack message or email to the team, notifying whoever's on call, or automatically opening a ticket — Jenkins has plugins for most common notification channels, wired in exactly this way.

#### Running stages in parallel

Not every stage needs to run one after another.

**Jenkinsfile**
```groovy
stages {
    // NEW CONFIG
    stage('Parallel Checks') {
        parallel {
            stage('Unit Tests') {
                steps {
                    echo 'Running unit tests...'
                }
            }
            stage('Linting') {
                steps {
                    echo 'Running linter...'
                }
            }
        }
    }
}
```

**ASK** <br>
Why might we want unit tests and linting to run in parallel rather than one after the other? <br>
**ANSWER** <br>
They're independent of each other — neither result affects whether the other should run — so running them in parallel gives faster overall feedback without changing the outcome.

#### Multibranch Pipelines

So far we've configured one Jenkins job pointing at one branch. A **Multibranch Pipeline** job instead scans an entire repository and automatically creates a sub-job for *every* branch (and often every open Pull Request) that contains a `Jenkinsfile` — deleting the job again automatically when the branch is deleted.

**ASK** <br>
Why is this particularly useful for the Pull-Request-based workflow we discussed back in the Azure Fundamentals session? <br>
**ANSWER** <br>
Every feature branch and every open PR automatically gets its own pipeline run, without anyone manually creating a Jenkins job per branch — which is exactly the mechanism a "run `terraform plan` on every PR" pattern relies on.

We won't fully configure Multibranch today, but it's worth knowing it exists — we'll return to something close to this idea, applied specifically to `terraform plan`/`apply`, in the next session.

#### Exercise
Students:
1. Convert their Freestyle job from Session 2 into a Pipeline job with at least three named stages
2. Add a `post` section with `success` and `failure` blocks
3. Add a `parameters` block, and use the parameter's value inside at least one stage

<br>

---

## Session 4: Git, Agents, and a Real Build-and-Push Pipeline
### 15:15 - 16:45

### Sequence

#### Pulling the Jenkinsfile from Git

So far our Jenkinsfile has lived pasted directly into the Jenkins job configuration — which has exactly the same "not really code" problem as a Freestyle job. Let's fix that.

* **Configure** our pipeline job > under **Pipeline**, change **Definition** to `Pipeline script from SCM`
* **SCM**: `Git`
* **Repository URL**: our fork of the starter repo
* **Branch**: `*/main`
* **Script Path**: `Jenkinsfile`

For a private repository (or to avoid GitHub's rate limits on a public one), we'll want Jenkins to authenticate properly, using its **Credentials** store rather than anything typed directly into a config screen.

* On GitHub: **Settings > Developer settings > Personal access tokens** — generate a token with `repo` scope
* In Jenkins: **Manage Jenkins > Credentials > (global) > Add Credentials**
  * **Kind**: Username with password
  * **Username**: your GitHub username
  * **Password**: the token
  * **ID**: `github-credentials`
* Select these credentials on the Pipeline job's **Repository URL** configuration

**ASK** <br>
Why does this matter — the token could have just gone straight into the Repository URL field as plain text, after all? <br>
**ANSWER** <br>
Jenkins' Credentials store keeps the secret value out of both the job configuration *and* any `Jenkinsfile` in Git, masks it automatically wherever it might otherwise appear in build logs, and means the secret can be rotated centrally without editing anything else.

#### Triggering builds with a webhook

* On GitHub: **Settings > Webhooks > Add webhook**
  * **Payload URL**: `http://<your-jenkins-url>:8080/github-webhook/`
  * **Content type**: `application/json`
  * **Which events**: Just the push event
* Back in Jenkins: **Configure** our job > **Build Triggers** > **GitHub hook trigger for GITScm polling**

**NOTE FOR TRAINERS** <br>
If Jenkins is running locally, GitHub can't reach `localhost` directly — a tunnelling tool like [ngrok](https://ngrok.com/) is the standard training-room workaround. Falling back to **Poll SCM** with a short interval is an acceptable substitute if webhook setup is eating into time. <br>
**END OF NOTE**

Push a trivial change to the repo and confirm a build kicks off automatically, without touching the Jenkins UI at all.

#### Using build agents

We mentioned this morning that Jenkins can offload work to separate **agents**. The most common modern pattern is a **Docker agent** — Jenkins spins up a fresh, disposable container for each stage (or the whole pipeline), runs the steps inside it, then throws it away.

**Jenkinsfile**
```groovy
// NEW CONFIG
pipeline {
    agent {
        docker {
            image 'node:20'
        }
    }

    stages {
        stage('Check Node version') {
            steps {
                sh 'node --version'
            }
        }
    }
}
```

**ASK** <br>
Why might we want each build to run inside a disposable container, rather than directly on the Jenkins controller machine? <br>
**ANSWER** <br>
The exact tool versions and dependencies a build needs are defined by the `image` itself, so different projects can use completely different toolchains without conflicting; the environment is guaranteed clean and identical every single run; and nothing a build does can leave stray files or processes behind on the Jenkins host itself.

#### Building and pushing a real Docker image

Let's put everything from today together into one pipeline: checking out code, running tests, building a Docker image, and pushing it to Docker Hub — the same shape of pipeline we'll adapt next session to run Terraform instead.

First, store our Docker Hub credentials properly:

* **Manage Jenkins > Credentials > Add Credentials**
  * **Kind**: Username with password
  * **Username**: your Docker Hub username
  * **Password**: a Docker Hub access token (not your account password — generate one under Docker Hub's Account Settings > Security)
  * **ID**: `dockerhub-credentials`

**Jenkinsfile**
```groovy
// NEW CONFIG
pipeline {
    agent any

    environment {
        IMAGE_NAME = "your-dockerhub-username/hello-world-rest-api"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Push to Docker Hub') {
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
            echo "Image pushed: ${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo 'Something went wrong — check the console output.'
        }
    }
}
```

* `withCredentials` — a different pattern to the `environment { }` block we used for our GitHub token; it injects credentials only for the duration of the enclosed steps, and is the standard way to bind credentials that need custom variable names
* `$BUILD_NUMBER` — reused from Session 2, now used to give every image a unique, traceable tag

Push this `Jenkinsfile` and confirm the whole thing runs end to end: our webhook fires, Jenkins checks out the code, builds the image, and pushes it — visible afterwards in our Docker Hub account.

#### Where this is heading

**ASK** <br>
Looking at the shape of that pipeline — checkout, then a stage that changes something real, then a stage that ships the result somewhere — what would need to change if, instead of building and pushing a Docker image, we wanted this same pipeline to run Terraform against Azure? <br>
**ANSWER** <br>
Genuinely very little structurally: the credentials pattern is identical (Service Principal values instead of a Docker Hub token), the `sh` steps just run `terraform init/plan/apply` instead of `docker build/push`, and the same `post` block still tells us whether it worked. That's exactly the session we're moving into next.

#### Exercise
Students:
1. Build the full checkout → test → Docker build → Docker push pipeline against their own fork
2. Confirm the image appears in their own Docker Hub account after a successful run
3. **Stretch**: add a stage that runs *before* the Docker build which fails deliberately (e.g. `exit 1`), and confirm the pipeline stops before anything gets pushed — the same "don't let a broken stage reach something real" principle we'll rely on again next session with `terraform plan`

<br>

---

## Wrap-up & Q&A
### 16:45 - 17:00

- **Recap** the day's arc: CI/CD concepts → installing Jenkins → Freestyle jobs → Pipelines-as-code → connecting Git and agents → a full build-and-push pipeline
- **Reinforce** that every mechanism we'll need next session already exists in what we built today — credentials, stages, webhooks, `post` blocks — only the specific commands inside the stages will change
- **Preview** the next session directly: we'll take this exact pipeline shape and, instead of building and pushing a Docker image, use it to run Terraform against Azure — including a manual approval gate before anything gets applied
- **Direct** students to the exercises for further practice, and to the [Jenkins Pipeline documentation](https://www.jenkins.io/doc/book/pipeline/) for anyone wanting to go deeper before next time

---

[Back](./README.md)

---

