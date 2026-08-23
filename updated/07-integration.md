# Integrating the DevOps Toolchain: Git, Jenkins, Terraform, Azure & Kubernetes — Trainer Script

The capstone day. Everything covered separately across this course — the shell, Docker, version control, CI/CD orchestration, Infrastructure as Code, the cloud, and container orchestration — assembled into one real, working, automated pipeline. This is written as a script — read it out, adapt where it feels natural, but the intent is that you could deliver this cold from the page.

## Organisation

### Audience

Trainees who have completed the full course: **Azure fundamentals**, **bash scripting & automation**, **Docker**, **Jenkins & CI/CD**, **Terraform & IaC on Azure**, **resource provisioning & remote state**, and **Kubernetes & AKS**.

Nothing today is a genuinely new tool. Every piece is something they've used in isolation. **The learning is the wiring** — and the specific problems that only appear once tools have to talk to each other unattended.

### How this document is laid out — read before delivering

Today involves more context-switching than any other session: a local terminal, the Jenkins UI, the Azure Portal, GitHub, and a Git repo. Every instruction block is labelled:

- *(Run from `~/toolchain-day/...`)* — a **local terminal** command, in that folder
- *(In the Jenkins UI — Dashboard)* — a **browser** action in Jenkins
- *(In the Azure Portal — Resource groups)* — a **browser** action in Azure
- *(On GitHub — your fork)* — a **browser** action on GitHub

Navigation is written as breadcrumbs, e.g. **Manage Jenkins → Credentials → Add Credentials**.

The folders we use today:

| Folder | What it's for |
|---|---|
| `~/toolchain-day` | Parent folder; the custom Jenkins image is built here |
| `~/toolchain-day/<your-fork>` | Your cloned fork — where the `Jenkinsfile` and `.tf` files live |

**Every activity has a `**Solution**` block** immediately afterwards.

### Duration & schedule

Full day, **09:00–17:00**, with a one-hour lunch and two 15-minute breaks. Roughly **4 hours of instruction and 3 hours of hands-on**, weighted towards a large end-of-day capstone.

| Time | Section | Shape |
|---|---|---|
| 09:00–09:15 | Welcome: the whole course in one diagram | Talk |
| 09:15–10:00 | The toolchain — why no single tool does everything | Talk + discussion |
| **10:00–10:15** | **Break** | |
| 10:15–11:15 | A Jenkins that can actually do the job | **Exercise** |
| 11:15–12:15 | Authenticating to Azure & the first Terraform stages | **Exercise** |
| 12:15–13:00 | The apply gate: saved plans and human approval | **Exercise + Challenge** |
| **13:00–14:00** | **Lunch** | |
| 14:00–14:45 | The Pull Request workflow: plan on PR, apply on merge | Talk + demo |
| 14:45–15:00 | Failure modes: what breaks and what it costs | Talk + discussion |
| **15:00–15:15** | **Break** | |
| 15:15–16:45 | Capstone: the complete pipeline, code to running application | **Exercise (1 hr 30 min)** |
| 16:45–17:00 | Wrap-up, tear everything down, & course close | Talk |

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
- **Make sure**, *before the session starts*, every student has:
  - **Docker Desktop** installed and running — we run Jenkins as a container again
  - A **GitHub account** with the starter repo **forked and cloned**
  - A **Docker Hub** account with an access token
  - An **Azure subscription** with remaining credit
  - Their **Service Principal** credentials (`ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, `ARM_SUBSCRIPTION_ID`, `ARM_TENANT_ID`) — we'll recreate one if not
  - Their **remote backend storage account** from the Terraform provisioning session, if it still exists

**NOTE FOR TRAINERS — the four things that will eat your day** <br>
**(1) Everything at once.** This is the highest-moving-parts day of the course. If a student is missing Docker, or a Service Principal, or their fork — sort it in the first fifteen minutes, not at 15:15. <br>
**(2) The Jenkins image.** The stock `jenkins/jenkins:lts` image has **no** Terraform, Azure CLI, kubectl or Docker CLI. We build a custom image in Section 3. Every "command not found" failure today traces back to this. <br>
**(3) Webhooks can't reach localhost.** Same as the Jenkins day. Use **Poll SCM** as the reliable classroom path; teach webhooks as the concept and offer ngrok as a stretch. <br>
**(4) Cost.** By the end of the capstone students may have an AKS cluster, a container registry and VMs running. The 16:45 teardown is not optional — build in the full fifteen minutes. <br>
**END OF NOTE**

## Learning objectives

- **Explain** where each tool sits in a DevOps toolchain, and why no single tool covers the whole loop
- **Build** a Jenkins environment capable of running Terraform, the Azure CLI, kubectl and Docker
- **Authenticate** Jenkins to Azure with a Service Principal, and to Docker Hub, without hardcoding any secret
- **Write** a multi-stage `Jenkinsfile` that runs `terraform init`, `plan` and `apply`
- **Explain** why remote state is a hard requirement for pipeline-run Terraform
- **Implement** a human approval gate, and explain when to remove it
- **Design** a PR-based workflow: plan on pull request, apply on merge
- **Assemble** a complete pipeline: code change → build image → push → provision infrastructure → deploy to Kubernetes
- **Reason** about failure modes at each stage and what each one costs

<br>

## Sequence

### 09:00–09:15 — Welcome: The Whole Course in One Diagram

Morning. Today is different from every other session on this course, and it's worth saying how.

**There is almost no new technology today.** No new tool to install and learn from scratch. Every single component we use, you have already used on its own:

- You can write **bash** and you understand exit codes
- You can build a **Docker** image and push it to a registry
- You can build a **Jenkins** pipeline that tests, builds and pushes
- You can write **Terraform** that provisions real Azure infrastructure
- You understand **remote state**, and why it matters
- You can deploy an application to **Kubernetes** with declarative YAML

Today is about the **wiring**. And wiring is genuinely where the interesting problems live, because a set of tools that each work perfectly alone will still fail the moment they have to cooperate without a human present.

Here's the shape of what we're building:

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

Every arrow in that diagram is something you'll build today.

**ASK** <br>
Across every session on this course, one question has kept coming back in different clothing. What is it? <br>
**ANSWER** <br>
*Who actually typed that command?* Every time — `terraform apply`, `docker push`, `kubectl apply` — the answer has been "a human, at a laptop." Today the answer becomes "nobody, it just happened, and there's a record of exactly why."

Lunch is at 1 for an hour, breaks mid-morning and mid-afternoon. And a hard rule: we **tear everything down** at 16:45. By the end of today you'll have real infrastructure running.

**First thing — everyone check their prerequisites:**

*(Run from `~/`)*
```bash
mkdir -p ~/toolchain-day && cd ~/toolchain-day
docker --version
az account show
git --version
```

If any of those fail, flag it now.

<br>
<br>

### 09:15–10:00 — The Toolchain: Why No Single Tool Does Everything

#### The gap we're closing

**ASK** <br>
Every time we've run `terraform apply` on this course, who typed it? <br>
**ANSWER** <br>
Us — a human, sat at a laptop.

That's the gap today closes. We first drew this picture in the **Azure Fundamentals** session, on day one: git push, a PR reviewed alongside a `terraform plan`, then merge triggers an automatic apply. It was a diagram then. Today it becomes a thing that runs.

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
Terraform can run scripts with `remote-exec`, and Kubernetes can restart failed containers. Why not just pick one tool and push it as far as it goes? <br>
**ANSWER** <br>
Because each tool is excellent at the thing it was built for and mediocre-to-dangerous outside it — you saw this directly. Terraform's `remote-exec` provisioner "worked", but only ran at creation time and couldn't be re-run, which is why HashiCorp themselves call provisioners a last resort. Kubernetes keeps containers alive but has no idea how to create the VNet its cluster sits in. Trying to make one tool do everything gets you the worst version of each job. The toolchain exists because the problems are genuinely different shapes.

#### Where Jenkins fits

**Jenkins** is the glue. Its entire job is: *watch for an event, then run some tasks in a defined order, and stop if any of them fail.*

It doesn't know what Terraform is. It doesn't know what Kubernetes is. It runs shell commands and checks exit codes.

**ASK** <br>
Jenkins genuinely just runs shell commands and checks exit codes. Why did we spend a whole session on exit codes in the bash module? <br>
**ANSWER** <br>
Because exit codes are the **entire contract** between your scripts and Jenkins. A stage passes if the last command exits `0` and fails otherwise — that's it. A script that swallows an error and exits `0` anyway will make a broken pipeline look green and ship broken infrastructure. Everything Jenkins does about safety rests on commands being honest about failure, which is exactly why we hammered it.

**ASK** <br>
Jenkins isn't the only tool that does this. What else might you use? <br>
**ANSWER** <br>
GitHub Actions, GitLab CI, Azure Pipelines, CircleCI. Jenkins is one of the oldest and most widely adopted, highly extensible via plugins, and — importantly for learning — **self-hosted**, so every piece is something we set up and can inspect rather than a black box on someone else's platform.

**ASK** <br>
What's the trade-off of self-hosting Jenkins versus a SaaS tool like GitHub Actions? <br>
**ANSWER** <br>
Full control, no vendor lock-in, and nothing running outside infrastructure we manage — but we're on the hook for patching, scaling and maintaining the Jenkins server itself, which a SaaS tool handles for you. There's also a subtler cost: a self-hosted Jenkins with credentials for your whole cloud estate is an extremely attractive target, and securing it properly is real work.

#### The problem you haven't hit yet

Here's something that's about to matter enormously.

**ASK** <br>
Your Terraform state file has been sitting on your laptop all week. A Jenkins build runs inside a fresh container that's thrown away afterwards. What happens when that pipeline runs `terraform apply`? <br>
**ANSWER** <br>
Disaster, if state is local. The build starts with **no state file at all**, so Terraform believes nothing exists and plans to create everything from scratch — duplicating resources or erroring on name collisions. Then the container is destroyed and whatever state it wrote vanishes, so the *next* build starts from nothing again. **Remote state isn't a nice-to-have for pipeline Terraform, it's a hard prerequisite** — which is exactly why we built that backend storage account.

That's the first genuine integration problem of the day: two tools that each worked fine alone are incompatible until you change something. There'll be more.

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>

### 10:15–11:15 — A Jenkins That Can Actually Do the Job
*(Exercise)*

You built a Jenkins in the CI/CD session. Today it needs to do considerably more, so we're building a better one.

**ASK** <br>
Our pipeline needs to run `terraform`, `az`, `kubectl` and `docker`. The official `jenkins/jenkins:lts` image contains exactly none of them. What happens when a stage calls `terraform init`? <br>
**ANSWER** <br>
`terraform: not found`, a non-zero exit code, and a red stage. Which is *correct behaviour* — the pipeline failed honestly. But it means our first job is building a Jenkins image that has the tools it needs. This is the Docker skill from earlier in the course doing real work: we're solving a dependency problem by building a purpose-built image.

#### Build a capable Jenkins image

*(Run from `~/toolchain-day`)*
- Run: `touch Dockerfile`
- Then open it: `code Dockerfile`

```dockerfile
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

# kubectl, via the Azure CLI
RUN az aks install-cli

USER jenkins
```

Read it through — every line is Docker knowledge you already have:

- **`FROM jenkins/jenkins:lts-jdk17`** — start from official Jenkins. `lts` is Long Term Support
- **`USER root`** — installing packages needs admin rights
- **The first `RUN`** — base tools plus `docker.io`, which gives us the Docker **client**. Note `&&` chaining (only continue if the previous step succeeded — straight from the bash session) and `rm -rf /var/lib/apt/lists/*` to discard the package index and keep the image smaller
- **The Terraform block** — adds HashiCorp's apt repository. The `gpg --dearmor` step imports their signing key so apt can verify the packages are genuinely from HashiCorp. `$(lsb_release -cs)` is command substitution, printing the Debian release codename
- **Azure CLI** — Microsoft's install script
- **`az aks install-cli`** — installs `kubectl` *and* `kubelogin`, which we flagged in the Kubernetes session
- **`USER jenkins`** — drop back to the unprivileged user

**ASK** <br>
Each of those `RUN` commands creates a separate image layer. Why chain commands with `&&` inside one `RUN` rather than writing six separate `RUN` lines? <br>
**ANSWER** <br>
Because each `RUN` is a layer, and layers are additive — deleting a file in a later layer doesn't reclaim the space from an earlier one. If you `apt-get update` in one layer and `rm -rf /var/lib/apt/lists/*` in another, the package index is still in the image, just hidden. Chaining means the download and cleanup happen in the same layer, so the cleanup actually shrinks the result.

Build it:

*(Run from `~/toolchain-day`)*
```bash
docker build -t jenkins-devops .
```

This takes a few minutes the first time.

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

Single-line version for any shell, including PowerShell:

*(Run from `~/toolchain-day`)*
```bash
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -u root jenkins-devops
```

Every flag is revision:
- `-d` — detached, runs in the background
- `-p 8080:8080` — the Jenkins web UI
- `-p 50000:50000` — agent communication. Unused today, but standard
- `-v jenkins_home:/var/jenkins_home` — a **named volume** so config, jobs and credentials survive the container being recreated
- `-v /var/run/docker.sock:/var/run/docker.sock` — mounts the host's Docker socket so pipelines can build images
- `-u root` — sidesteps Docker permission issues. A **training shortcut**, not production practice

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
- **Install suggested plugins** — gives us Git, Pipeline and Credentials
- Create your **admin user** and write the details down
- Accept the default Jenkins URL

#### Verify the tools are actually there

This is the step that saves you an hour later.

*(Run from `~/toolchain-day`)*
```bash
docker exec jenkins terraform version
docker exec jenkins az version
docker exec jenkins kubectl version --client
docker exec jenkins docker --version
```

All four should respond. If any doesn't, fix it **now** — every later failure will trace back here.

**HANDS ON (remaining time)** <br>
*(Run from `~/toolchain-day`)*
1. Write the `Dockerfile` above and build it as `jenkins-devops`.
2. Run the container with all the flags shown, and confirm with `docker ps`.
3. Get the unlock password, complete setup in the browser, install suggested plugins, create your admin user.
4. Verify all four CLIs respond inside the container.
5. *(In the Jenkins UI)* Go to **Manage Jenkins → Credentials** and have a look around — this is where every secret goes today.
**END OF NOTE**

**Solution**

*(Run from `~/toolchain-day`)*
```bash
# 1. Build (Dockerfile contents as above)
docker build -t jenkins-devops .

# 2. Run
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -u root jenkins-devops
docker ps

# 3. Unlock
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
# ...then http://localhost:8080 in the browser

# 4. Verify — all four MUST respond
docker exec jenkins terraform version
docker exec jenkins az version
docker exec jenkins kubectl version --client
docker exec jenkins docker --version
```

Troubleshooting to have ready:

*(Run from `~/toolchain-day`)*
```bash
docker logs jenkins              # why won't it start
docker exec -it jenkins bash     # get a shell inside to poke around
docker rm -f jenkins             # remove the container — the VOLUME survives
docker volume ls                 # confirm jenkins_home still exists
```

If the build fails at the Terraform step, it's usually a transient network issue fetching HashiCorp's key — re-run the build; Docker's layer cache means it resumes rather than starting over.

<br>
<br>
### 11:15–12:15 — Authenticating to Azure & the First Terraform Stages
*(Exercise)*

#### The Service Principal

Jenkins needs to authenticate to Azure, and — as with everything automated on this course — that means a **Service Principal**, not a human login.

If you don't already have one:

*(Run from `~/toolchain-day`)*
```bash
az account show --query id -o tsv
az ad sp create-for-rbac --name "jenkins-terraform" --role="Contributor" --scopes="/subscriptions/<your-subscription-id>"
```

Returns `appId`, `password` and `tenant` — exactly as in the Terraform sessions. Copy them somewhere safe; the password is shown once.

**ASK** <br>
Back in the Azure Fundamentals session we asked why automation should have its own identity rather than running as a person. Now that a *machine* is genuinely about to use it — which of those reasons matters most? <br>
**ANSWER** <br>
All of them, but two become concrete. It **works headlessly** — there's no browser for an interactive login, and a human isn't present at 3am when the pipeline runs. And it can be **revoked independently** — if the Jenkins server is compromised, you kill that one identity without touching anyone's account or any other system. There's a third that only appears now: the credential is *scoped*, so a compromised pipeline can only damage what that Service Principal was allowed to touch. That's why `Contributor` on the whole subscription is a training shortcut you'd narrow in production.

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
Why Secret text credentials rather than just writing these four values into the `Jenkinsfile` as environment variables? <br>
**ANSWER** <br>
The same reasoning as every secret on this course, but sharper here: a `Jenkinsfile` is committed to Git, and **Git history is permanent** — deleting the line later doesn't remove it from history. The credential store keeps values out of the repo entirely, **automatically masks them** if they'd otherwise appear in console output, and lets you rotate centrally without editing code. A cloud credential with Contributor rights committed to a repository is a genuine security incident, not a tidiness problem.

#### Connect Jenkins to your repository

*(Run from `~/toolchain-day`)*
```bash
git clone https://github.com/<your-username>/<your-fork>.git
cd <your-fork>
ls
```

You should see the Terraform configuration from earlier sessions — a resource group and a storage account is plenty.

*(On GitHub — your fork)*
- **Settings → Developer settings → Personal access tokens** — generate a token with `repo` scope

*(In the Jenkins UI — Manage Jenkins → Credentials → Add Credentials)*
- **Kind**: Username with password
- **Username**: your GitHub username
- **Password**: the token
- **ID**: `github-credentials`

Now the job:

*(In the Jenkins UI — Dashboard)*
- **New Item** → name it `terraform-pipeline` → choose **Pipeline** → **OK**
- Scroll to the **Pipeline** section
- **Definition**: `Pipeline script from SCM`
- **SCM**: `Git`
- **Repository URL**: your fork's HTTPS URL
- **Credentials**: select `github-credentials`
- **Branch Specifier**: `*/main`
- **Script Path**: `Jenkinsfile`
- Scroll up to **Build Triggers** → tick **Poll SCM** → **Schedule**: `H/2 * * * *`
- **Save**

**NOTE FOR TRAINERS** <br>
Webhooks are the right answer and can't work here: GitHub is on the public internet and the student's Jenkins is on `localhost`. **Poll SCM** demonstrates the identical principle — builds trigger themselves from Git — with no networking magic. Teach the webhook as the goal, offer ngrok as a stretch, and use polling for the hands-on. <br>
**END OF NOTE**

#### The first Jenkinsfile

*(Run from `~/toolchain-day/<your-fork>`)*
- Run: `touch Jenkinsfile`
- Then open it: `code Jenkinsfile`

Start with just the credentials and a checkout:

**Jenkinsfile**
```groovy
// NEW CONFIG
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
    }
}
```

The `credentials()` helper looks up a credential by ID and injects its **value** as an environment variable at run time. The secret never appears in the file.

**ASK** <br>
Where have you seen these exact four variable names before? <br>
**ANSWER** <br>
`ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, `ARM_SUBSCRIPTION_ID`, `ARM_TENANT_ID` — you `export`ed these by hand in your terminal on the very first Terraform day. Terraform's `azurerm` provider looks for **precisely those names** automatically. We're inventing nothing: Jenkins is doing what you did with `export`, just for a machine. That's the whole trick — the pipeline isn't special, it's just a different thing typing the same commands.

#### Add the Terraform stages

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

        // NEW CONFIG
        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Terraform Validate') {
            steps {
                sh 'terraform validate'
            }
        }

        stage('Terraform Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
            }
        }
    }

    post {
        success {
            echo 'Plan generated successfully.'
        }
        failure {
            echo 'Pipeline failed — check which stage went red.'
        }
    }
}
```

Note there's **no apply yet** — deliberately. Let's get a plan running safely first.

**Before you push — the state problem.** Make sure your Terraform config has a `backend "azurerm"` block pointing at the storage account from the provisioning session:

```tf
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-backend-state-jbloggs-devops"
    storage_account_name = "stdevappsbackendjbloggs"
    container_name       = "tfstate"
    key                  = "dev/toolchain/pipeline.tfstate"
  }
}
```

**ASK** <br>
What specifically goes wrong if you push this without a backend block? <br>
**ANSWER** <br>
The build starts with an empty workspace and therefore **no state**. Terraform assumes nothing exists and plans to create everything — which either duplicates resources or fails on name collisions with things that are already there. Then the workspace is discarded and the next build repeats the same mistake. With a remote backend, the pipeline reads the same authoritative state every time and takes a **lock** while it works, so two concurrent builds can't corrupt each other.

Push it:

*(Run from `~/toolchain-day/<your-fork>`)*
```bash
git add Jenkinsfile
git commit -m "Add Jenkins pipeline for Terraform plan"
git push origin main
```

*(In the Jenkins UI — the `terraform-pipeline` job)*
- **Build Now** for the first run
- Click into the build → **Console Output**

You'll watch `terraform init` download the `azurerm` provider — the exact output you've seen dozens of times, just on Jenkins' machine instead of yours.

**HANDS ON (remaining time)** <br>
1. Create a Service Principal (or reuse yours) and store all **four** values as Secret text credentials.
2. Create a GitHub personal access token and store it as `github-credentials`.
3. Create the `terraform-pipeline` Pipeline job pointing at your fork, with **Poll SCM** on `H/2 * * * *`.
4. Add a `backend "azurerm"` block to your Terraform config.
5. Write the `Jenkinsfile` with Checkout, Init, Validate and Plan stages, and push it.
6. Get a **green** run through all four stages, and read the plan output in the console.
7. Make a trivial change (add a tag to a resource), push, and **wait** for the poll to trigger a build on its own.
**END OF NOTE**

**Solution**

The complete `Jenkinsfile` for this stage:

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
                sh 'terraform init'
            }
        }

        stage('Terraform Validate') {
            steps {
                sh 'terraform validate'
            }
        }

        stage('Terraform Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
            }
        }
    }

    post {
        success {
            echo 'Plan generated successfully.'
        }
        failure {
            echo 'Pipeline failed — check which stage went red.'
        }
    }
}
```

And the backend block in `main.tf`:

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
```

*(Run from `~/toolchain-day/<your-fork>`)*
```bash
git add Jenkinsfile main.tf
git commit -m "Add Jenkins pipeline for Terraform plan"
git push origin main

# Step 7 — trigger it without touching Jenkins
echo "  # a trivial change" >> main.tf
git add main.tf
git commit -m "Trigger the pipeline"
git push origin main
# then wait up to two minutes and watch the Dashboard
```

If **Terraform Init** fails with a backend authentication error, the Service Principal needs access to the *backend storage account's* resource group as well as wherever it's provisioning. Subscription-scoped `Contributor` covers both — which is why we used it.

<br>
<br>

### 12:15–13:00 — The Apply Gate: Saved Plans and Human Approval
*(Exercise + Challenge)*

We have a pipeline that plans. Now let's let it actually change something — carefully.

**Jenkinsfile**
```groovy
        stage('Terraform Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
            }
        }

        // NEW CONFIG
        stage('Terraform Apply') {
            steps {
                input message: 'Apply this Terraform plan?', ok: 'Apply'
                sh 'terraform apply -auto-approve tfplan'
            }
        }
```

Two things worth pausing on properly.

**`terraform plan -out=tfplan`** saves the plan to a file — you met this on the first Terraform day. In a pipeline it stops being a curiosity and becomes the entire point: **the exact plan a human reviewed is the exact plan that gets applied.** Nothing can drift between the review and the execution, because we're not re-planning, we're replaying a saved decision.

**`terraform apply -auto-approve tfplan`** — `-auto-approve` skips the interactive `yes` prompt, which is essential because there's no human at a terminal to type it. That's the same lesson as `apt-get install -y` in your provisioners: **interactive prompts are fatal in automation.**

**`input message: '...'`** pauses the pipeline and waits for a human to click **Apply** in the Jenkins UI.

**ASK** <br>
The whole point of CI/CD is removing manual steps. So why deliberately add one back, right before `apply`? <br>
**ANSWER** <br>
Because infrastructure changes can be **destructive and irreversible** in a way that application deploys usually aren't. You've seen `must be replaced` in plan output — on a storage account holding data or a database, that's catastrophic. A human reading the actual plan immediately before it executes is a cheap, valuable safety net, especially while a team is still building trust in a new pipeline. Fully unattended apply is common and legitimate — but it should be a deliberate decision to remove the gate, not something that happened by default.

**ASK** <br>
When would you actually remove it? <br>
**ANSWER** <br>
When the review has genuinely happened somewhere else — which is the PR workflow we'll cover after lunch. If a teammate already read the `plan` output on the pull request and approved the merge, then gating again on merge is just friction: you're asking someone to approve a decision that was already made. The gate should sit at the point where a human is actually making a judgement, and only there.

Push and run it:

*(Run from `~/toolchain-day/<your-fork>`)*
```bash
git add Jenkinsfile
git commit -m "Add apply stage with approval gate"
git push origin main
```

*(In the Jenkins UI — the running build)*
- Watch it reach **Terraform Apply** and pause
- Read the plan output in the Console Output above
- Click **Apply** when you're satisfied

*(In the Azure Portal — Resource groups)* — confirm the resource exists.

**Nobody ran `terraform apply`.** Sit with that for a second — that's the thing this whole course has been building towards.

**HANDS ON (15 min)** <br>
1. Add the **Terraform Apply** stage with the `input` gate and push.
2. Watch the build pause, read the plan, and approve it.
3. *(In the Azure Portal)* Verify the resource was created.
4. Make a change to your Terraform (add a tag), push, and watch the pipeline produce a *different* plan — an update rather than a create.
5. Deliberately break your Terraform (misspell an attribute), push, and confirm the pipeline **stops at Validate** and never reaches Apply.
**END OF NOTE**

**Solution**

```groovy
stage('Terraform Apply') {
    steps {
        input message: 'Apply this Terraform plan?', ok: 'Apply'
        sh 'terraform apply -auto-approve tfplan'
    }
}
```

*(Run from `~/toolchain-day/<your-fork>`)*
```bash
git add Jenkinsfile
git commit -m "Add apply stage with approval gate"
git push origin main

# Step 4 — a change that produces an UPDATE plan, not a create
# add a tags block to your resource group, then:
git add . && git commit -m "Add environment tag" && git push origin main

# Step 5 — deliberately break it
# change 'location' to 'locatoin' in a resource, then:
git add . && git commit -m "Break it on purpose" && git push origin main
```

**Step 5 answer:** the pipeline goes **Checkout ✅ → Init ✅ → Validate ❌**, and **Plan and Apply never run**. `terraform validate` exits non-zero, Jenkins sees the non-zero exit code, and stops the line. Nothing in Azure was touched. Fix the typo and push again to go green.

That's the exit-code contract doing its job — and it's why Validate earns its place as a separate stage before Plan: it's the cheapest check, so it should fail first.

---

**Challenge**

*Direct* students, **in pairs**, to harden the pipeline so that:

* The **plan output is saved as an artifact**, downloadable from the build page, so there's a permanent record of what was proposed
* The approval gate **times out after 15 minutes** rather than waiting forever, and the build is marked aborted rather than failed
* The `post` section reports the **build number and job name** on both success and failure
* A `terraform fmt -check` stage runs **before** Validate, failing the build if the code isn't formatted correctly
* **OPTIONAL** — the pipeline records **who approved** the apply, and echoes that name in the success message

*Provide* this hint as an aid: Jenkins has an `archiveArtifacts` step, and `timeout` can wrap other steps. `terraform show` can turn a saved plan file back into readable text.

*Grant* students ~10 minutes.

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
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        // Cheapest check first — fail fast on formatting
        stage('Terraform Format') {
            steps {
                sh 'terraform fmt -check -recursive'
            }
        }

        stage('Terraform Validate') {
            steps {
                sh 'terraform validate'
            }
        }

        stage('Terraform Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
                // Turn the binary plan into readable text, then keep it
                sh 'terraform show -no-color tfplan > tfplan.txt'
                archiveArtifacts artifacts: 'tfplan.txt', fingerprint: true
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
                sh 'terraform apply -auto-approve tfplan'
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

Points to draw out when revealing:

* **`terraform fmt -check`** exits non-zero if any file isn't correctly formatted — it doesn't reformat, it *reports*. Putting the cheapest, fastest check first means a trivial mistake fails in seconds rather than after a slow plan.
* **`terraform show -no-color tfplan > tfplan.txt`** converts the binary plan file into readable text — `-no-color` strips the terminal escape codes that would otherwise litter the file. Then `>` redirects it, exactly as in the bash session.
* **`archiveArtifacts`** attaches the file to the build permanently. Six months later you can answer "what exactly did we change on 14 March?" — an audit trail the Azure Activity Log genuinely cannot give you, which was the whole complaint back in the Azure Fundamentals session.
* **`timeout`** wrapping the `input` matters more than it looks: without it, an un-approved build holds a Jenkins executor open **indefinitely**, and — worse — holds the Terraform **state lock**, blocking everyone else's pipeline.
* **`submitterParameter`** captures the username of whoever clicked Apply, so the approval is attributable.

<br>
<br>

*(Take a 1 hour lunch break here.)*

<br>
<br>

### 14:00–14:45 — The Pull Request Workflow: Plan on PR, Apply on Merge
*(Talk + demo)*

We've built the mechanics. Now let's connect them to the picture we drew on day one.

*REFER TO RESOURCE 2 - SLIDEE* <br>
![toolchain-2](./resources/toolchain-2.png)

The mature real-world pattern separates **plan** from **apply** based on *what kind of Git event* triggered the pipeline:

- **Pull Request opened or updated** → run `terraform plan` **only**, and post the output as a comment on the PR for reviewers to read
- **Pull Request merged to `main`** → run `terraform apply` automatically, **no manual gate** — because the review already happened as part of PR approval

**ASK** <br>
How does that map onto the diagram from the Azure Fundamentals session? <br>
**ANSWER** <br>
It *is* that diagram. "Terraform plan posted for review before merge, apply happens automatically after merge" was a picture on a slide on day one. Today it's a thing that runs.

**ASK** <br>
Why is a `plan` posted on a pull request a genuinely better review artifact than the `.tf` diff alone? <br>
**ANSWER** <br>
Because a diff shows *what the code says*; a plan shows *what will actually happen to real infrastructure*. A one-line change can produce `1 to add, 0 to change, 1 to destroy` — and the reviewer only sees that catastrophic replacement in the plan output, not in the diff. It's the difference between reviewing intent and reviewing consequence.

#### Implementing it: the `when` directive

The mechanism is a Jenkins directive called `when`, which makes a stage **conditional**.

```groovy
stage('Terraform Plan') {
    steps {
        sh 'terraform plan -out=tfplan'
        sh 'terraform show -no-color tfplan > tfplan.txt'
        archiveArtifacts artifacts: 'tfplan.txt'
    }
}

stage('Terraform Apply') {
    // Only run this stage on the main branch
    when {
        branch 'main'
    }
    steps {
        sh 'terraform apply -auto-approve tfplan'
    }
}
```

Read it plainly: **Plan runs on every branch. Apply runs only on `main`.**

So a feature branch or PR gets a plan and an archived artifact, and nothing else. Once merged to `main`, the same pipeline runs again and this time the Apply stage is eligible.

Note the `input` gate has **gone** from Apply — because the PR approval was the gate. Gating twice asks a human to re-approve a decision already made.

**ASK** <br>
`when { branch 'main' }` only works if Jenkins knows which branch it's building. Our current job is hard-coded to `*/main`. What job type do we need? <br>
**ANSWER** <br>
A **Multibranch Pipeline** — the job type we named but didn't build in the CI/CD session. It scans the whole repository, automatically creates a sub-job for **every branch and every open pull request** containing a `Jenkinsfile`, and deletes them when the branch goes. That's what makes `when { branch ... }` meaningful, and it's the missing piece that turns our single pipeline into a real PR workflow.

#### Setting it up

*(In the Jenkins UI — Dashboard)*
- **New Item** → name it `terraform-multibranch` → choose **Multibranch Pipeline** → **OK**
- **Branch Sources** → **Add source** → **Git**
- **Project Repository**: your fork's URL
- **Credentials**: `github-credentials`
- **Scan Multibranch Pipeline Triggers** → tick **Periodically if not otherwise run** → set to **1 minute**
- **Save**

Jenkins immediately scans the repo and creates a job per branch. Now demonstrate the whole flow:

*(Run from `~/toolchain-day/<your-fork>`)*
```bash
git checkout -b add-storage-tag
# make a change to your .tf files
git add .
git commit -m "Add a tag to the storage account"
git push origin add-storage-tag
```

*(On GitHub — your fork)* — open a **Pull Request** from `add-storage-tag` into `main`.

*(In the Jenkins UI — `terraform-multibranch`)* — within a minute a job appears for the new branch, runs, produces a plan, archives it — and **skips Apply**, because the branch isn't `main`.

*(On GitHub)* — merge the PR.

*(In the Jenkins UI)* — the `main` job runs, and this time Apply **does** execute.

**NOTE FOR TRAINERS** <br>
Posting the plan **as a comment on the PR** is the one piece we're not building today — it needs either the GitHub Branch Source plugin's checks API or a small `curl` script against the GitHub API with a token. It's a great stretch exercise but a poor use of classroom time, since it's mostly API plumbing rather than a new concept. Archiving the plan as an artifact demonstrates the same idea: **the reviewer sees the consequence, not just the diff**. <br>
**END OF NOTE**

**ASK** <br>
In this model, what is now the *single* thing standing between a developer and production infrastructure? <br>
**ANSWER** <br>
**A pull request approval.** Which is exactly the point — and it's a much better control than a person carefully typing commands, because it's a decision made with full information (the plan output), attributed to a named human, recorded permanently, and made *before* anything happens rather than during. That's the whole "you build it, you run it" thread from day one, with a safety rail that scales to a whole team.

<br>
<br>

### 14:45–15:00 — Failure Modes: What Breaks and What It Costs

Before the capstone, let's think properly about what goes wrong. Designing pipelines is largely about deciding **where** you want failures to happen.

**ASK** <br>
If the `Terraform Plan` stage fails outright — a typo in a `.tf` file — does anything happen to real Azure infrastructure? <br>
**ANSWER** <br>
No. `apply` never runs. Splitting plan and apply into separate stages is a deliberate **safety boundary**, not tidiness — a broken plan stops the pipeline dead before anything real is touched. Cost of this failure: a few wasted minutes.

**ASK** <br>
What if `apply` itself fails halfway — three resources created, the fourth errors? <br>
**ANSWER** <br>
This is the interesting one, and it connects to the idempotency conversation from the automation session. Terraform's **state reflects exactly what did get created**. There's no automatic rollback — you saw this directly when three AD users couldn't all take the same name. But because state is accurate, re-running once the problem is fixed attempts **only the missing resources**, not everything again. That's why idempotency matters: a partially-failed run must be safe to retry. Cost: real resources exist in a half-built state until someone fixes it.

**ASK** <br>
Two engineers merge PRs within thirty seconds of each other, and two pipeline runs start against the same state. What happens? <br>
**ANSWER** <br>
The first run takes a **lock** on the state blob. The second finds it locked and **waits, then fails** with a clear message naming who holds the lock and since when. That's the correct behaviour — far better than both proceeding and corrupting state. It's also why the `timeout` on our approval gate mattered: a build waiting forever on approval holds that lock and blocks the whole team.

**ASK** <br>
The pipeline runs green, applies cleanly — and the application is broken. What did the pipeline get wrong? <br>
**ANSWER** <br>
Nothing, and that's the point worth landing. **A green pipeline means "the steps I was told to run succeeded", not "the system works".** If you didn't write a test, the pipeline can't run one. This is why the capstone puts a test stage *before* the build and deploy stages, and why Kubernetes readiness probes matter — the pipeline verifies the process, the probes verify the result.

**A rough hierarchy of where you want failures to land:**

| Fails at | Cost |
|---|---|
| `fmt` / `validate` | Seconds. Ideal |
| Tests | Minutes. Good — caught before anything is built |
| `plan` | Minutes. Good — nothing real touched |
| `apply` | Real resources in a partial state. Recoverable, but work |
| Runtime, after deploy | Users affected. Worst |

**Design pipelines so the cheapest checks fail first.** That's why `fmt` came before `validate`, which came before `plan`.

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>
### 15:15–16:45 — Capstone: The Complete Pipeline, Code to Running Application
*(Exercise — 1 hour 30 minutes)*

This is the end of the course, and the exercise is the whole course.

You're going to build **one pipeline** that takes a code change and, with no human typing a single command, runs it through: test → build a Docker image → push it to a registry → provision infrastructure with Terraform → deploy to Kubernetes.

Every stage uses a tool you already know. The learning is in making them cooperate.

Work individually or in pairs. Build it **stage by stage, getting each one green before adding the next** — that's not just teaching advice, it's how you'd genuinely build this at work. Do not write all six stages and then debug.

---

#### Part 1 (≈15 min) — Credentials and the app

You'll need a Docker Hub token alongside your Azure credentials.

*(On Docker Hub — [hub.docker.com](https://hub.docker.com))*
- **Account Settings → Security → New Access Token**, with **Read & Write**
- Copy it

*(In the Jenkins UI — Manage Jenkins → Credentials → Add Credentials)*
- **Kind**: Username with password
- **Username**: your Docker Hub username
- **Password**: the token
- **ID**: `dockerhub-credentials`

*(Run from `~/toolchain-day/<your-fork>`)*
```bash
ls
cat package.json
cat Dockerfile
```

You should have a small Node application with a test that passes, and a Dockerfile. (If you'd rather, use the `countries` app you scaffolded in the bash session — it already has both.)

---

#### Part 2 (≈20 min) — Test, build and push

Add three stages **before** your existing Terraform ones.

**Jenkinsfile**
```groovy
pipeline {
    agent any

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
            steps {
                checkout scm
            }
        }

        // NEW CONFIG
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

        // ... your existing Terraform stages follow ...
    }
}
```

All revision from the CI/CD session: the per-stage `docker` agent giving a clean Node environment, `$BUILD_NUMBER` as a traceable image tag, `withCredentials` scoping the secret to just those steps, and `--password-stdin` keeping the password off the command line.

Push and get these three green before continuing.

---

#### Part 3 (≈25 min) — Provision an AKS cluster with Terraform

Now Terraform provisions the thing the application will run on.

*(Run from `~/toolchain-day/<your-fork>`)*
- Run: `touch aks.tf`

**aks.tf**
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

- **`default_node_pool { }`** — the worker nodes you inspected in the Portal on the Kubernetes day, now declared as code. `node_count = 1` and a small `vm_size` keep the cost down
- **`identity { type = "SystemAssigned" }`** — gives the cluster its own **managed identity**, so it can create Azure resources (like the load balancer a `LoadBalancer` Service needs) without any credential being stored anywhere. It's a Service Principal that Azure creates and rotates for you

**ASK** <br>
On the Kubernetes day you created a cluster by clicking through the Portal. What does having it in Terraform change? <br>
**ANSWER** <br>
Everything we've argued for all course: it's reviewable in a PR, versioned, and reproducible — you can destroy the cluster and recreate an identical one from the file. Crucially it also makes the cluster **disposable**, which changes behaviour: if standing up a full environment is one pipeline run, you can afford to build one per feature branch and tear it down after. That's only possible when it's code.

**NOTE FOR TRAINERS** <br>
An AKS cluster takes **5–10 minutes** to provision. The first pipeline run through this stage will feel like it's hung. Warn students beforehand, and use the wait to talk through what the Apply stage is actually doing. Subsequent runs are fast because Terraform sees the cluster already exists. <br>
**END OF NOTE**

---

#### Part 4 (≈25 min) — Deploy to Kubernetes

Now the final stage — and this is where the tools genuinely have to cooperate.

**ASK** <br>
The pipeline just created a brand-new cluster. `kubectl` on the Jenkins agent has never heard of it. How does the deploy stage know where to connect? <br>
**ANSWER** <br>
It has to **fetch credentials at run time**, using `az aks get-credentials` — exactly the command you ran on the Kubernetes day, except the values come from Terraform's outputs rather than being copied out of the Portal. That's the integration point: Terraform knows what it built, and passes that knowledge to the next stage. Nothing about the cluster is hardcoded.

Add a `k8s/deployment.yaml` to your repo — this is your cleaned YAML from the Kubernetes session, with one change:

**k8s/deployment.yaml**
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
          image: IMAGE_PLACEHOLDER
          ports:
            - containerPort: 8080
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

Note `image: IMAGE_PLACEHOLDER` — the pipeline substitutes the real image and tag at deploy time, because the tag changes every build.

**Jenkinsfile** — add after Terraform Apply:
```groovy
        // NEW CONFIG
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Read what Terraform built
                    def clusterName = sh(
                        script: 'terraform output -raw cluster_name',
                        returnStdout: true
                    ).trim()
                    def clusterRg = sh(
                        script: 'terraform output -raw cluster_resource_group',
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
                }
            }
        }
```

Several new things, each worth explaining:

- **`terraform output -raw <name>`** prints an output value with no quotes or formatting. `returnStdout: true` captures what a shell command printed into a Groovy variable, and `.trim()` strips the trailing newline. **This is the join between Terraform and Kubernetes** — Terraform tells the next stage what it built
- **`az login --service-principal`** — a non-interactive login using the same four credentials. No browser, no human
- **`--overwrite-existing`** — replaces any stale kubeconfig entry with the same name
- **`sed -i 's|OLD|NEW|g' file`** — stream editor, editing in place. We use `|` as the delimiter instead of `/` because the image name contains slashes. `g` replaces every occurrence
- **`kubectl rollout status --timeout=180s`** — **this is the most important line in the stage.** It waits until the rolling update genuinely completes and exits non-zero if it doesn't within three minutes

**ASK** <br>
Why does that last line matter so much? Without it, `kubectl apply` succeeds and the stage goes green. <br>
**ANSWER** <br>
Because `kubectl apply` only means *"Kubernetes accepted my YAML"* — not *"the new version is running"*. Without `rollout status`, a deployment whose Pods crash on startup would still show a green pipeline, and you'd tell the whole team the release succeeded while it was silently failing. This is the exit-code contract at the very end of the chain: **the pipeline must not report success until the thing it deployed is actually working.**

---

#### Part 5 — Run the whole thing

*(Run from `~/toolchain-day/<your-fork>`)*
```bash
git add .
git commit -m "Complete pipeline: test, build, push, provision, deploy"
git push origin main
```

Then watch the Stage View march through:

**Checkout → Install & Test → Build Image → Push Image → Terraform Init → Validate → Plan → Apply → Deploy to Kubernetes**

*(Run from `~/toolchain-day/<your-fork>`)* — once it's green, find your application:
```bash
az aks get-credentials --resource-group rg-toolchain-jbloggs --name aks-toolchain-jbloggs
kubectl get service toolchain-app
```

*(In your browser)* — visit the `EXTERNAL-IP`.

**Then do the thing that proves it all works.** Change something visible in the application code, commit, push — and touch nothing else. Watch the pipeline test it, build a new image tagged with the new build number, push it, confirm the infrastructure is unchanged, and perform a rolling update on the cluster. Refresh your browser and see the change live.

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
            steps {
                checkout scm
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

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Terraform Validate') {
            steps {
                sh 'terraform validate'
            }
        }

        stage('Terraform Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
                sh 'terraform show -no-color tfplan > tfplan.txt'
                archiveArtifacts artifacts: 'tfplan.txt', fingerprint: true
            }
        }

        stage('Terraform Apply') {
            steps {
                script {
                    timeout(time: 15, unit: 'MINUTES') {
                        input message: 'Apply this Terraform plan?', ok: 'Apply'
                    }
                }
                sh 'terraform apply -auto-approve tfplan'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def clusterName = sh(
                        script: 'terraform output -raw cluster_name',
                        returnStdout: true
                    ).trim()
                    def clusterRg = sh(
                        script: 'terraform output -raw cluster_resource_group',
                        returnStdout: true
                    ).trim()

                    sh '''
                        az login --service-principal \
                          -u $ARM_CLIENT_ID \
                          -p $ARM_CLIENT_SECRET \
                          --tenant $ARM_TENANT_ID
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
| `terraform: not found` | The custom Jenkins image wasn't used, or wasn't built properly |
| `docker: permission denied` | The Docker socket wasn't mounted, or `-u root` was omitted |
| Init fails on the backend | The Service Principal can't reach the backend storage account |
| Apply hangs ~8 minutes | Normal. AKS provisioning is genuinely slow |
| `az login` fails in the deploy stage | Use `'''` triple single quotes so the shell expands `$ARM_*`, not Groovy |
| `rollout status` times out | Image architecture mismatch (arm64 image on amd64 nodes), or the container is crashing — check `kubectl describe pod` |
| `EXTERNAL-IP` stays `<pending>` | Give it two minutes; Azure is provisioning a load balancer |

**Stretch, if anyone gets there:**
- Add `when { branch 'main' }` to Apply and Deploy, and run it as a Multibranch Pipeline so PRs only plan
- Add a `post { failure { ... } }` step that runs `kubectl rollout undo deployment/toolchain-app`
- Replace the `sed` substitution with a `kubectl set image` command and consider which you prefer, and why

<br>
<br>

### 16:45–17:00 — Wrap-up, Tear Everything Down, & Course Close

#### Tear down — do this properly, and first

Today's infrastructure is the most expensive on the course.

*(Run from `~/toolchain-day/<your-fork>`)*
```bash
# Delete the Service FIRST — this releases the Azure load balancer and public IP
kubectl delete -f k8s/deployment.yaml

# Then destroy everything Terraform created
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
AKS creates a second resource group named `MC_<rg>_<cluster>_<region>` holding the node VMs, disks, load balancer and public IP. `terraform destroy` normally removes it with the cluster — but **not reliably if the Kubernetes Service wasn't deleted first**, because the load balancer it created isn't in Terraform's state. Make every student run `az group list -o table` and confirm no `MC_` group survives. Two minutes now, versus a surprise bill later. <br>
**END OF NOTE**

Finally, stop Jenkins if you want your laptop's resources back:

*(Run from `~/toolchain-day`)*
```bash
docker stop jenkins
# 'docker start jenkins' brings it back — the named volume kept everything
```

#### What you built today

One pipeline, triggered by a `git push`, that:
- ran your tests in a clean containerised environment
- built a Docker image and tagged it traceably with the build number
- pushed it to a registry
- checked, planned and applied Terraform against Azure, with state locked in a shared remote backend
- paused for a human to review the actual consequences before touching infrastructure
- provisioned a Kubernetes cluster
- fetched credentials for that cluster automatically, from Terraform's own outputs
- performed a zero-downtime rolling update
- and refused to report success until the rollout genuinely completed

**Nobody typed a command.** And every decision is recorded: who pushed, who reviewed, who approved, what the plan said, which build produced which image.

#### Where every session plugs in

- **Linux & shell scripting** — every `sh` step is bash, and the whole pipeline's safety model rests on exit codes
- **Docker** — the app image, *and* the Jenkins image itself, *and* the per-stage build environments
- **Azure Fundamentals** — Service Principals, RBAC and least privilege are what let Jenkins authenticate at all
- **Jenkins & CI/CD** — stages, credentials, gating, `post` blocks, Multibranch
- **Terraform** — unchanged `.tf` files from earlier sessions; only *who runs them* changed
- **Remote state** — the thing that made pipeline-run Terraform possible in the first place
- **Kubernetes** — declarative YAML, rolling updates, and `rollout status` as the final honesty check
- **Git** — the source of truth for application code, infrastructure code *and* the pipeline definition, and the trigger for all of it

**ASK** *(the last question of the course)* <br>
On day one we said DevOps was "you build it, you run it", and that the wall between Dev and Ops came down. Looking at what you built today — where did the wall actually go? <br>
**ANSWER** <br>
It became **code, in one repository, reviewed the same way**. The application, the infrastructure it runs on, and the process that ships it all live in the same place, go through the same pull request, and are read by the same people. There's no longer a Dev artifact thrown over to an Ops process — there's one repository and one pipeline. That's not a cultural slogan any more; you can point at the files.

And the honest caveat: **none of this is finished.** There's no monitoring, no alerting, no secrets management beyond Jenkins' store, no staging environment, no automated rollback, no cost controls. Those are the next things you'll meet in a real team. But the spine is right, and you built it.

**Q&A** — take remaining questions. Confirm one final time that `az group list -o table` is clean for everyone.

<br>
<br>

### Exercise (take-home / reinforcement)

Working individually or in pairs:

1. Rebuild the pipeline **from an empty repository**, from memory, up to a working `terraform plan`. Rebuilding is the fastest way to make it stick.
2. Convert your job to a **Multibranch Pipeline** and add `when { branch 'main' }` so pull requests only plan. Open a PR and prove Apply is skipped.
3. Add a `post { failure { ... } }` block that runs `kubectl rollout undo` so a failed deploy rolls itself back. Test it by pushing a deliberately broken image tag.
4. Add a **parameter** letting you choose the environment (`dev`/`test`), and use it in resource names and the Terraform state key.
5. Write a short paragraph explaining to a new starter why remote state is mandatory for pipeline-run Terraform.
6. **(Stretch)** Post the `terraform plan` output as a comment on the pull request, using the GitHub API and a token from the credential store.
7. **(Stretch)** Research **Azure Container Registry** and swap Docker Hub for ACR. What changes in the pipeline, and what does it buy you?

**Solution** *(for the guided ones — 2, 3, 4)*

**Take-home 2** — branch-conditional stages:
```groovy
stage('Terraform Apply') {
    when { branch 'main' }
    steps {
        sh 'terraform apply -auto-approve tfplan'
    }
}

stage('Deploy to Kubernetes') {
    when { branch 'main' }
    steps {
        // ... as before
    }
}
```
On a PR branch these show as **skipped** in the Stage View, while Plan still runs and archives its artifact.

**Take-home 3** — automatic rollback on failure:
```groovy
post {
    failure {
        echo "Build ${BUILD_NUMBER} failed — attempting rollback"
        // '|| true' so a rollback failure doesn't mask the original error
        sh 'kubectl rollout undo deployment/toolchain-app || true'
    }
}
```
The `|| true` is the operator from the bash session: if there's nothing to roll back to, we don't want that failure to obscure the real one in the logs.

**Take-home 4** — an environment parameter:
```groovy
parameters {
    choice(
        name: 'ENVIRONMENT',
        choices: ['dev', 'test'],
        description: 'Which environment to target'
    )
}

// ...then in a stage:
sh "terraform plan -out=tfplan -var='environment=${params.ENVIRONMENT}'"
```

Remember from the Terraform session that a `backend` block **cannot** be interpolated — so to vary the state key per environment you need **partial configuration**:
```groovy
sh "terraform init -backend-config='key=${params.ENVIRONMENT}/toolchain/pipeline.tfstate'"
```
Leave `key` out of the `backend` block in your `.tf` file and supply it at init time.

<br>
<br>

### Conclusion

- **Inform** students this marks the end of the DevOps Toolchain Integration session — and the end of the course
- **Recap** that every tool from the course has now been seen working together in one real, automated pipeline, and that today introduced almost no new technology: the skill was the wiring
- **Reinforce** the three ideas that carried across every session: **everything as code**, **fail cheaply and early**, and **honest exit codes** — the contract every automated system depends on
- **Confirm** every student has torn down their infrastructure and checked `az group list -o table`, including any `MC_` resource group
- **Tell** students where to go next: monitoring and observability, secrets management, multi-environment promotion, and automated rollback — the things a real team adds on top of the spine they've just built
- **Direct** students to the take-home exercises, the [Jenkins Pipeline documentation](https://www.jenkins.io/doc/book/pipeline/), and the [Terraform](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs) and [Kubernetes](https://kubernetes.io/docs/concepts/) docs

---

[Back](./README.md)

---
