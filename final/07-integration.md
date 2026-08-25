# Session 7 — Integrating the DevOps Toolchain: Git, Jenkins, Terraform, Azure & Kubernetes — Trainer Script

The capstone day. Everything covered separately across this course — the shell, Docker, version control, CI/CD orchestration, Infrastructure as Code, the cloud, and container orchestration — assembled into one real, working, automated pipeline. This is written as a script — read it out, adapt where it feels natural, but the intent is that you could deliver this cold from the page.

---

## 📦 STARTER CODE — put this in the repo before training

Everything here goes into **`/devops-toolchain-integration/starter-code`** before the session.

**Today is the one day where shipping generously is correct.** There is no new technology to learn — every component is something students have already built from scratch. **The learning is the wiring**, and re-typing a Dockerfile they wrote in Session 3 or a `deployment.yaml` they wrote in Session 6 buys nothing while costing twenty minutes each.

What is *not* shipped: the `Jenkinsfile`. That's the join between everything, and building it stage by stage is the entire session.

<br>

**`README.md`**
```markdown
# DevOps Toolchain Integration — Starter Code

## What's here

- **jenkins-image/Dockerfile** — a Jenkins image with Terraform,
  the Azure CLI, kubectl AND the Docker CLI inside it.
  You built a simpler version of this in Session 3.

- **app/** — a small Node app with a passing test and a Dockerfile.
  Same shape as Session 3's app.

- **k8s/deployment.yaml** — the manifest you cleaned in Session 6,
  with the image tag replaced by a placeholder the pipeline fills in.

- **infra/** — a starting main.tf. YOU add the AKS cluster to it.

- **integration-notes.md** — collect every credential and value
  from Sessions 1-6 here. You will need all of them today.

- **completed-code/Jenkinsfile** — CATCH-UP ONLY.
  Building this stage by stage IS the session.

## Before we start

    docker --version
    az account show
    git --version
    export | grep ARM
```

<br>

**`jenkins-image/Dockerfile`**
```dockerfile
# A Jenkins image that can run everything today's pipeline needs.
# The stock jenkins/jenkins:lts image has NONE of these tools.

FROM jenkins/jenkins:lts-jdk17

USER root

# Base utilities plus the Docker CLI
RUN apt-get update && apt-get install -y \
      curl \
      unzip \
      gnupg \
      lsb-release \
      docker.io \
    && rm -rf /var/lib/apt/lists/*

# Terraform, from HashiCorp's own apt repository
RUN curl -fsSL https://apt.releases.hashicorp.com/gpg \
      | gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" \
      > /etc/apt/sources.list.d/hashicorp.list \
    && apt-get update && apt-get install -y terraform \
    && rm -rf /var/lib/apt/lists/*

# Azure CLI
RUN curl -sL https://aka.ms/InstallAzureCLIDeb | bash

# kubectl AND kubelogin, via the Azure CLI
RUN az aks install-cli

USER jenkins
```

<br>

**`app/package.json`**
```json
{
  "name": "toolchain-app",
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
const PORT = process.env.PORT || 8080;

app.get("/", (req, res) => {
  res.json({
    message: "Hello from the full toolchain",
    version: process.env.APP_VERSION || "dev"
  });
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

**`app/test.js`**
```javascript
// Deliberately framework-free, so npm test works with nothing
// extra installed and nothing to go wrong in the pipeline.

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

check("app is exported", () => assert.ok(app));
check("app has routes registered", () => assert.ok(app._router));

if (failures > 0) {
  console.log(`\n${failures} test(s) failed`);
  process.exit(1);        // NON-ZERO — the pipeline will fail
}

console.log("\nAll tests passed");
process.exit(0);          // ZERO — success
```

<br>

**`app/Dockerfile`**
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 8080
CMD ["npm", "start"]
```

<br>

**`app/.dockerignore`**
```
node_modules
.git
```

<br>

**`k8s/deployment.yaml`** *(the manifest from Session 6, with a placeholder the pipeline substitutes)*
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: toolchain-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: toolchain-app
  template:
    metadata:
      labels:
        app: toolchain-app
    spec:
      containers:
        - name: toolchain-app
          # The pipeline replaces this line at deploy time.
          image: IMAGE_PLACEHOLDER
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: toolchain-app
spec:
  type: LoadBalancer
  selector:
    app: toolchain-app
  ports:
    - port: 80
      targetPort: 8080
```

<br>

**`infra/main.tf`** *(a starting point — students add the AKS cluster themselves)*
```tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }

  # YOU fill this in from your Session 5 backend details.
  # Without it, this pipeline CANNOT work.
  backend "azurerm" {
    resource_group_name  = "CHANGEME"
    storage_account_name = "CHANGEME"
    container_name       = "tfstate"
    key                  = "dev/toolchain/pipeline.tfstate"
  }
}

provider "azurerm" {
  features {}
}

# TODO: add an azurerm_resource_group
# TODO: add an azurerm_kubernetes_cluster
# TODO: add outputs for cluster_name and cluster_resource_group
#       (the deploy stage reads these)
```

<br>

**`integration-notes.md`** *(students fill this in first thing — it collects six sessions of values)*
```markdown
# Integration Notes — everything I need today

## From Session 1 (Azure)

| What | Value |
|---|---|
| Subscription ID | |
| Tenant ID | |
| Tenant domain | |

## Service Principal (Session 4)

| Returned by az | Env var | Value |
|---|---|---|
| appId | `ARM_CLIENT_ID` | |
| password | `ARM_CLIENT_SECRET` | **never commit** |
| tenant | `ARM_TENANT_ID` | |
| (subscription) | `ARM_SUBSCRIPTION_ID` | |

## Remote backend (Session 5)

| What | Value |
|---|---|
| Backend resource group | |
| Backend storage account | |
| Container | `tfstate` |
| My state key | `dev/toolchain/pipeline.tfstate` |

## Docker Hub (Session 3)

| What | Value |
|---|---|
| Username | |
| Access token | **never commit** |
| Image name | `<username>/toolchain-app` |

## Jenkins credential IDs I create today

| ID | Kind | Holds |
|---|---|---|
| `azure-client-id` | Secret text | |
| `azure-client-secret` | Secret text | |
| `azure-subscription-id` | Secret text | |
| `azure-tenant-id` | Secret text | |
| `github-credentials` | Username/password | |
| `dockerhub-credentials` | Username/password | |
```

<br>

---

## 💬 SLACK SNIPPETS — queue these before you start

**Post reactively.** Today has the highest paste-to-type ratio of the course, and that's correct — the value is in understanding the joins, not in transcribing Groovy.

| # | When | What to post |
|---|---|---|
| 1 | 09:05 | Pre-flight check |
| 2 | 10:20 | `docker build` + `docker run` for the Jenkins image |
| 3 | 10:45 | **The four-tool verification** — nobody proceeds until this passes |
| 4 | 11:20 | `az ad sp create-for-rbac` + the six credential IDs |
| 5 | 11:40 | **The first `Jenkinsfile`** — checkout, init, validate, plan |
| 6 | 12:20 | The apply stage with the `input` gate |
| 7 | 14:10 | The `when { branch 'main' }` block |
| 8 | 15:20 | **The AKS `aks.tf`** |
| 9 | 15:45 | **The Deploy to Kubernetes stage** — the hardest block of the day |
| 10 | 16:00 | The complete `Jenkinsfile` |
| 11 | 16:45 | **The teardown sequence** |

Each is marked **💬 SLACK** in the body below.

---

## Organisation

### Audience

Trainees who have completed the full course: **Azure fundamentals**, **bash scripting & automation**, **Docker**, **Jenkins & CI/CD**, **Terraform Parts 1 & 2**, and **Kubernetes & AKS**.

They are experienced developers: **Node, Express, REST, MVC, SQL, Git, GitHub, Docker, frontend, unit and integration testing**.

**Nothing today is a genuinely new tool.** Every piece is something they've used in isolation. **The learning is the wiring** — and the specific problems that only appear once tools have to talk to each other unattended.

**NOTE FOR TRAINERS** <br>
This room will be tempted to feel today is easy, because they recognise every component. Set expectations in the kickoff: **integration is where the interesting failures live.** A set of tools that each work perfectly alone will still fail the moment they have to cooperate with no human present — and today's four "integration problems" are exactly the ones that catch professionals. <br>
The other risk is the opposite: someone who's shaky on one earlier session gets stuck and can't tell which layer is broken. Push them hard on **naming the layer** before debugging — container, credential, network, or cloud. <br>
**END OF NOTE**

### How this document is laid out

Today involves more context-switching than any other session. Every instruction block is labelled:

- *(Run from `~/toolchain-day/...`)* — a **local terminal** command
- *(In the Jenkins UI — Dashboard)* — a **browser** action in Jenkins
- *(In the Azure Portal — Resource groups)* — a **browser** action in Azure
- *(On GitHub — your fork)* — a **browser** action on GitHub

The folders we use today:

| Folder | What it's for |
|---|---|
| `~/toolchain-day` | Parent folder; the custom Jenkins image is built here |
| `~/toolchain-day/<your-fork>` | Your cloned fork — the `Jenkinsfile`, `infra/`, `app/` and `k8s/` |

**Every activity has a `**Solution**` block** immediately afterwards.

### Duration & schedule

Full day, **09:00–17:00**, with a one-hour lunch and two 15-minute breaks.

**Hands-on time today: ~3 hours 40 minutes** across eight activities, every one with a solution in this document.

| Time | Section | Activity time |
|---|---|---|
| 09:00–09:15 | Welcome: the whole course in one diagram | |
| 09:15–10:00 | The toolchain — why no single tool does everything | |
| **10:00–10:15** | **Break** | |
| 10:15–11:15 | A Jenkins that can actually do the job | **30 min** |
| 11:15–12:15 | Authenticating to Azure & the first Terraform stages | **35 min** |
| 12:15–13:00 | The apply gate: saved plans and human approval | **15 min + challenge** |
| **13:00–14:00** | **Lunch** | |
| 14:00–14:45 | The Pull Request workflow: plan on PR, apply on merge | **15 min** |
| 14:45–15:00 | Failure modes: what breaks and what it costs | **15 min research** |
| **15:00–15:15** | **Break** | |
| 15:15–16:45 | Capstone: the complete pipeline, code to running application | **90 min** |
| 16:45–17:00 | Wrap-up, tear everything down, & course close | **10 min** |

### Set-up

#### For Trainers

**For the slidee presentation** <br>
**Open** the [Slidee](https://github.com/mklilley/slidee) presentation:

- Run `npx slidee` from within the `curriculum` folder to see the available slide decks (those with extension `.slides.md`)
- Click on `See your presentations`
- Open `Weeks X Y > CDO > DevOps Toolchain Integration`
- Hit `s` to open the presenter notes
- Resource to be kept open on a tab throughout the session to refer to

**Check out** [FAQs](/FAQs/README.md) for help with using Slidee

#### For Students
- **Direct** students to the starter code for this module
  - [Student Facing Repo](https://github.com/LaFosseAcademy/cloud-devops-student-repos/tree/main)
- **Use**
  - **/devops-toolchain-integration/starter-code**
- **Make sure**, before the session starts, every student has:
  - **Docker Desktop** installed and running
  - A **GitHub account** with the starter repo **forked and cloned**
  - A **Docker Hub** account with an access token
  - An **Azure subscription** with remaining credit
  - Their **Service Principal** credentials from Session 4
  - Their **remote backend storage account** from Session 5, **still existing**

**NOTE FOR TRAINERS — the four things that will eat your day** <br>
**(1) Everything at once.** Highest moving-parts day of the course. If a student is missing Docker, a Service Principal, a backend, or their fork — sort it in the first fifteen minutes, not at 15:15. <br>
**(2) The Jenkins image.** The stock image has **no** Terraform, Azure CLI, kubectl or Docker CLI. Every "command not found" today traces here. **Nobody proceeds past 10:45 without the four-tool verification passing.** <br>
**(3) The backend may be gone.** Session 5 ended with `terraform destroy` on everything — **including, for some students, the backend storage account.** Check this early; rebuilding it takes five minutes but not if it's discovered at 11:50. <br>
**(4) Cost.** By the end students may have an AKS cluster, VMs, a load balancer and a registry running. The 16:45 teardown is not optional — build in the full fifteen minutes. <br>
**END OF NOTE**

## Learning objectives

- **Explain** where each tool sits in a DevOps toolchain, and why no single tool covers the whole loop
- **Build** a Jenkins environment capable of running Terraform, the Azure CLI, kubectl and Docker
- **Authenticate** Jenkins to Azure and Docker Hub without hardcoding any secret
- **Write** a multi-stage `Jenkinsfile` running `terraform init`, `plan` and `apply`
- **Explain** why remote state is a hard requirement for pipeline-run Terraform
- **Implement** a human approval gate, and explain when to remove it
- **Design** a PR-based workflow: plan on pull request, apply on merge
- **Assemble** a complete pipeline: code change → build image → push → provision infrastructure → deploy to Kubernetes
- **Reason** about failure modes at each stage and what each one costs

<br>

## Sequence

### 09:00–09:15 — Welcome: The Whole Course in One Diagram

Morning. Today is different from every other session on this course, in one specific way.

**There is almost no new technology today.** No new tool to install and learn from scratch. Every component, you have already used on its own:

- You can write **bash** and you understand exit codes
- You can build a **Docker** image and push it to a registry
- You can build a **Jenkins** pipeline that tests, builds and pushes
- You can write **Terraform** that provisions real Azure infrastructure
- You understand **remote state**, and why it matters
- You can deploy to **Kubernetes** with declarative YAML

Today is about the **wiring**. And wiring is where the interesting problems live, because a set of tools that each work perfectly alone will still fail the moment they have to cooperate without a human present.

Here's what we're building:

```
engineer edits code and/or .tf files
        |
        v
git push
        |
        v
Jenkins pipeline triggered automatically
        |
        v
build & test  ->  docker build & push  ->  terraform plan
        |
        v
human approval  (or: the PR review WAS the approval)
        |
        v
terraform apply  ->  kubectl apply
        |
        v
running application on infrastructure that was
never touched by hand
```

**Every arrow in that diagram is something you'll build today.**

**ASK** <br>
Across every session on this course, one question has kept coming back in different clothing. What is it? <br>
**ANSWER** <br>
***Who actually typed that command?*** Every time — `terraform apply`, `docker push`, `kubectl apply` — the answer has been "a human, at a laptop". **Today the answer becomes "nobody, it just happened, and there's a record of exactly why."**

Lunch at 1 for an hour, breaks mid-morning and mid-afternoon. **Hard rule: we tear everything down at 16:45.** By the end of today you'll have real infrastructure running.

**First thing — everyone check their prerequisites.**

**💬 SLACK — snippet 1**, post at 09:05:
```bash
mkdir -p ~/toolchain-day && cd ~/toolchain-day

docker --version              # Docker running?
az account show               # signed in?
git --version
export | grep ARM             # Service Principal creds present?

# CRITICAL — does your Session 5 backend still exist?
az storage account list -o table
```

That last one matters. **Session 5 ended with `terraform destroy`, and some of you will have destroyed your backend storage account too.** If it's gone, tell me now — we rebuild it in five minutes, but not if you discover it at midday.

<br>
<br>

### 09:15–10:00 — The Toolchain: Why No Single Tool Does Everything

#### The gap we're closing

**ASK** <br>
Every time we've run `terraform apply` on this course, who typed it? <br>
**ANSWER** <br>
Us — a human, sat at a laptop.

That's the gap today closes. We first drew this picture in **Session 1**, on day one: git push, a PR reviewed alongside a `terraform plan`, then merge triggers an automatic apply. **It was a diagram then. Today it becomes a thing that runs.**

*REFER TO RESOURCE 1 - SLIDEE* <br>
![toolchain-1](./resources/toolchain-1.png)

#### Where each tool sits

Let's be precise about what each piece does, because the boundaries matter.

| Tool | Its job | The question it answers |
|---|---|---|
| **Git** | Version control and the trigger | *What changed, who changed it, and why?* |
| **Docker** | Packaging | *How do I run this identically everywhere?* |
| **Jenkins** | Orchestration | *What should happen, in what order, when something changes?* |
| **Terraform** | Infrastructure as Code | *What cloud resources should exist?* |
| **Azure** | The cloud | *Where does it all actually run?* |
| **Kubernetes** | Container orchestration | *How do I keep applications running and update them safely?* |

**ASK** <br>
Terraform can run scripts with `remote-exec`, and Kubernetes can restart failed containers. Why not pick one tool and push it as far as it goes? <br>
**ANSWER** <br>
Because each is excellent at what it was built for and **mediocre-to-dangerous outside it** — and you saw this directly. Terraform's `remote-exec` "worked", but only ran at creation time and couldn't be re-run, which is why HashiCorp themselves call provisioners a last resort. Kubernetes keeps containers alive but has no idea how to create the VNet its cluster sits in. **Trying to make one tool do everything gets you the worst version of each job.**

**ASK** <br>
That argument should sound familiar. Where else on this course did you meet it? <br>
**ANSWER** <br>
**Kubernetes' Single Responsibility split** — Pod, ReplicaSet, Deployment, Service, each with one job. And before that, **MVC** in your own applications. It's the same principle at three different scales: **separate things that change for different reasons.** A toolchain is just SRP applied to your tooling, and the boundaries between tools are the module boundaries.

#### Where Jenkins fits

**Jenkins** is the glue. Its entire job: *watch for an event, then run some tasks in a defined order, and stop if any of them fail.*

It doesn't know what Terraform is. It doesn't know what Kubernetes is. **It runs shell commands and checks exit codes.**

**ASK** <br>
Jenkins genuinely just runs shell commands and checks exit codes. Why did we spend a whole session on exit codes back in Session 2? <br>
**ANSWER** <br>
Because exit codes are the **entire contract** between your scripts and Jenkins. A stage passes if the last command exits `0` and fails otherwise — that's it. A script that swallows an error and exits `0` anyway will make a broken pipeline look green and **ship broken infrastructure**. Everything Jenkins does about safety rests on commands being **honest about failure**, which is exactly why we hammered it in week one.

**ASK** <br>
Jenkins isn't the only tool that does this. What else might you use? <br>
**ANSWER** <br>
GitHub Actions, GitLab CI, Azure Pipelines, CircleCI. Jenkins is one of the oldest and most widely adopted, highly extensible via plugins, and — importantly for learning — **self-hosted**, so every piece is something we set up and can inspect rather than a black box.

**ASK** <br>
What's the trade-off of self-hosting Jenkins versus a SaaS tool? <br>
**ANSWER** <br>
Full control, no vendor lock-in, and nothing running outside infrastructure we manage — but **we're on the hook for patching, scaling and maintaining the Jenkins server itself.** There's also a subtler cost: **a self-hosted Jenkins holding credentials for your whole cloud estate is an extremely attractive target**, and securing it properly is real work. Today we're giving ours subscription-wide `Contributor` — worth noticing what that means if someone got into it.

#### The problem you haven't hit yet

Here's something about to matter enormously.

**ASK** <br>
Your Terraform state has been on your laptop all course. A Jenkins build runs inside a fresh container that's thrown away afterwards. What happens when that pipeline runs `terraform apply`? <br>
**ANSWER** <br>
**Disaster, if state is local.** The build starts with **no state file at all**, so Terraform believes nothing exists and plans to create everything from scratch — duplicating resources or erroring on name collisions. Then the container is destroyed and whatever state it wrote vanishes, so the **next** build starts from nothing again. <br>
**Remote state isn't a nice-to-have for pipeline Terraform, it's a hard prerequisite** — which is exactly why we built that backend storage account in Session 5.

**That's the first genuine integration problem of the day.** Two tools that each worked fine alone are **incompatible until you change something**. There will be three more, and I'll name each as we hit it. Watch for them — they're the actual content of today.

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>
### 10:15–11:15 — A Jenkins That Can Actually Do the Job
*(Activity: 30 min)*

You built a Jenkins in Session 3. Today it needs to do considerably more.

**ASK** <br>
Our pipeline needs to run `terraform`, `az`, `kubectl` and `docker`. The official `jenkins/jenkins:lts` image contains exactly none of them. What happens when a stage calls `terraform init`? <br>
**ANSWER** <br>
`terraform: not found`, a non-zero exit code, and a red stage. Which is **correct behaviour** — the pipeline failed honestly. But it means our first job is building a Jenkins image that has the tools it needs. <br>
**That's integration problem number two:** the orchestrator needs every tool it orchestrates installed inside it. Obvious in hindsight, invisible until it bites.

#### Build a capable Jenkins image

The Dockerfile is in your starter repo at `jenkins-image/Dockerfile` — **no need to type it**. Read it through:

```dockerfile
FROM jenkins/jenkins:lts-jdk17

USER root

RUN apt-get update && apt-get install -y \
      curl unzip gnupg lsb-release docker.io \
    && rm -rf /var/lib/apt/lists/*

RUN curl -fsSL https://apt.releases.hashicorp.com/gpg \
      | gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" \
      > /etc/apt/sources.list.d/hashicorp.list \
    && apt-get update && apt-get install -y terraform \
    && rm -rf /var/lib/apt/lists/*

RUN curl -sL https://aka.ms/InstallAzureCLIDeb | bash

RUN az aks install-cli

USER jenkins
```

Every line is Docker knowledge you already have:

- **`FROM jenkins/jenkins:lts-jdk17`** — official Jenkins. `lts` is Long Term Support
- **`USER root`** — installing packages needs admin rights
- **First `RUN`** — base tools plus `docker.io`, giving us the Docker **client**. Note `&&` chaining (only continue if the previous step succeeded — Session 2) and `rm -rf /var/lib/apt/lists/*` to discard the package index
- **The Terraform block** — adds HashiCorp's apt repository. The `gpg --dearmor` step imports their signing key so apt can verify the packages are genuinely from HashiCorp. `$(lsb_release -cs)` is command substitution, printing the Debian release codename
- **Azure CLI** — Microsoft's install script
- **`az aks install-cli`** — installs `kubectl` **and** `kubelogin`, which we flagged in Session 6
- **`USER jenkins`** — drop back to the unprivileged user

**ASK** <br>
Each `RUN` creates a separate image layer. Why chain commands with `&&` inside one `RUN` rather than writing six separate `RUN` lines? <br>
**ANSWER** <br>
Because **each `RUN` is a layer, and layers are additive** — deleting a file in a later layer doesn't reclaim the space from an earlier one. If you `apt-get update` in one layer and `rm -rf /var/lib/apt/lists/*` in another, the package index is **still in the image**, just hidden. Chaining means download and cleanup happen in the same layer, so the cleanup actually shrinks the result. Same reason you don't `COPY` your `node_modules` and then delete it.

Build it:

*(Run from `~/toolchain-day/<your-fork>/jenkins-image`)*
```bash
docker build -t jenkins-devops .
```

Takes a few minutes the first time.

#### Run it

*(Run from `~/toolchain-day`)* — Mac Terminal, or **Git Bash** on Windows so the `\` line continuations work:
```bash
docker run -d --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -u root \
  jenkins-devops
```

Single-line version for any shell:

*(Run from `~/toolchain-day`)*
```bash
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -u root jenkins-devops
```

Every flag is revision from Session 3:
- `-d` — detached, runs in the background
- `-p 8080:8080` — the Jenkins web UI
- `-p 50000:50000` — agent communication
- `-v jenkins_home:/var/jenkins_home` — a **named volume** so config, jobs and credentials survive the container being recreated
- `-v /var/run/docker.sock:/var/run/docker.sock` — mounts the host's Docker socket so pipelines can build images
- `-u root` — sidesteps Docker permission issues. **A training shortcut, not production practice**

Check it started:

*(Run from `~/toolchain-day`)*
```bash
docker ps
```

#### Unlock and configure

*(Run from `~/toolchain-day`)*
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

*(In your browser — `http://localhost:8080`)*
- Paste the password
- **Install suggested plugins** — Git, Pipeline and Credentials
- Create your **admin user** and write the details down
- Accept the default Jenkins URL

#### Verify the tools are actually there

**This is the step that saves you an hour later. Nobody proceeds until all four respond.**

*(Run from `~/toolchain-day`)*
```bash
docker exec jenkins terraform version
docker exec jenkins az version
docker exec jenkins kubectl version --client
docker exec jenkins docker --version
```

**HANDS ON (30 min)** <br>

Part A *(20 min)* — build and run.
1. *(Run from `~/toolchain-day/<your-fork>/jenkins-image`)* Build the image as `jenkins-devops`
2. *(Run from `~/toolchain-day`)* Run the container with all the flags shown. Confirm with `docker ps`
3. Get the unlock password, complete setup in the browser, install suggested plugins, create your admin user

Part B *(10 min)* — verify and record.
4. **Verify all four CLIs respond inside the container.** If any doesn't, fix it now
5. Run `docker exec jenkins which terraform az kubectl docker` and note the paths. Which folders from Session 2 are they in?
6. Fill in the top three sections of `integration-notes.md` from your previous sessions' notes
**END OF NOTE**

**💬 SLACK — snippet 2**:
```bash
# 1. Build (from the starter repo's jenkins-image folder)
docker build -t jenkins-devops .

# 2. Run (single line — works in any shell)
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -u root jenkins-devops

# 3. Confirm
docker ps

# 4. Unlock password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# 5. Browser: http://localhost:8080
```

**💬 SLACK — snippet 3**, the verification. **Post this and don't let anyone move on until it passes:**
```bash
# ALL FOUR must respond. Any failure = the image didn't build properly.
docker exec jenkins terraform version
docker exec jenkins az version
docker exec jenkins kubectl version --client
docker exec jenkins docker --version
```

**Solution**

*(Run from `~/toolchain-day/<your-fork>/jenkins-image`)*
```bash
docker build -t jenkins-devops .
```

*(Run from `~/toolchain-day`)*
```bash
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -u root jenkins-devops
docker ps
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

docker exec jenkins terraform version
docker exec jenkins az version
docker exec jenkins kubectl version --client
docker exec jenkins docker --version
```

**Step 5 answer** — a nice callback to Session 2:
```
/usr/bin/terraform      # installed by apt -> /usr/bin
/usr/bin/az             # ...though az is often a wrapper script
/usr/local/bin/kubectl  # installed manually by az aks install-cli
/usr/bin/docker         # installed by apt
```

**Notice the pattern**: things installed by the **package manager** land in `/usr/bin`; things installed **manually by a script** land in `/usr/local/bin`. That's exactly the convention from Session 2, holding true inside a container built by someone else.

**Troubleshooting to have ready:**

*(Run from `~/toolchain-day`)*
```bash
docker logs jenkins              # why won't it start
docker exec -it jenkins bash     # get a shell inside to poke around
docker rm -f jenkins             # remove the container — the VOLUME survives
docker volume ls                 # confirm jenkins_home still exists
```

If the build fails at the Terraform step, it's usually a transient network issue fetching HashiCorp's key — **re-run the build**; Docker's layer cache means it resumes rather than starting over.

<br>
<br>

### 11:15–12:15 — Authenticating to Azure & the First Terraform Stages
*(Activity: 35 min)*

#### The Service Principal

Jenkins needs to authenticate to Azure — and as with everything automated on this course, that means a **Service Principal**, not a human login.

If you don't already have one:

*(Run from `~/toolchain-day`)*
```bash
az account show --query id -o tsv
az ad sp create-for-rbac --name "jenkins-terraform" --role="Contributor" --scopes="/subscriptions/<your-subscription-id>"
```

Returns `appId`, `password` and `tenant` — exactly as in Sessions 1 and 4. Copy them into `integration-notes.md`; the password is shown once.

**ASK** <br>
Back in Session 1 we asked why automation should have its own identity. Now a *machine* is genuinely about to use it — which of those reasons matters most? <br>
**ANSWER** <br>
All of them, but two become concrete. It **works headlessly** — there's no browser for an interactive login, and no human present at 3am. And it can be **revoked independently** — if the Jenkins server is compromised you kill that one identity without touching anyone's account. <br>
There's a third that only appears now: the credential is **scoped**, so a compromised pipeline can only damage what that Service Principal was allowed to touch. Which is why `Contributor` on the whole subscription is a training shortcut you'd narrow in production — and why **`Owner` would be genuinely reckless**, since it could grant permissions to others.

#### Storing secrets in Jenkins

*(In the Jenkins UI — Dashboard)*
- **Manage Jenkins → Credentials → System → Global credentials (unrestricted) → Add Credentials**

Add **four** entries, each **Kind: Secret text**:

| ID | Value |
|---|---|
| `azure-client-id` | the `appId` |
| `azure-client-secret` | the `password` |
| `azure-subscription-id` | your subscription ID |
| `azure-tenant-id` | the `tenant` |

**ASK** <br>
Why Secret text credentials rather than writing these four into the `Jenkinsfile` as environment variables? <br>
**ANSWER** <br>
Same reasoning as every secret on this course, sharper here: a `Jenkinsfile` is **committed to Git**, and **Git history is permanent** — deleting the line later doesn't remove it. The credential store keeps values out of the repo, **automatically masks them** in console output, and lets you rotate centrally without editing code. **A cloud credential with Contributor rights committed to a repository is a genuine security incident**, not a tidiness problem.

**ASK** <br>
You've now stored secrets three different ways on this course. Where, and what's the common principle? <br>
**ANSWER** <br>
**Jenkins' credentials store** (Session 3), **`ARM_*` environment variables** for Terraform (Session 4), and **Kubernetes Secrets** (Session 6 take-home). Three different mechanisms, one principle: **secrets live outside the artifact that uses them.** The mechanism changes per tool; the rule never does. That's the kind of pattern worth recognising, because the next tool you meet will have its own mechanism and you'll know immediately what to look for.

#### Connect Jenkins to your repository

*(Run from `~/toolchain-day`)*
```bash
git clone https://github.com/<your-username>/<your-fork>.git
cd <your-fork>
ls
```

*(On GitHub — your fork)*
- **Settings → Developer settings → Personal access tokens** — generate one with `repo` scope

*(In the Jenkins UI — Manage Jenkins → Credentials → Add Credentials)*
- **Kind**: Username with password
- **Username**: your GitHub username
- **Password**: the token
- **ID**: `github-credentials`

Now the job:

*(In the Jenkins UI — Dashboard)*
- **New Item** → `terraform-pipeline` → **Pipeline** → **OK**
- **Pipeline** section:
  - **Definition**: `Pipeline script from SCM`
  - **SCM**: `Git`
  - **Repository URL**: your fork's HTTPS URL
  - **Credentials**: `github-credentials`
  - **Branch Specifier**: `*/main`
  - **Script Path**: `Jenkinsfile`
- Scroll up to **Build Triggers** → tick **Poll SCM** → **Schedule**: `H/2 * * * *`
- **Save**

**NOTE FOR TRAINERS** <br>
Webhooks are the right answer and can't work here: GitHub is on the public internet and the student's Jenkins is on `localhost`. **Poll SCM** demonstrates the identical principle with no networking magic. Teach the webhook as the goal, offer ngrok as a stretch, use polling for the hands-on. <br>
**END OF NOTE**

#### Fix the backend first

**Before writing any pipeline** — open `infra/main.tf` and fill in your backend details from Session 5.

*(Run from `~/toolchain-day/<your-fork>/infra`)*
```tf
  backend "azurerm" {
    resource_group_name  = "rg-backend-state-jbloggs-devops"
    storage_account_name = "stdevappsbackendjbloggs"
    container_name       = "tfstate"
    key                  = "dev/toolchain/pipeline.tfstate"
  }
```

**ASK** <br>
What specifically goes wrong if you push without this? <br>
**ANSWER** <br>
The build starts with an empty workspace and therefore **no state**. Terraform assumes nothing exists and plans to create everything — which either **duplicates resources** or **fails on name collisions**. Then the workspace is discarded and the next build repeats the mistake. With a remote backend the pipeline reads the same authoritative state every time and takes a **lock** while it works, so two concurrent builds can't corrupt each other.

#### The first Jenkinsfile

*(Run from `~/toolchain-day/<your-fork>`)*
- Run: `touch Jenkinsfile`
- Then: `code Jenkinsfile`

**Jenkinsfile**
```groovy
pipeline {
    agent any

    environment {
        ARM_CLIENT_ID       = credentials('azure-client-id')
        ARM_CLIENT_SECRET   = credentials('azure-client-secret')
        ARM_SUBSCRIPTION_ID = credentials('azure-subscription-id')
        ARM_TENANT_ID       = credentials('azure-tenant-id')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                dir('infra') {
                    sh 'terraform init'
                }
            }
        }

        stage('Terraform Validate') {
            steps {
                dir('infra') {
                    sh 'terraform validate'
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                dir('infra') {
                    sh 'terraform plan -out=tfplan'
                }
            }
        }
    }

    post {
        success { echo 'Plan generated successfully.' }
        failure { echo 'Pipeline failed — check which stage went red.' }
    }
}
```

Note there's **no apply yet** — deliberately. Get a plan running safely first.

The `credentials()` helper looks up a credential by ID and injects its **value** as an environment variable at run time. The secret never appears in the file.

`dir('infra')` runs the enclosed steps **inside that subfolder** — `cd infra` for a block of steps, putting you back afterwards.

**ASK** <br>
Where have you seen these exact four variable names before? <br>
**ANSWER** <br>
`ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, `ARM_SUBSCRIPTION_ID`, `ARM_TENANT_ID` — you `export`ed these **by hand in your terminal** on the first Terraform day. Terraform's `azurerm` provider looks for **precisely those names** automatically. <br>
**We're inventing nothing.** Jenkins is doing what you did with `export`, for a machine. That's the whole trick of today: **the pipeline isn't special, it's just a different thing typing the same commands.**

Push it:

*(Run from `~/toolchain-day/<your-fork>`)*
```bash
git add Jenkinsfile infra/main.tf
git commit -m "Add Jenkins pipeline for Terraform plan"
git push origin main
```

*(In the Jenkins UI — the `terraform-pipeline` job)*
- **Build Now** for the first run
- Click into the build → **Console Output**

You'll watch `terraform init` download the `azurerm` provider — the exact output you've seen dozens of times, **just on Jenkins' machine instead of yours**.

**HANDS ON (35 min)** <br>

Part A *(15 min)* — credentials.
1. Create a Service Principal (or reuse yours) and store all **four** values as **Secret text** credentials
2. Create a GitHub personal access token and store it as `github-credentials`
3. Record all six credential IDs in `integration-notes.md`

Part B *(20 min)* — the first pipeline.
4. Fill in the `backend "azurerm"` block in `infra/main.tf` from your Session 5 notes
5. Create the `terraform-pipeline` job pointing at your fork, with **Poll SCM** on `H/2 * * * *`
6. Write the `Jenkinsfile` with Checkout, Init, Validate and Plan, and push it
7. Get a **green** run through all four stages, and **read the plan output** in the console
8. Make a trivial change (add a tag to a resource), push, and **wait** for the poll to trigger a build on its own
**END OF NOTE**

**💬 SLACK — snippet 4**:
```bash
# Get your subscription ID
az account show --query id -o tsv

# Create the Service Principal (paste YOUR subscription id)
az ad sp create-for-rbac --name "jenkins-terraform" --role="Contributor" \
  --scopes "/subscriptions/<your-subscription-id>"
```
```
Jenkins > Manage Jenkins > Credentials > System > Global credentials > Add Credentials

Kind: Secret text
  ID: azure-client-id          Value: the appId
  ID: azure-client-secret      Value: the password
  ID: azure-subscription-id    Value: your subscription id
  ID: azure-tenant-id          Value: the tenant

Kind: Username with password
  ID: github-credentials       your GitHub username + a PAT with 'repo' scope
  ID: dockerhub-credentials    your Docker Hub username + access token
```

**💬 SLACK — snippet 5**, the first `Jenkinsfile` — *(contents as the block above)*.

**Solution**

The complete `Jenkinsfile` for this stage is as written above. And the backend block in `infra/main.tf`:

```tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }

  backend "azurerm" {
    resource_group_name  = "rg-backend-state-jbloggs-devops"
    storage_account_name = "stdevappsbackendjbloggs"
    container_name       = "tfstate"
    key                  = "dev/toolchain/pipeline.tfstate"
  }
}

provider "azurerm" {
  features {}
}
```

*(Run from `~/toolchain-day/<your-fork>`)*
```bash
git add Jenkinsfile infra/main.tf
git commit -m "Add Jenkins pipeline for Terraform plan"
git push origin main

# Step 8 — trigger it WITHOUT touching Jenkins
echo "  # a trivial change" >> infra/main.tf
git add infra/main.tf
git commit -m "Trigger the pipeline"
git push origin main
# then wait up to two minutes and watch the Dashboard
```

**Common failures here:**

| Symptom | Cause |
|---|---|
| `terraform: not found` | The custom image wasn't used, or wasn't built |
| Init fails: "backend not found" | Backend storage account was destroyed at the end of Session 5. Rebuild it |
| Init fails: authorisation | The Service Principal can't reach the backend's resource group. Subscription-scoped `Contributor` covers both, which is why we used it |
| `Error building AzureRM Client` | One of the four credential IDs is misspelt |
| Poll SCM never triggers | The job was saved before any manual build. Run **Build Now** once first |

<br>
<br>
### 12:15–13:00 — The Apply Gate: Saved Plans and Human Approval
*(Activity: 15 min + challenge)*

We have a pipeline that plans. Now let's let it actually change something — carefully.

**Jenkinsfile**
```groovy
        stage('Terraform Plan') {
            steps {
                dir('infra') {
                    sh 'terraform plan -out=tfplan'
                }
            }
        }

        // NEW CONFIG
        stage('Terraform Apply') {
            steps {
                input message: 'Apply this Terraform plan?', ok: 'Apply'
                dir('infra') {
                    sh 'terraform apply -auto-approve tfplan'
                }
            }
        }
```

Two things worth pausing on properly.

**`terraform plan -out=tfplan`** saves the plan to a file — you met this in Session 4. In a pipeline it stops being a curiosity and becomes the entire point: **the exact plan a human reviewed is the exact plan that gets applied.** Nothing can drift between review and execution, because we're not re-planning, we're replaying a saved decision.

**`terraform apply -auto-approve tfplan`** — `-auto-approve` skips the interactive `yes` prompt, essential because **there is no human at a terminal to type it.**

**ASK** <br>
That's the third time this course we've had to remove an interactive prompt for automation. Where were the other two? <br>
**ANSWER** <br>
**`apt-get install -y`** in your Terraform provisioners, and the **`read -p` discussion** in Session 2's stretch goals. Same lesson each time: **interactive prompts are helpful by hand and fatal in automation**, because there's nobody there and the job hangs until it times out. When you meet a new tool, "how do I make this non-interactive?" is one of the first questions worth asking.

**`input message: '...'`** pauses the pipeline and waits for a human to click **Apply** in the Jenkins UI.

**ASK** <br>
The whole point of CI/CD is removing manual steps. So why deliberately add one back, right before `apply`? <br>
**ANSWER** <br>
Because **infrastructure changes can be destructive and irreversible** in a way application deploys usually aren't. You've seen `must be replaced` in plan output — on a storage account holding data or a database, that's catastrophic. A human reading the actual plan immediately before it executes is a cheap, valuable safety net, **especially while a team is still building trust in a new pipeline**. <br>
Fully unattended apply is common and legitimate — but it should be **a deliberate decision to remove the gate**, not something that happened by default.

**ASK** <br>
When would you actually remove it? <br>
**ANSWER** <br>
**When the review has genuinely happened somewhere else** — which is the PR workflow after lunch. If a teammate already read the `plan` output on the pull request and approved the merge, gating again on merge is just friction: **you're asking someone to approve a decision that was already made.** The gate belongs at the point where a human is actually exercising judgement, and only there.

Push and run it:

*(Run from `~/toolchain-day/<your-fork>`)*
```bash
git add Jenkinsfile
git commit -m "Add apply stage with approval gate"
git push origin main
```

*(In the Jenkins UI — the running build)*
- Watch it reach **Terraform Apply** and pause
- **Read the plan output** in the Console Output above
- Click **Apply** when satisfied

*(In the Azure Portal — Resource groups)* — confirm the resource exists.

**Nobody ran `terraform apply`.** Sit with that — it's what this whole course has been building towards.

**HANDS ON (15 min)** <br>
*(Run from `~/toolchain-day/<your-fork>`)*
1. Add the **Terraform Apply** stage with the `input` gate and push
2. Watch the build pause, read the plan, and approve it
3. *(In the Azure Portal)* Verify the resource was created
4. Make a change to your Terraform (add a tag), push, and watch the pipeline produce a **different** plan — an update rather than a create
5. **Deliberately break your Terraform** (misspell an attribute), push, and confirm the pipeline **stops at Validate** and never reaches Apply
**END OF NOTE**

**💬 SLACK — snippet 6**:
```groovy
        stage('Terraform Apply') {
            steps {
                input message: 'Apply this Terraform plan?', ok: 'Apply'
                dir('infra') {
                    sh 'terraform apply -auto-approve tfplan'
                }
            }
        }
```

**Solution**

*(Run from `~/toolchain-day/<your-fork>`)*
```bash
git add Jenkinsfile && git commit -m "Add apply stage with approval gate" && git push origin main

# Step 4 — a change producing an UPDATE plan, not a create
# add a tags block to your resource group, then:
git add . && git commit -m "Add environment tag" && git push origin main

# Step 5 — deliberately break it
# change 'location' to 'locatoin' in a resource, then:
git add . && git commit -m "Break it on purpose" && git push origin main
```

**Step 5 answer:** the pipeline goes **Checkout ✅ → Init ✅ → Validate ❌**, and **Plan and Apply never run.** `terraform validate` exits non-zero, Jenkins reads the non-zero exit code, and stops the line. **Nothing in Azure was touched.** Fix the typo and push again to go green.

That's the exit-code contract doing its job — and it's why Validate earns its place as a separate stage before Plan: **it's the cheapest check, so it should fail first.**

---

**Challenge**

*Direct* students, **in pairs**, to harden the pipeline so that:

* The **plan output is saved as an artifact**, downloadable from the build page, so there's a permanent record of what was proposed
* The approval gate **times out after 15 minutes** rather than waiting forever
* The `post` section reports the **build number and job name** on both success and failure
* A `terraform fmt -check` stage runs **before** Validate, failing the build if the code isn't formatted
* **OPTIONAL** — the pipeline records **who approved** the apply, and echoes that name in the success message

*Provide* this hint: Jenkins has an `archiveArtifacts` step, and `timeout` can wrap other steps. `terraform show` can turn a saved plan file back into readable text.

*Grant* ~10 minutes.

**SOLUTION**

```groovy
pipeline {
    agent any

    environment {
        ARM_CLIENT_ID       = credentials('azure-client-id')
        ARM_CLIENT_SECRET   = credentials('azure-client-secret')
        ARM_SUBSCRIPTION_ID = credentials('azure-subscription-id')
        ARM_TENANT_ID       = credentials('azure-tenant-id')
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Terraform Init') {
            steps { dir('infra') { sh 'terraform init' } }
        }

        // Cheapest check first — fail fast on formatting
        stage('Terraform Format') {
            steps { dir('infra') { sh 'terraform fmt -check -recursive' } }
        }

        stage('Terraform Validate') {
            steps { dir('infra') { sh 'terraform validate' } }
        }

        stage('Terraform Plan') {
            steps {
                dir('infra') {
                    sh 'terraform plan -out=tfplan'
                    // Turn the BINARY plan into readable text, then keep it
                    sh 'terraform show -no-color tfplan > tfplan.txt'
                }
                archiveArtifacts artifacts: 'infra/tfplan.txt', fingerprint: true
            }
        }

        stage('Terraform Apply') {
            steps {
                script {
                    // timeout wraps the input so it can't hang forever
                    timeout(time: 15, unit: 'MINUTES') {
                        env.APPROVER = input(
                            message: 'Apply this Terraform plan?',
                            ok: 'Apply',
                            submitterParameter: 'APPROVER'
                        )
                    }
                }
                dir('infra') {
                    sh 'terraform apply -auto-approve tfplan'
                }
            }
        }
    }

    post {
        success {
            echo "${JOB_NAME} build ${BUILD_NUMBER} applied successfully, approved by ${env.APPROVER}"
        }
        failure {
            echo "${JOB_NAME} build ${BUILD_NUMBER} FAILED — see ${BUILD_URL}console"
        }
        always {
            echo "Finished with status: ${currentBuild.currentResult}"
        }
    }
}
```

Points to draw out:

- **`terraform fmt -check`** exits non-zero if any file isn't correctly formatted — it **reports**, it doesn't reformat. Putting the cheapest, fastest check first means a trivial mistake fails in seconds rather than after a slow plan
- **`terraform show -no-color tfplan > tfplan.txt`** converts the binary plan into readable text; `-no-color` strips terminal escape codes that would otherwise litter the file. Then `>` redirects it — Session 2 again
- **`archiveArtifacts`** attaches the file permanently. Six months later you can answer *"what exactly did we change on 14 March?"* — **an audit trail the Azure Activity Log genuinely cannot give you**, which was the original complaint in Session 1
- **`timeout` wrapping the `input`** matters more than it looks. Without it, an un-approved build holds a Jenkins executor open **indefinitely** — and, worse, **holds the Terraform state lock**, blocking everyone else's pipeline. That's the locking mechanism from Session 5, biting in a way you'd never predict
- **`submitterParameter`** captures who clicked Apply, so the approval is attributable

<br>
<br>

*(Take a 1 hour lunch break here.)*

<br>
<br>

### 14:00–14:45 — The Pull Request Workflow: Plan on PR, Apply on Merge
*(Activity: 15 min)*

We've built the mechanics. Now let's connect them to the picture from day one.

*REFER TO RESOURCE 2 - SLIDEE* <br>
![toolchain-2](./resources/toolchain-2.png)

The mature real-world pattern separates **plan** from **apply** based on *what kind of Git event* triggered the pipeline:

- **Pull Request opened or updated** → run `terraform plan` **only**, and surface the output for reviewers
- **Pull Request merged to `main`** → run `terraform apply` automatically, **no manual gate** — because the review already happened

**ASK** <br>
How does that map onto the diagram from Session 1? <br>
**ANSWER** <br>
**It *is* that diagram.** "Terraform plan posted for review before merge, apply happens automatically after merge" was a picture on a slide on day one. Today it becomes a thing that runs.

**ASK** <br>
Why is a `plan` on a pull request a genuinely better review artifact than the `.tf` diff alone? <br>
**ANSWER** <br>
Because **a diff shows what the code says; a plan shows what will actually happen to real infrastructure.** A one-line change can produce `1 to add, 0 to change, 1 to destroy` — and the reviewer only sees that catastrophic replacement in the **plan output**, not in the diff. It's the difference between reviewing **intent** and reviewing **consequence**. <br>
The parallel from your own work: reading a schema migration versus reading the SQL it will actually execute.

#### Implementing it: the `when` directive

The mechanism is `when`, which makes a stage **conditional**.

```groovy
stage('Terraform Plan') {
    steps {
        dir('infra') {
            sh 'terraform plan -out=tfplan'
            sh 'terraform show -no-color tfplan > tfplan.txt'
        }
        archiveArtifacts artifacts: 'infra/tfplan.txt'
    }
}

stage('Terraform Apply') {
    // Only run this stage on the main branch
    when {
        branch 'main'
    }
    steps {
        dir('infra') {
            sh 'terraform apply -auto-approve tfplan'
        }
    }
}
```

Read it plainly: **Plan runs on every branch. Apply runs only on `main`.**

A feature branch or PR gets a plan and an archived artifact, and nothing else. Once merged to `main`, the same pipeline runs again and this time Apply is eligible.

Note the `input` gate has **gone** from Apply — because the PR approval was the gate.

**ASK** <br>
`when { branch 'main' }` only works if Jenkins knows which branch it's building. Our job is hard-coded to `*/main`. What job type do we need? <br>
**ANSWER** <br>
A **Multibranch Pipeline** — the job type we named but didn't build in Session 3. It scans the whole repository, automatically creates a sub-job for **every branch and every open pull request** containing a `Jenkinsfile`, and deletes them when the branch goes. That's what makes `when { branch ... }` meaningful, and it's the missing piece that turns our single pipeline into a real PR workflow.

#### Setting it up

*(In the Jenkins UI — Dashboard)*
- **New Item** → `terraform-multibranch` → **Multibranch Pipeline** → **OK**
- **Branch Sources** → **Add source** → **Git**
- **Project Repository**: your fork's URL
- **Credentials**: `github-credentials`
- **Scan Multibranch Pipeline Triggers** → tick **Periodically if not otherwise run** → **1 minute**
- **Save**

Jenkins immediately scans and creates a job per branch.

**HANDS ON (15 min)** <br>
Demonstrate the full flow yourself:

1. Add `when { branch 'main' }` to your Apply stage and push to `main`
2. *(Run from `~/toolchain-day/<your-fork>`)* Create a branch and push it:
   ```bash
   git checkout -b add-storage-tag
   # make a change to your .tf files
   git add . && git commit -m "Add a tag" && git push origin add-storage-tag
   ```
3. *(On GitHub)* Open a **Pull Request** from that branch into `main`
4. *(In the Jenkins UI — `terraform-multibranch`)* Within a minute a job appears for the branch, runs, produces a plan, archives it — and **skips Apply**
5. **Download the archived `tfplan.txt`** from the branch build and read it as a reviewer would
6. *(On GitHub)* Merge the PR, and watch the `main` job run — this time Apply **does** execute
**END OF NOTE**

**💬 SLACK — snippet 7**:
```groovy
stage('Terraform Apply') {
    when {
        branch 'main'          // only on main — PRs get a plan only
    }
    steps {
        dir('infra') {
            sh 'terraform apply -auto-approve tfplan'
        }
    }
}
```
```bash
git checkout -b add-storage-tag
# ...make a change...
git add . && git commit -m "Add a tag" && git push origin add-storage-tag
# then open a PR on GitHub
```

**Solution**

The flow as described. What students should observe:

- On the **branch** job: Checkout ✅ → Init ✅ → Format ✅ → Validate ✅ → Plan ✅ → **Apply shown as skipped** (grey, not red)
- On the **main** job after merging: all stages run, including Apply

**NOTE FOR TRAINERS** <br>
Posting the plan **as a comment on the PR** is the one piece we're not building today — it needs either the GitHub Branch Source plugin's checks API or a small `curl` against the GitHub API with a token. **Great stretch exercise, poor use of classroom time**, since it's API plumbing rather than a new concept. Archiving the plan demonstrates the same idea: **the reviewer sees the consequence, not just the diff.** <br>
**END OF NOTE**

**ASK** <br>
In this model, what is now the **single** thing standing between a developer and production infrastructure? <br>
**ANSWER** <br>
**A pull request approval.** Which is exactly the point — and it's a far better control than a person carefully typing commands, because it's a decision made **with full information** (the plan output), **attributed** to a named human, **recorded permanently**, and made **before** anything happens rather than during. <br>
That's the "you build it, you run it" thread from day one, with a safety rail that scales to a whole team. And notice: **it's the same control you already use for application code.** You've just extended it to servers.

<br>
<br>

### 14:45–15:00 — Failure Modes: What Breaks and What It Costs
*(Research activity: 15 min)*

Before the capstone, let's think properly about what goes wrong. **Designing pipelines is largely about deciding where you want failures to happen.**

**ASK** <br>
If `Terraform Plan` fails outright — a typo in a `.tf` file — does anything happen to real Azure infrastructure? <br>
**ANSWER** <br>
**No.** `apply` never runs. Splitting plan and apply into separate stages is a **deliberate safety boundary**, not tidiness. Cost of this failure: a few wasted minutes.

**ASK** <br>
What if `apply` itself fails halfway — three resources created, the fourth errors? <br>
**ANSWER** <br>
The interesting one. Terraform's **state reflects exactly what did get created**. There's **no automatic rollback** — you saw this when three AD users couldn't all take the same name. But because state is accurate, **re-running once the problem is fixed attempts only the missing resources**. That's why idempotency matters: **a partially-failed run must be safe to retry.** Cost: real resources exist in a half-built state until someone fixes it.

**ASK** <br>
Two engineers merge PRs within thirty seconds of each other. Two pipeline runs start against the same state. What happens? <br>
**ANSWER** <br>
The first takes a **lock** on the state blob. The second finds it locked and **waits, then fails** with a clear message naming who holds the lock and since when. That's the **correct** behaviour — far better than both proceeding and corrupting state. And it's why the `timeout` on our approval gate mattered: **a build waiting forever on approval holds that lock and blocks the whole team.**

**ASK** <br>
The pipeline runs green, applies cleanly — and the application is broken. What did the pipeline get wrong? <br>
**ANSWER** <br>
**Nothing**, and that's the point worth landing. **A green pipeline means "the steps I was told to run succeeded", not "the system works".** If you didn't write a test, the pipeline can't run one. That's why the capstone puts a test stage *before* build and deploy, and why **Kubernetes readiness probes** matter — the pipeline verifies the **process**, the probes verify the **result**.

**A rough hierarchy of where you want failures to land:**

| Fails at | Cost |
|---|---|
| `fmt` / `validate` | Seconds. **Ideal** |
| Tests | Minutes. Good — caught before anything is built |
| `plan` | Minutes. Good — nothing real touched |
| `apply` | Real resources in a partial state. Recoverable, but work |
| Runtime, after deploy | **Users affected. Worst** |

**Design pipelines so the cheapest checks fail first.** That's why `fmt` came before `validate`, which came before `plan`.

**HANDS ON — research (15 min)** <br>
**In pairs**, produce short written answers:

1. **What's missing?** Our pipeline has no monitoring, no alerting, no staging environment and no automated rollback. Pick **two** of those and write two sentences each on what you'd add and why
2. **DORA metrics.** Look up the **four DORA metrics** (deployment frequency, lead time for changes, change failure rate, time to restore). For each, say whether today's pipeline would improve it, and how
3. **GitOps.** Research **ArgoCD** or **Flux**. How does a "pull-based" GitOps deployment differ from the "push-based" pipeline we've built, and what does it buy you?
4. **Blast radius.** Our Service Principal has `Contributor` on the whole subscription, and Jenkins holds its secret. Write down **three** things an attacker could do with access to your Jenkins, and one change that would reduce that
5. **(Stretch)** Research **`terraform force-unlock`** — when would a pipeline leave a lock behind, and what's the safe way to clear it?
**END OF NOTE**

**Solution**

**1 — What's missing.** Expect answers like:

| Gap | What you'd add |
|---|---|
| **Monitoring** | Azure Monitor / Application Insights on the cluster and app, so you know it's broken **before a user tells you** |
| **Alerting** | A `post { failure }` hook posting to Slack, plus alerts on the running app, so failures reach a human |
| **Staging** | A second environment (a `test.tfvars`, a separate state key) that gets deployed to first, so production isn't the first place a change runs |
| **Automated rollback** | `kubectl rollout undo` in the `failure` block, or a health check that triggers reversion |

**2 — DORA metrics.**

| Metric | Does today's pipeline improve it? |
|---|---|
| **Deployment frequency** | **Yes, enormously.** Deploying is now a `git push`, so there's no reason to batch changes |
| **Lead time for changes** | **Yes.** Commit to running application is minutes, not a scheduled release window |
| **Change failure rate** | **Somewhat.** Tests, `validate`, `plan` and readiness probes all catch failures — but only failures you thought to check for |
| **Time to restore** | **Partly.** `kubectl rollout undo` and reverting a Git commit are both fast, but nothing is automatic yet. This is the weakest of the four |

That last row is a genuinely useful observation: **the pipeline is good at preventing bad changes and less good at recovering from them.** Most teams' pipelines look the same.

**3 — GitOps.** Our pipeline is **push-based**: Jenkins holds cluster credentials and *pushes* changes in. A GitOps tool like ArgoCD is **pull-based**: an agent *inside* the cluster watches a Git repository and pulls changes in, continuously reconciling the cluster toward what's in Git. <br>
Two advantages: **no external system needs cluster credentials** (smaller blast radius), and **drift is corrected continuously** rather than only when a pipeline runs — which is the Kubernetes reconciliation loop from Session 6, applied to your whole deployment.

**4 — Blast radius.** With access to Jenkins, an attacker could: **read every stored credential**, **run `terraform destroy` against your entire subscription**, **push a malicious image** to your registry and have the pipeline deploy it, or **create new Service Principals** to persist after being locked out. <br>
The single biggest reduction: **scope the Service Principal to specific resource groups rather than the subscription** — a compromised pipeline then damages one environment, not everything. Second: **don't run Jenkins as root with the Docker socket mounted**, which is the training shortcut we took this morning.

**5 — `force-unlock`.** A pipeline leaves a lock behind when a job is **cancelled or killed mid-apply** — including someone aborting a build stuck at an approval gate. `terraform force-unlock <lock-id>` clears it. **It's dangerous because you might be removing a lock another process is legitimately still using**, which is exactly the corruption locking prevents. Safe procedure: confirm nothing is running, check the lock's holder and timestamp in the error message, then unlock.

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>
### 15:15–16:45 — Capstone: The Complete Pipeline, Code to Running Application
*(Activity: 90 min)*

This is the end of the course, and the exercise is the whole course.

You're going to build **one pipeline** that takes a code change and, with no human typing a single command, runs it through: **test → build a Docker image → push it to a registry → provision infrastructure with Terraform → deploy to Kubernetes.**

Every stage uses a tool you already know. **The learning is in making them cooperate.**

Work individually or in pairs. Build it **stage by stage, getting each one green before adding the next** — that's not just teaching advice, it's how you'd genuinely build this at work. **Do not write all six stages and then debug.**

**NOTE FOR TRAINERS** <br>
Say the build-incrementally instruction twice, and enforce it. A student who writes the whole `Jenkinsfile` and pushes will hit three failures at once with no idea which is which, and will burn forty minutes. A student adding one stage at a time gets a green build every ten minutes and knows exactly what broke. <br>
It's also, not incidentally, the same lesson as small frequent commits from Session 1. <br>
**END OF NOTE**

---

#### Part 1 (≈10 min) — Docker Hub credentials

*(In your browser — [hub.docker.com](https://hub.docker.com))*
- **Account Settings → Security → New Access Token**, **Read & Write**, **Generate**
- **Copy it now** — shown once

*(In the Jenkins UI — Manage Jenkins → Credentials → Add Credentials)*
- **Kind**: Username with password
- **Username**: your Docker Hub username
- **Password**: the token
- **ID**: `dockerhub-credentials`

*(Run from `~/toolchain-day/<your-fork>/app`)*
```bash
cat package.json
cat test.js
cat Dockerfile
```

**Read `test.js`** and note the ending — `process.exit(1)` on failure, `process.exit(0)` on success. **That exit code is the only signal that reaches Jenkins.**

---

#### Part 2 (≈15 min) — Test, build and push

Add three stages **before** your existing Terraform ones.

**Jenkinsfile**
```groovy
    environment {
        ARM_CLIENT_ID       = credentials('azure-client-id')
        ARM_CLIENT_SECRET   = credentials('azure-client-secret')
        ARM_SUBSCRIPTION_ID = credentials('azure-subscription-id')
        ARM_TENANT_ID       = credentials('azure-tenant-id')

        IMAGE_NAME = "your-dockerhub-username/toolchain-app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        // NEW
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

        // ... your existing Terraform stages follow ...
    }
```

All revision from Session 3: the per-stage `docker` agent giving a clean Node environment, `$BUILD_NUMBER` as a traceable image tag, `withCredentials` scoping the secret, and `--password-stdin` keeping the password off the command line.

**Push and get these three green before continuing.**

---

#### Part 3 (≈20 min) — Provision an AKS cluster with Terraform

Now Terraform provisions the thing the application runs on.

*(Run from `~/toolchain-day/<your-fork>/infra`)*
- Run: `touch aks.tf`

**infra/aks.tf**
```tf
resource "azurerm_resource_group" "aks_rg" {
  name     = "rg-toolchain-jbloggs"
  location = "uksouth"
}

resource "azurerm_kubernetes_cluster" "aks" {
  name                = "aks-toolchain-jbloggs"
  location            = azurerm_resource_group.aks_rg.location
  resource_group_name = azurerm_resource_group.aks_rg.name
  dns_prefix          = "toolchain"

  default_node_pool {
    name       = "default"
    node_count = 1
    vm_size    = "Standard_B2s"
  }

  identity {
    type = "SystemAssigned"
  }

  tags = {
    environment = "training"
  }
}

output "cluster_name" {
  value = azurerm_kubernetes_cluster.aks.name
}

output "cluster_resource_group" {
  value = azurerm_resource_group.aks_rg.name
}
```

Most of this is familiar shape. Two new pieces:

- **`default_node_pool { }`** — the worker nodes you inspected in the Portal in Session 6, now **declared as code**. `node_count = 1` and a small `vm_size` keep the cost down
- **`identity { type = "SystemAssigned" }`** — gives the cluster its own **managed identity**, so it can create Azure resources (like the load balancer a `LoadBalancer` Service needs) **without any credential being stored anywhere**. It's a Service Principal that Azure creates and rotates for you

**ASK** <br>
`SystemAssigned` identity — no credential stored, created and rotated by Azure. Why is that better than giving the cluster a Service Principal you made? <br>
**ANSWER** <br>
Because **there's no secret for you to store, leak, or forget to rotate.** Azure manages the whole lifecycle, and the identity dies with the resource. Compare with our Jenkins Service Principal, whose password is sitting in a credentials store and will still be valid in a year unless someone rotates it. <br>
**Where a managed identity is available, it's the better answer** — and it's a genuinely useful thing to know exists, because it eliminates a whole class of secret-handling problems.

**ASK** <br>
In Session 6 you created a cluster by clicking through the Portal. What does having it in Terraform change? <br>
**ANSWER** <br>
Everything we've argued for all course: **reviewable in a PR, versioned, reproducible.** But there's one more that only appears now — it makes the cluster **disposable**, which changes behaviour. If standing up a full environment is one pipeline run, you can afford to **build one per feature branch and tear it down after.** That's only realistic when it's code.

**NOTE FOR TRAINERS** <br>
An AKS cluster takes **5–10 minutes** to provision. The first pipeline run through this stage will feel like it's hung. **Warn students beforehand**, and use the wait to talk through what Apply is doing. Subsequent runs are fast because Terraform sees the cluster already exists. <br>
**END OF NOTE**

---

#### Part 4 (≈25 min) — Deploy to Kubernetes

Now the final stage — **where the tools genuinely have to cooperate.**

**ASK** <br>
The pipeline just created a brand-new cluster. `kubectl` on the Jenkins agent has never heard of it. How does the deploy stage know where to connect? <br>
**ANSWER** <br>
It has to **fetch credentials at run time**, using `az aks get-credentials` — exactly the command you ran in Session 6, except the values come from **Terraform's outputs** rather than being copied out of the Portal. <br>
**That's integration problem number three, and it's the most interesting one:** Terraform knows what it built, and has to pass that knowledge to the next stage. **Nothing about the cluster is hardcoded.**

**Jenkinsfile** — add after Terraform Apply:
```groovy
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Read what Terraform built
                    def clusterName = sh(
                        script: 'cd infra && terraform output -raw cluster_name',
                        returnStdout: true
                    ).trim()
                    def clusterRg = sh(
                        script: 'cd infra && terraform output -raw cluster_resource_group',
                        returnStdout: true
                    ).trim()

                    // Log in to Azure as the Service Principal
                    sh '''
                        az login --service-principal \
                          -u $ARM_CLIENT_ID \
                          -p $ARM_CLIENT_SECRET \
                          --tenant $ARM_TENANT_ID
                    '''

                    // Fetch kubeconfig for the cluster Terraform just made
                    sh "az aks get-credentials --resource-group ${clusterRg} --name ${clusterName} --overwrite-existing"

                    // Substitute the real image, then apply
                    sh "sed -i 's|IMAGE_PLACEHOLDER|${IMAGE_NAME}:${IMAGE_TAG}|g' k8s/deployment.yaml"
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    sh 'kubectl rollout status deployment/toolchain-app --timeout=180s'
                    sh 'kubectl get service toolchain-app'
                }
            }
        }
```

Several new things, each worth explaining:

- **`terraform output -raw <name>`** prints an output value with no quotes or formatting. **`returnStdout: true`** captures what a shell command printed into a Groovy variable, and **`.trim()`** strips the trailing newline. **This is the join between Terraform and Kubernetes**
- **`az login --service-principal`** — a non-interactive login using the same four credentials. No browser, no human
- **`--overwrite-existing`** — replaces any stale kubeconfig entry with the same name
- **`sed -i 's|OLD|NEW|g' file`** — stream editor, editing **in place**. We use `|` as the delimiter instead of `/` because **the image name contains slashes**. `g` replaces every occurrence
- **`kubectl rollout status --timeout=180s`** — **the most important line in the stage**

**ASK** <br>
Why does that last line matter so much? Without it, `kubectl apply` succeeds and the stage goes green. <br>
**ANSWER** <br>
Because **`kubectl apply` only means "Kubernetes accepted my YAML"** — not "the new version is running". Without `rollout status`, a deployment whose Pods crash on startup would still show a **green pipeline**, and you'd tell the whole team the release succeeded while it silently failed. <br>
**That's integration problem number four, and it's the exit-code contract arriving at the very end of the chain:** the pipeline must not report success until the thing it deployed is actually working. And notice it only works because your `deployment.yaml` has a **readiness probe** — without one, Kubernetes considers a Pod ready the moment the process starts, and `rollout status` would return green on a broken app.

---

#### Part 5 — Run the whole thing

*(Run from `~/toolchain-day/<your-fork>`)*
```bash
git add .
git commit -m "Complete pipeline: test, build, push, provision, deploy"
git push origin main
```

Watch the Stage View march through:

**Checkout → Install & Test → Build Image → Push Image → Terraform Init → Format → Validate → Plan → Apply → Deploy to Kubernetes**

Once green, find your application:

*(Run from `~/toolchain-day/<your-fork>`)*
```bash
az aks get-credentials --resource-group rg-toolchain-jbloggs --name aks-toolchain-jbloggs
kubectl get service toolchain-app
```

*(In your browser)* — visit the `EXTERNAL-IP`.

**Then do the thing that proves it all works.** Change something visible in `app/index.js`, commit, push — **and touch nothing else.** Watch the pipeline test it, build a new image tagged with the new build number, push it, confirm the infrastructure is unchanged, and perform a rolling update on the cluster. Refresh your browser and see the change live.

**💬 SLACK — snippet 8**, the AKS config — *(contents as `infra/aks.tf` above)*.

**💬 SLACK — snippet 9**, the Deploy stage — **the hardest block of the day, definitely post it** — *(contents as above)*.

**💬 SLACK — snippet 10**, the complete `Jenkinsfile` — post at ~16:00 for anyone assembling.

**Solution**

The complete `Jenkinsfile`:

```groovy
pipeline {
    agent any

    environment {
        ARM_CLIENT_ID       = credentials('azure-client-id')
        ARM_CLIENT_SECRET   = credentials('azure-client-secret')
        ARM_SUBSCRIPTION_ID = credentials('azure-subscription-id')
        ARM_TENANT_ID       = credentials('azure-tenant-id')

        IMAGE_NAME = "jbloggs/toolchain-app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Install & Test') {
            agent { docker { image 'node:20' } }
            steps {
                dir('app') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }

        stage('Build Image') {
            steps {
                dir('app') { sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .' }
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

        stage('Terraform Init') {
            steps { dir('infra') { sh 'terraform init' } }
        }

        stage('Terraform Format') {
            steps { dir('infra') { sh 'terraform fmt -check -recursive' } }
        }

        stage('Terraform Validate') {
            steps { dir('infra') { sh 'terraform validate' } }
        }

        stage('Terraform Plan') {
            steps {
                dir('infra') {
                    sh 'terraform plan -out=tfplan'
                    sh 'terraform show -no-color tfplan > tfplan.txt'
                }
                archiveArtifacts artifacts: 'infra/tfplan.txt', fingerprint: true
            }
        }

        stage('Terraform Apply') {
            steps {
                script {
                    timeout(time: 15, unit: 'MINUTES') {
                        input message: 'Apply this Terraform plan?', ok: 'Apply'
                    }
                }
                dir('infra') { sh 'terraform apply -auto-approve tfplan' }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def clusterName = sh(
                        script: 'cd infra && terraform output -raw cluster_name',
                        returnStdout: true
                    ).trim()
                    def clusterRg = sh(
                        script: 'cd infra && terraform output -raw cluster_resource_group',
                        returnStdout: true
                    ).trim()

                    sh '''
                        az login --service-principal \
                          -u $ARM_CLIENT_ID -p $ARM_CLIENT_SECRET --tenant $ARM_TENANT_ID
                    '''

                    sh "az aks get-credentials --resource-group ${clusterRg} --name ${clusterName} --overwrite-existing"

                    sh "sed -i 's|IMAGE_PLACEHOLDER|${IMAGE_NAME}:${IMAGE_TAG}|g' k8s/deployment.yaml"
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    sh 'kubectl rollout status deployment/toolchain-app --timeout=180s'
                    sh 'kubectl get service toolchain-app'
                }
            }
        }
    }

    post {
        success {
            echo "Deployed ${IMAGE_NAME}:${IMAGE_TAG} — build ${BUILD_NUMBER}"
        }
        failure {
            echo "FAILED at build ${BUILD_NUMBER} — see ${BUILD_URL}console"
        }
        always {
            echo "Finished with status: ${currentBuild.currentResult}"
        }
    }
}
```

**Common failures, in the order you'll hit them:**

| Symptom | Cause |
|---|---|
| `terraform: not found` | The custom Jenkins image wasn't used or wasn't built properly |
| `docker: permission denied` | The Docker socket wasn't mounted, or `-u root` omitted |
| Init fails on the backend | Service Principal can't reach the backend storage account, or it was destroyed in Session 5 |
| Apply hangs ~8 minutes | **Normal.** AKS provisioning is genuinely slow |
| `az login` fails in the deploy stage | Use `'''` triple single quotes so the **shell** expands `$ARM_*`, not Groovy |
| `terraform output` returns nothing | You forgot to `cd infra`, or the outputs aren't defined |
| `rollout status` times out | Image architecture mismatch (arm64 on amd64), or the container is crashing — `kubectl describe pod` |
| `EXTERNAL-IP` stays `<pending>` | Give it two minutes; Azure is provisioning a load balancer |

**Stretch, if anyone gets there:**
- Add `when { branch 'main' }` to Apply and Deploy, and run as a Multibranch Pipeline so PRs only plan
- Add `post { failure { sh 'kubectl rollout undo deployment/toolchain-app || true' } }`
- Replace the `sed` substitution with `kubectl set image` and consider which you prefer, and why

<br>
<br>

### 16:45–17:00 — Wrap-up, Tear Everything Down, & Course Close
*(Activity: 10 min)*

#### Tear down — do this properly, and first

Today's infrastructure is the most expensive on the course.

*(Run from `~/toolchain-day/<your-fork>`)*
```bash
# Delete the Service FIRST — releases the Azure load balancer and public IP
kubectl delete -f k8s/deployment.yaml

# Then destroy everything Terraform created
cd infra
terraform destroy
# type: yes
```

Then verify — and check for AKS's second, auto-generated resource group:

*(Run from `~/toolchain-day`)*
```bash
az group list -o table
az resource list -o table
```

**NOTE FOR TRAINERS** <br>
AKS creates a second resource group named `MC_<rg>_<cluster>_<region>` holding node VMs, disks, load balancer and public IP. `terraform destroy` normally removes it with the cluster — **but not reliably if the Kubernetes Service wasn't deleted first**, because the load balancer it created isn't in Terraform's state. <br>
**Make every student run `az group list -o table` and confirm no `MC_` group survives.** Two minutes now versus a surprise bill later. <br>
**END OF NOTE**

**💬 SLACK — snippet 11**:
```bash
# 1. Service FIRST — releases the Azure load balancer + public IP
kubectl delete -f k8s/deployment.yaml

# 2. Then Terraform
cd infra && terraform destroy      # type: yes

# 3. CONFIRM — no 'MC_...' group should survive
az group list -o table
az resource list -o table

# 4. Stop Jenkins if you want your laptop's resources back
docker stop jenkins                # 'docker start jenkins' brings it back
```

#### What you built today

One pipeline, triggered by a `git push`, that:
- ran your tests in a clean containerised environment
- built a Docker image and tagged it traceably with the build number
- pushed it to a registry
- checked, planned and applied Terraform against Azure, with state **locked in a shared remote backend**
- paused for a human to review the **actual consequences** before touching infrastructure
- provisioned a Kubernetes cluster
- **fetched credentials for that cluster automatically, from Terraform's own outputs**
- performed a zero-downtime rolling update
- and **refused to report success until the rollout genuinely completed**

**Nobody typed a command.** And every decision is recorded: who pushed, who reviewed, who approved, what the plan said, which build produced which image.

#### The four integration problems

Worth naming them explicitly, because they're the real content of today:

1. **Remote state is a prerequisite** — a disposable agent has no local state
2. **The orchestrator needs every tool installed inside it** — the stock Jenkins image has none
3. **Terraform must pass what it built to the next stage** — via `terraform output`, not hardcoding
4. **"Accepted" is not "working"** — `kubectl apply` succeeding doesn't mean the app runs

**None of those are visible when you use the tools individually.** They only appear at the joins — which is why this session exists.

#### Where every session plugs in

- **Session 1, Azure** — Service Principals, RBAC and least privilege are what let Jenkins authenticate at all
- **Session 2, bash** — every `sh` step is bash, and the whole safety model rests on exit codes
- **Docker** — the app image, *and* the Jenkins image, *and* the per-stage build environments
- **Session 3, Jenkins** — stages, credentials, gating, `post` blocks, Multibranch
- **Sessions 4–5, Terraform** — unchanged `.tf` files; only *who runs them* changed. Remote state made it possible
- **Session 6, Kubernetes** — declarative YAML, rolling updates, readiness probes, and `rollout status` as the final honesty check
- **Git** — the source of truth for application code, infrastructure code *and* the pipeline definition, and the trigger for all of it

**ASK** *(the last question of the taught course)* <br>
On day one we said DevOps was "you build it, you run it", and that the wall between Dev and Ops came down. Looking at what you built today — **where did the wall actually go?** <br>
**ANSWER** <br>
**It became code, in one repository, reviewed the same way.** The application, the infrastructure it runs on, and the process that ships it all live in the same place, go through the same pull request, and are read by the same people. There's no longer a Dev artifact thrown over to an Ops process — **there's one repository and one pipeline.** <br>
That's not a cultural slogan any more. **You can point at the files.**

And the honest caveat: **none of this is finished.** No monitoring, no alerting, no secrets management beyond Jenkins' store, no staging environment, no automated rollback, no cost controls — you listed most of those yourselves before the break. Those are the next things you'd meet in a real team. **But the spine is right, and you built it.**

**Q&A** — take remaining questions. Confirm one final time that `az group list -o table` is clean for everyone.

<br>
<br>

### Exercise (take-home / reinforcement)

Individually or in pairs. **At least one practical and one research task.**

**Practical**

1. Rebuild the pipeline **from an empty repository**, from memory, up to a working `terraform plan`
2. Convert your job to a **Multibranch Pipeline** and add `when { branch 'main' }` so pull requests only plan. Open a PR and **prove Apply is skipped**
3. Add a `post { failure { ... } }` block running `kubectl rollout undo` so a failed deploy rolls itself back. Test it by pushing a deliberately broken image tag
4. Add a **parameter** letting you choose the environment (`dev`/`test`), and use it in resource names and the Terraform state key
5. Break the pipeline **four different ways** — one per stage — and record what each failure looks like. This is genuinely the fastest way to get good at diagnosing them

**Research**

6. Write a paragraph explaining to a new starter **why remote state is mandatory** for pipeline-run Terraform
7. **Pick one gap** from the failure-modes research (monitoring, alerting, staging, rollback) and write half a page on how you'd actually implement it here
8. **(Stretch)** Post the `terraform plan` output as a **comment on the pull request**, using the GitHub API and a token from the credential store
9. **(Stretch)** Research **Azure Container Registry** and swap Docker Hub for ACR. What changes in the pipeline, and what does it buy you?

**Solutions** *(for the guided ones — 2, 3, 4)*

**Take-home 2** — branch-conditional stages:
```groovy
stage('Terraform Apply') {
    when { branch 'main' }
    steps { dir('infra') { sh 'terraform apply -auto-approve tfplan' } }
}

stage('Deploy to Kubernetes') {
    when { branch 'main' }
    steps { /* ... as before */ }
}
```
On a PR branch these show as **skipped** (grey) while Plan still runs and archives its artifact.

**Take-home 3** — automatic rollback on failure:
```groovy
post {
    failure {
        echo "Build ${BUILD_NUMBER} failed — attempting rollback"
        // '|| true' so a rollback failure doesn't mask the ORIGINAL error
        sh 'kubectl rollout undo deployment/toolchain-app || true'
    }
}
```
The `|| true` is the operator from Session 2: if there's nothing to roll back to, we don't want that failure obscuring the real one in the logs.

**Take-home 4** — an environment parameter:
```groovy
parameters {
    choice(name: 'ENVIRONMENT', choices: ['dev', 'test'], description: 'Which environment to target')
}

// ...then in a stage:
sh "terraform plan -out=tfplan -var='environment=${params.ENVIRONMENT}'"
```

Remember from Session 4 that a `backend` block **cannot be interpolated** — so to vary the state key per environment you need **partial configuration**:
```groovy
sh "terraform init -backend-config='key=${params.ENVIRONMENT}/toolchain/pipeline.tfstate'"
```
Leave `key` out of the `backend` block in your `.tf` and supply it at init time.

<br>
<br>

### Conclusion

- **Inform** students this marks the end of the DevOps Toolchain Integration session — and the end of the **taught** course
- **Recap** that every tool has now been seen working together in one real, automated pipeline, and that today introduced **almost no new technology**: the skill was the wiring
- **Reinforce** the three ideas that carried across every session: **everything as code**, **fail cheaply and early**, and **honest exit codes** — the contract every automated system depends on
- **Name the four integration problems** again; they're the transferable lesson
- **Confirm** every student has torn down their infrastructure and checked `az group list -o table`, including any `MC_` resource group
- **Tell** students what's next: the **two-day capstone**, where they build all of this themselves, on an application of their own choosing, with no step-by-step instructions
- **Direct** students to the take-home exercises, the [Jenkins Pipeline documentation](https://www.jenkins.io/doc/book/pipeline/), and the [Terraform](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs) and [Kubernetes](https://kubernetes.io/docs/concepts/) docs

---

[Back](./README.md)

---
