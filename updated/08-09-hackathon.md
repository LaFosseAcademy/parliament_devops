# End-to-End DevOps Deployment Challenge — Trainer Script

A 2-day capstone. No new lecture content — this is where everything from Linux, Git, Docker, Terraform, Azure, Kubernetes and Jenkins gets applied, independently, to an application of the student's own choosing.

## Organisation

### How this document differs from the rest of the course

Every other script in this series is written to be **read from the front**. This one isn't, and shouldn't be. After a 20-minute kickoff you are a **facilitator**, not a lecturer.

So the recurring sections are adapted:

- **Instead of ASK/ANSWER lectures** → **DIAGNOSTIC QUESTIONS**: what to ask a stuck student to make *them* find the answer, with the answer you're steering towards
- **Instead of Solution blocks** → **UNBLOCKING SNIPPETS**: reference code to paste into Slack when someone is genuinely stuck, not a model answer. Every student's solution differs, and it should
- **Instead of HANDS ON** → **CHECKPOINTS**, with what "good" looks like and what to do about students who aren't there

Location annotations still apply — *(Run from `~/capstone/<their-repo>`)* etc. — because you'll be typing alongside students at their machines.

**`SLACK` markers** flag snippets worth having queued up to paste. **Post them reactively, not pre-emptively** — a snippet posted before someone has struggled removes the learning; posted after five minutes of genuine effort, it removes only the frustration.

### Duration & schedule

2 days, **09:00–17:00** each day.

**Day 1**

| Time | Session |
|---|---|
| 09:00–09:20 | Kickoff: the brief, checkpoints, support model |
| 09:20–10:30 | Choose the app, containerise it, get it running locally |
| **10:30–10:45** | **Break** |
| 10:45–12:30 | Repo, registry push, and infrastructure design decision |
| 12:30–13:00 | **Checkpoint 1** — trainer round |
| **13:00–14:00** | **Lunch** |
| 14:00–15:15 | Provision infrastructure with Terraform |
| **15:15–15:30** | **Break** |
| 15:30–16:30 | Deploy the container manually onto that infrastructure |
| 16:30–17:00 | **Checkpoint 2** — trainer round, and catch-up triage |

**Day 2**

| Time | Session |
|---|---|
| 09:00–09:15 | Standup: yesterday's blockers, today's plan |
| 09:15–10:30 | Pipeline stages 1–3: checkout, build, push |
| **10:30–10:45** | **Break** |
| 10:45–12:30 | Pipeline stages 4–7: terraform, gate, deploy |
| 12:30–13:00 | **Checkpoint 3** — trainer round |
| **13:00–14:00** | **Lunch** |
| 14:00–15:00 | Hardening — pick your own improvement |
| **15:00–15:15** | **Break** |
| 15:15–15:45 | Peer review, cross-paired by infrastructure choice |
| 15:45–16:00 | **Checkpoint 4** + demo prep |
| 16:00–17:00 | **Final showcase** |

### Set-up

#### For Trainers

This runs differently to the rest of the course — plan your day as a **facilitator**.

- **Prepare** the kickoff (20 min, hard limit) covering brief, checkpoints and support model. Everything after that is roaming, reviewing and unblocking
- **Schedule yourself around the four checkpoints** so you're free to check in with each student/pair as they hit them, rather than at fixed lecture times
- **Prepare a shared space** — a Slack channel, a whiteboard, a shared doc — for students to log where they're stuck. This is the single highest-value piece of prep: it lets you spot a blocker three people share and fix it once to the room instead of five times one-to-one
- **Have the example applications ready** for anyone arriving without a project
- **Have the unblocking snippets in this document queued** in a draft Slack message before Day 1 starts
- **Book the showcase properly.** It matters more than it looks; presenting their own work is a large part of the learning

**Optional Slidee slide** for the kickoff only: <br>
`Weeks X Y > CDO > End-to-End Deployment Challenge` — just the brief and checkpoint timings, nothing more.

**NOTE FOR TRAINERS — running the room** <br>
Two failure modes to watch for, and they look opposite but have the same fix. <br>
**The student who won't ask.** They'll sit quietly failing for ninety minutes. The shared "where I'm stuck" channel exists mostly for them — make posting in it explicitly normal by posting in it yourself during the kickoff. <br>
**The student who asks immediately.** They'll route every error straight to you without reading it. Answer with a diagnostic question, not a fix: "what does the error actually say?" and "which layer is that — container, network, or cloud?" Your job across two days is to make yourself progressively less necessary. <br>
**END OF NOTE**

#### For Students
- **Direct** students to the starter code for this module
  - [Student Facing Repo](https://github.com/LaFosseAcademy/cloud-devops-student-repos/tree/main)
- **Use**
  - **/end-to-end-challenge/starter-code** — contains the example applications, plus a blank project template with a suggested folder structure (not mandatory to use)
- **Make sure** everyone has, **before Day 1 starts**:
  - A GitHub account and the ability to create a new repository
  - Docker installed and running
  - A Docker Hub account (or Azure Container Registry access, if preferred)
  - An Azure subscription with a **Service Principal ready to go** from the Terraform/Azure sessions
  - A working **Jenkins instance** — their own from the Jenkins day, or a shared one you provide
  - **A chosen application** — see below

**NOTE FOR TRAINERS — pre-flight** <br>
Have students run this at 09:05 on Day 1, before the kickoff finishes. Ten minutes here saves an hour of scattered firefighting. <br>
**END OF NOTE**

*(Run from `~/`)* — **`SLACK`**: post this at 09:00 on Day 1
```bash
mkdir -p ~/capstone && cd ~/capstone

docker --version          # Docker installed and running?
docker run hello-world    # Docker actually working?
az account show           # Signed in to Azure?
terraform version         # Terraform installed?
git --version             # Git installed?
echo $ARM_CLIENT_ID       # Service Principal credentials present?
```

Anything blank or erroring gets fixed before that student starts the brief.

## Learning objectives

- **Apply** Linux, Git, Docker, Terraform, Azure and Jenkins together on one real project, independently
- **Design and justify** their own architecture and toolchain decisions, rather than following a prescribed path
- **Produce** a working automated deployment pipeline — from `git push` to a live, reachable application
- **Practise** troubleshooting across the full stack without step-by-step instructions
- **Present** and demonstrate a finished solution clearly to peers

<br>

---

## The Brief

Take an application — ideally one already packaged as, or easily packaged as, a **Docker image** — and get it running in Azure, deployed and updated entirely through an automated pipeline they build themselves.

By the end of Day 2, a student should be able to:
1. Push a code change to `main`
2. Watch a Jenkins pipeline pick it up automatically
3. See that change reflected in a live, publicly reachable application in Azure
4. Explain, to someone else, exactly how every part of that chain works

**How they get there is largely up to them.** This brief gives checkpoints and guardrails, not a script. Where earlier sessions showed one way to do something, they're free to make different choices here — that's the point.

### Choosing an application

Roughly in order of extra setup needed:

- **Bring your own** — any small-to-medium application they've already built, any language. If it isn't containerised, writing the `Dockerfile` is part of Day 1
- **`lfacademy/hello-world-rest-api`** — the image used all course. Already working, lets them focus entirely on infrastructure and pipeline
- **A starter app from the module repo** — minimal Flask/Node "to-do list" apps in `/end-to-end-challenge/starter-code/example-apps`, for a little more surface area than a hello-world endpoint without writing an app from scratch

**NOTE FOR TRAINERS** <br>
Actively push students who have *any* side project, however rough, to use it. The engagement difference between "deploying my own thing" and "deploying the same demo everyone else has" is significant — and debugging a real, slightly messy codebase is far closer to actual DevOps work than a pre-cleaned example will ever be. <br>
**END OF NOTE**

### Choosing an infrastructure target

A genuinely open decision, and part of the challenge is making it **deliberately** rather than defaulting:

- **A single Azure VM running Docker** — closest to the resource-provisioning sessions; simplest to reason about; no orchestration
- **Azure Kubernetes Service (AKS)** — more moving parts, but they know the primitives; scaling and self-healing close to free

There's no wrong answer. If they choose the VM route, they should be ready to explain **why** in the demo. *"It's simpler and this app doesn't need to scale"* is a better answer than defaulting to Kubernetes because it's the newest thing they learned.

**NOTE FOR TRAINERS** <br>
Push for a **rough split across the room**. It makes the Day 2 peer review far richer, and it means the showcase demonstrates two valid architectures rather than fifteen copies of one. If everyone gravitates to AKS, ask a couple of the stronger students to take the VM route deliberately — they'll finish sooner and have time for stretch goals. <br>
**END OF NOTE**

<br>

---

## Day 1: Foundations

### 09:00–09:20 — Kickoff
*(This is the only part of two days you deliver from the front. Keep to 20 minutes.)*

Morning. Today and tomorrow are different from everything else on this course, in one specific way: **I'm not going to teach you anything new.**

There's no new tool, no new syntax, nothing to install that you haven't installed before. Everything you need, you've already done — in isolation, with me walking you through it. The last two days changes one variable: **nobody is walking you through it.**

Here's the brief. Take an application. Get it running in Azure. Make it so that pushing a change to `main` results, automatically, in that change being live — with no human typing a command. By tomorrow afternoon you'll demo that working, to this room.

Three things about how this runs:

**One — the choices are yours.** VM or Kubernetes. Docker Hub or ACR. How many stages in your pipeline. I'll tell you if something won't work, but I won't tell you which to pick. In the showcase I'll ask *why* you chose what you chose, and "it seemed simplest for this app" is a genuinely good answer. "It's what we did in the session" is not.

**Two — being stuck is the work.** Not a sign it's going wrong. Two days of an unfamiliar problem with no instructions is much closer to the job than any session we've run. When you're stuck, post in the channel — where you are, what you expected, what actually happened. I'd rather see fifteen messages in there than fifteen people quietly failing.

**Three — there are four checkpoints.** Around midday and end of day, both days. They're not tests; they're me checking in so nobody discovers at 4pm tomorrow that something broke this morning. If you're behind at a checkpoint, that's useful information, not a problem.

**DIAGNOSTIC QUESTION** *(ask the room, take a few answers)* <br>
Before you touch anything — what's the first thing you should get working? <br>
**STEERING TOWARDS** <br>
The application running **locally**, in a container, responding to a request. Nothing else. The most common way to lose a day here is trying to debug in the cloud something that was never working on your laptop. Every layer you add — a registry, a VM, a network security group, a pipeline — is another place the failure could be. Start with zero layers and add one at a time.

Right. Choose your app, choose your target, and start. I'll come round.

<br>

### 09:20–10:30 — Containerise and Run Locally

Students confirm their application choice and, if pairing, agree who drives first.

The sequence, and they should not skip ahead:

*(Run from `~/capstone/<their-app>`)*
```bash
docker build -t myapp:local .
docker run -p 8080:8080 myapp:local
```

Then, in a second terminal:

*(Run from anywhere)*
```bash
curl localhost:8080
```

**Nothing else happens until that `curl` returns something.**

**DIAGNOSTIC QUESTIONS — when a container won't serve** <br>
Ask these in order; they map to the layers. <br>

*"Is the container actually running?"* → `docker ps`. If it's not there, `docker ps -a` will show it exited, and `docker logs <name>` will say why. Nine times in ten the app crashed on startup and the answer is sitting in those logs unread.

*"What port is your app listening on **inside** the container?"* → The commonest single error of Day 1. Their app listens on 3000, their `Dockerfile` says `EXPOSE 8080`, and they mapped `-p 8080:8080`. Get them to say the number out loud from their application code, not from memory.

*"Is it bound to localhost or to all interfaces?"* → An app listening on `127.0.0.1` inside a container is unreachable from outside it, even with correct port mapping. It must bind `0.0.0.0`. This one is genuinely non-obvious and worth explaining rather than just fixing — it catches experienced people too.

**UNBLOCKING SNIPPET** — a minimal working Node Dockerfile. **`SLACK`**: post only when someone has been stuck on their own for 10+ minutes.
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 8080
CMD ["npm", "start"]
```

And the debugging trio, worth posting to the channel early as a general resource:

**`SLACK`**
```bash
docker ps -a                    # is it running, or did it exit?
docker logs <container-name>    # why did it exit?
docker exec -it <name> sh       # get a shell inside and look around
```

<br>

*(Take a 15 minute break — 10:30.)*

<br>

### 10:45–12:30 — Repo, Registry, and the Infrastructure Decision

Three things this block:

**1. A new GitHub repository** — a fresh repo, not a fork this time. Application code and `Dockerfile` pushed.

**2. An image pushed to a registry, by hand, once.** This is deliberate — proving the whole chain works manually before automating any of it.

*(Run from `~/capstone/<their-repo>`)*
```bash
docker login
docker build -t <username>/<appname>:v1 .
docker push <username>/<appname>:v1
```

**3. The infrastructure decision.** Have them write it down — one sentence on why VM or AKS — before they start building. They'll be asked in the showcase, and a decision recorded now is a decision made rather than one drifted into.

**DIAGNOSTIC QUESTION — for anyone defaulting without thinking** <br>
*"What does your application actually need? Does it need to scale? Does it need to survive a node failing? Is there more than one of it?"* <br>
**STEERING TOWARDS** <br>
If the honest answer to all three is no, a single VM is the right call and they should say so confidently. Kubernetes has real operational cost — a cluster to maintain, more moving parts, more to go wrong. Choosing the simpler thing on purpose is a senior instinct, and this is a good moment to name it as one.

**NOTE FOR TRAINERS** <br>
Watch for Apple Silicon users here. An image built on an M-series Mac defaults to `linux/arm64` and will fail on standard Azure VMs and AKS nodes with `exec format error` — which surfaces *hours later*, at deploy time, looking like a completely unrelated problem. Catch it now. <br>
**END OF NOTE**

**UNBLOCKING SNIPPET** — **`SLACK`**: post this to the channel proactively, before lunch on Day 1.
```bash
# Building on an Apple Silicon Mac? Build for the cloud's architecture:
docker build --platform linux/amd64 -t <username>/<appname>:v1 .
```

<br>

### 12:30–13:00 — ✅ Checkpoint 1

Go round every student or pair. Looking for:

- [ ] A GitHub repo containing the application and a working `Dockerfile`
- [ ] An image built from that `Dockerfile`, pushed to Docker Hub or ACR
- [ ] The app running locally via `docker run`, responding to a request
- [ ] A one-sentence, written-down justification for their infrastructure choice

**What "behind" looks like here, and what to do:**

| Situation | Action |
|---|---|
| App not containerised yet | Switch them to `lfacademy/hello-world-rest-api` now. Losing the afternoon to a Dockerfile costs them the whole capstone |
| Container runs, not reachable | Work the layer questions above with them. 10 minutes, not 40 |
| No registry push yet | Lower priority — it can happen over lunch. Don't block them |
| Nothing working at all | Pair them with someone who's ahead for the afternoon. Better a strong pair than a lost individual |

<br>

*(Lunch — 13:00–14:00.)*

<br>

### 14:00–15:15 — Provision Infrastructure with Terraform

Using Terraform, students provision whatever their chosen target needs:

- **VM route**: Resource Group, VNet/Subnet, Network Security Group (with **the ports their app actually needs**), Public IP, NIC, and the VM
- **AKS route**: Resource Group and an AKS cluster

`terraform plan` and `terraform apply` run **from their own machine** today. We are not automating this yet — we're proving it works.

**DIAGNOSTIC QUESTIONS — Terraform** <br>

*"Where's your state?"* → If they've set up a remote backend, good, and it'll save them tomorrow. If it's local, that's fine for today but flag it now: **tomorrow's pipeline will not work with local state.** Better they hear it now than discover it at 11am on Day 2.

*"Which port did you open in the NSG, and which port is your app on?"* → Same mismatch as this morning, one layer up. Get them to state both numbers aloud.

*"What's your plan actually saying?"* → For anyone applying without reading. `1 to add, 0 to change, 1 to destroy` on day two of a capstone is worth noticing before pressing yes.

**NOTE FOR TRAINERS** <br>
An AKS cluster takes **5–10 minutes** to provision, and the first student to hit it will think it's hung. Announce it to the room the moment the first AKS `apply` starts. Tell them to use the wait to start writing tomorrow's `Jenkinsfile` skeleton rather than watching the terminal. <br>
**END OF NOTE**

**UNBLOCKING SNIPPET** — minimal AKS cluster. **`SLACK`**: for students who've chosen AKS and are reconstructing from the Kubernetes session.
```tf
resource "azurerm_resource_group" "rg" {
  name     = "rg-capstone-<yourname>"
  location = "uksouth"
}

resource "azurerm_kubernetes_cluster" "aks" {
  name                = "aks-capstone-<yourname>"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  dns_prefix          = "capstone"

  default_node_pool {
    name       = "default"
    node_count = 1
    vm_size    = "Standard_B2s"
  }

  identity {
    type = "SystemAssigned"
  }
}

output "cluster_name" {
  value = azurerm_kubernetes_cluster.aks.name
}

output "cluster_resource_group" {
  value = azurerm_resource_group.rg.name
}
```

Point out that those two outputs exist because **tomorrow's pipeline will need them** — that's the join between Terraform and the deploy stage.

<br>

*(Take a 15 minute break — 15:15.)*

<br>

### 15:30–16:30 — Deploy the Container Manually

Infrastructure exists. Now get the container running on it **by hand**.

**VM route** —

*(Run from `~/capstone/<their-repo>`)*
```bash
ssh -i ~/azure/azure_ssh_keys/<key>.pem azureuser@<public-ip>
```

*(Then, on the VM)*
```bash
sudo apt-get update -y
sudo apt-get install docker.io -y
sudo docker run -d -p 80:8080 <username>/<appname>:v1
sudo docker ps
```

**AKS route** —

*(Run from `~/capstone/<their-repo>`)*
```bash
az aks get-credentials --resource-group <rg> --name <cluster>
kubectl create deployment myapp --image=<username>/<appname>:v1
kubectl expose deployment myapp --type=LoadBalancer --port=80 --target-port=8080
kubectl get service myapp
```

**DIAGNOSTIC QUESTIONS — "it's not reachable"** <br>
This is the big one of Day 1. **Work the layers in order, smallest first** — the same layered approach from the Linux and Automation session. Don't let them skip to the middle. <br>

1. *"Is the container running?"* → `docker ps` / `kubectl get pods`. If `CrashLoopBackOff` or `Exited`, stop here and read the logs
2. *"Is it listening on the port you think?"* → `docker logs` / `kubectl logs`
3. *"Is the port mapping right?"* → VM: is it `-p 80:8080` and not `-p 8080:8080`? AKS: is `targetPort` the container's port and `port` the external one?
4. *"Does the NSG/Service allow that traffic from the internet?"* → Check the actual rule, in the Portal, not from memory
5. *"Have you waited?"* → An AKS `EXTERNAL-IP` sits at `<pending>` for a couple of minutes. Several students will "fix" a problem that didn't exist

**UNBLOCKING SNIPPET** — the layer-by-layer checklist. **`SLACK`**: pin this in the channel; it's the most reused thing across both days.
```bash
# 1. Is it running?
docker ps                 # VM
kubectl get pods          # AKS

# 2. If not, why did it stop?
docker logs <name>
kubectl logs <pod-name>
kubectl describe pod <pod-name>

# 3. What port is it actually on?
docker port <name>
kubectl get service myapp

# 4. Can you reach it from the machine itself?
curl localhost:80         # on the VM, over SSH
# If this works but the public IP doesn't, it's a NETWORK problem, not an app problem.
```

That last line is the one worth saying out loud to the room: **if `curl localhost` works on the box but the public IP doesn't, you've narrowed it to networking.** That single test halves the search space.

<br>

### 16:30–17:00 — ✅ Checkpoint 2 & Catch-up Triage

- [ ] `terraform apply` has run successfully and the infrastructure exists in Azure
- [ ] `.tf` files committed to their GitHub repo
- [ ] The application is reachable at a public IP or URL, deployed manually
- [ ] They can explain, **out loud**, every resource their Terraform config creates and why it's needed

That last one is not a formality — ask it properly. A student who copied an NSG block from the session notes without understanding it will be unable to debug it tomorrow when the pipeline touches it.

**DIAGNOSTIC QUESTION — the checkpoint question worth asking everyone** <br>
*"If I deleted all of this right now, how long would it take you to get it back?"* <br>
**STEERING TOWARDS** <br>
If their answer involves any manual step they'd have to remember, that's precisely what tomorrow automates. It's a good frame for Day 2: everything they did by hand today is a stage in tomorrow's pipeline.

**NOTE FOR TRAINERS — this is the important half hour of Day 1** <br>
Anyone significantly behind gets pulled aside **now** for a short catch-up, rather than starting Day 2 already lost. Twenty minutes here is worth far more than a whole day of them stuck tomorrow. <br>
Concretely: if someone has no working deployment by 17:00, get them onto `lfacademy/hello-world-rest-api` and the simplest possible VM setup overnight or first thing. The Day 2 learning is the **pipeline** — losing that because Day 1's app never containerised is the worst outcome available. <br>
**END OF NOTE**

**Costs — say this before anyone leaves.** VMs and AKS clusters charge overnight. Either they leave it running deliberately (fine — it's needed tomorrow) or they destroy and rebuild in the morning. What's not fine is not knowing which. Have everyone run:

*(Run from `~/capstone/<their-repo>`)* — **`SLACK`**
```bash
az resource list -o table
```

<br>

---

## Day 2: Automate, Harden, and Demo

### 09:00–09:15 — Standup

Short and structured. Round the room, thirty seconds each: **where you got to, what's blocking you, what you're doing first today.**

Two purposes. It surfaces overnight blockers before they cost an hour, and it lets the room hear that other people are stuck too — which materially changes whether people ask for help.

Then set the day: **the pipeline is the whole point of today.** Everything from yesterday becomes a stage.

**DIAGNOSTIC QUESTION** *(to the room)* <br>
Yesterday you deployed by hand. List the commands you typed, in order. <br>
**STEERING TOWARDS** <br>
Build the image, push the image, run terraform, deploy the container. **That's the pipeline.** There's nothing else to design — the stages are just yesterday's commands, in order, with the exit codes taken seriously. Framing it this way removes most of the intimidation.

<br>

### 09:15–10:30 — Pipeline Stages 1–3: Checkout, Build, Push

The `Jenkinsfile`, committed to their repo. Built **one stage at a time**, each passing before the next is added.

Say this explicitly and repeat it: *a pipeline with five stages you built and tested one at a time is trivially debuggable; five stages written at once and pushed is not.*

Stage order for this block:
1. Checkout
2. Build the Docker image
3. Push it, tagged with `$BUILD_NUMBER`

Plus the GitHub webhook (or Poll SCM) so it triggers on push rather than **Build Now**.

**DIAGNOSTIC QUESTIONS — pipeline** <br>

*"Does your Jenkins have the tools it needs?"* → The single biggest time sink of Day 2. Stock `jenkins/jenkins:lts` has no Terraform, no Azure CLI, no kubectl, and no Docker CLI. Every `command not found` traces here.

*"Where are your credentials?"* → If any secret is in the `Jenkinsfile`, stop them immediately. It's a success criterion, and Git history is permanent.

*"Is that stage green because it worked, or because it didn't check?"* → For the student whose build stage passes suspiciously fast.

**UNBLOCKING SNIPPET** — a Jenkins image with everything. **`SLACK`**: post proactively at 09:15 on Day 2. Nobody should lose an hour to this.
```dockerfile
FROM jenkins/jenkins:lts-jdk17
USER root
RUN apt-get update && apt-get install -y curl unzip gnupg lsb-release docker.io \
    && rm -rf /var/lib/apt/lists/*
RUN curl -fsSL https://apt.releases.hashicorp.com/gpg \
      | gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" \
      > /etc/apt/sources.list.d/hashicorp.list \
    && apt-get update && apt-get install -y terraform && rm -rf /var/lib/apt/lists/*
RUN curl -sL https://aka.ms/InstallAzureCLIDeb | bash
RUN az aks install-cli
USER jenkins
```

**`SLACK`** — and the run command plus verification:
```bash
docker build -t jenkins-devops .
docker rm -f jenkins
docker run -d --name jenkins -p 8080:8080 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -u root jenkins-devops

# Verify BEFORE writing any pipeline stages:
docker exec jenkins terraform version
docker exec jenkins az version
docker exec jenkins kubectl version --client
docker exec jenkins docker --version
```

<br>

*(Take a 15 minute break — 10:30.)*

<br>

### 10:45–12:30 — Pipeline Stages 4–7: Terraform, Gate, Deploy

Continuing one stage at a time:

4. `terraform init` and `terraform plan`
5. An **approval gate** (`input`) before applying
6. `terraform apply`
7. **Deploy the new image version**

Stage 7 is the interesting one, and it's deliberately open — it looks different depending on their infrastructure choice. Make them think it through rather than handing it over:

- **VM route**: SSH in and re-run the container with the new tag? A `remote-exec`? A restart policy?
- **AKS route**: `kubectl set image`? `sed` the tag into a YAML file and `kubectl apply`?

**DIAGNOSTIC QUESTIONS — the hard ones of Day 2** <br>

*"Where is your Terraform state?"* → **If it's local, this pipeline cannot work.** The Jenkins workspace starts empty every build, so Terraform believes nothing exists and plans to create everything. This is the single most common Day 2 wall, and it's the one worth letting them hit for five minutes and then explaining — the penny drops much harder that way than being told pre-emptively.

*"How does the deploy stage know where to deploy to?"* → For AKS students. The cluster's name and resource group have to come from somewhere. Steer them to `terraform output` rather than hardcoding.

*"Does your pipeline actually know if the deploy worked?"* → `kubectl apply` succeeding means "Kubernetes accepted the YAML", not "the new version is running". `kubectl rollout status` is the honest check.

**UNBLOCKING SNIPPET** — remote backend. **`SLACK`**: post reactively, the moment the first person hits the state problem.
```tf
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-backend-state-<yourname>"
    storage_account_name = "st<yourname>backend"
    container_name       = "tfstate"
    key                  = "capstone/terraform.tfstate"
  }
}
```
```bash
terraform init -reconfigure    # then 'yes' to copy existing state up
```

**UNBLOCKING SNIPPET** — the AKS deploy stage. **`SLACK`**: reactive only; make them attempt it first.
```groovy
stage('Deploy to Kubernetes') {
    steps {
        script {
            def clusterName = sh(script: 'terraform output -raw cluster_name', returnStdout: true).trim()
            def clusterRg   = sh(script: 'terraform output -raw cluster_resource_group', returnStdout: true).trim()

            sh '''
                az login --service-principal \
                  -u $ARM_CLIENT_ID -p $ARM_CLIENT_SECRET --tenant $ARM_TENANT_ID
            '''
            sh "az aks get-credentials --resource-group ${clusterRg} --name ${clusterName} --overwrite-existing"
            sh "kubectl set image deployment/myapp myapp=${IMAGE_NAME}:${IMAGE_TAG}"
            sh "kubectl rollout status deployment/myapp --timeout=180s"
        }
    }
}
```

<br>

### 12:30–13:00 — ✅ Checkpoint 3

- [ ] A full pipeline run, triggered by a **real `git push`**, not a manual build
- [ ] Every stage passing, including the approval gate and `terraform apply`
- [ ] The new version of the application actually live and reachable after the pipeline finishes

**The checkpoint question:** *"Change something visible in your app, push it, and let me watch."* Nothing else demonstrates it as convincingly, and it's a dry run for the showcase.

**Triage for anyone not there:**

| Situation | Action |
|---|---|
| Pipeline works on **Build Now** but not on push | Trigger config. Poll SCM (`H/2 * * * *`) is the reliable fallback for local Jenkins |
| Stuck on the state problem | Give them the backend snippet. Don't let this eat their afternoon |
| Terraform stages green, deploy stage failing | Most likely image architecture or a hardcoded cluster reference |
| Nowhere near | Drop the approval gate and the Terraform stages. **Get build → push → deploy working.** A three-stage pipeline they understand beats a seven-stage one they don't |

<br>

*(Lunch — 13:00–14:00.)*

<br>

### 14:00–15:00 — Hardening

Core loop working. Now they pick **one or two** improvements — not all of them. Depth over coverage.

- **Resilience** — what happens if the container crashes? VM: a `--restart` policy on `docker run`. AKS: close to free, but **confirm it** by killing a pod and watching it come back
- **Config/secrets hygiene** — is anything sensitive hardcoded in the `Jenkinsfile`, a `.tf` file, or the app itself? Go and look properly
- **A second environment** — could the Terraform be parameterised with an `environment` variable to stand up a separate `staging` copy?
- **Observability** — can they tell from *outside* whether the app is healthy right now? A health-check endpoint counts; so does `kubectl get pods` as a manual check

**DIAGNOSTIC QUESTION — for anyone who says "it's fine"** <br>
*"Show me. Kill the container and let's watch."* <br>
**STEERING TOWARDS** <br>
Resilience that hasn't been tested isn't resilience, it's an assumption. This is the same instinct as the deliberately-broken-image demo in the Kubernetes session: **prove the safety net catches you, don't assume it.** It's usually the most memorable five minutes of the afternoon.

<br>

*(Take a 15 minute break — 15:00.)*

<br>

### 15:15–15:45 — Peer Review

Pair students with someone who chose a **different infrastructure target** (VM ↔ AKS wherever possible). 15 minutes each way.

They're not looking for perfection. Two things only:
1. One thing they'd have done differently
2. One thing they're going to steal for their own next project

**NOTE FOR TRAINERS** <br>
Enforce the cross-pairing. A VM student reading an AKS pipeline learns more in fifteen minutes about the trade-off than any amount of you explaining it — they see the extra complexity *and* what it bought, on real code, side by side. Same-target pairs mostly just confirm each other's choices. <br>
**END OF NOTE**

<br>

### 15:45–16:00 — ✅ Checkpoint 4 & Demo Prep

- [ ] At least one hardening improvement made and **demonstrated**
- [ ] Peer review completed both ways
- [ ] A one-paragraph explanation ready of the single hardest problem solved across the two days

Then five minutes of demo prep. Two practical warnings worth giving:

- **Have your change ready to push before you present.** Don't write it live in front of the room
- **Know how long your pipeline takes.** If it's four minutes, talk through the architecture while it runs rather than watching a progress bar in silence

<br>

### 16:00–17:00 — Final Showcase

Each student or pair gets **5 minutes**:

1. **30-second architecture summary** — what's running, where
2. **A live demo** — make a small change, push it, let the group watch the pipeline run
3. **One thing that went wrong**, and how they fixed it

**NOTE FOR TRAINERS** <br>
The "what went wrong" part is deliberately in the brief, not optional colour. It normalises debugging as core expected work rather than personal failure, and it's frequently the most useful five minutes for the room — chances are several people hit the identical issue and thought they were alone in it. <br>
If someone's demo fails live, that's a *gift*, not a disaster. Let them debug it in front of everyone for two minutes. It's the most realistic thing that will happen across two days, and how you react sets whether the room reads failure as normal or shameful. <br>
**END OF NOTE**

**Questions worth asking each presenter** — pick one, keep it light:
- *"Why did you choose VM/AKS?"* — the deliberate-decision question from the kickoff, closed off
- *"What would break first if this got 100× the traffic?"*
- *"What's still manual that you'd automate next?"*
- *"If you started again on Monday, what would you do differently?"*

<br>

---

## Success Criteria

| Requirement | Evidence |
|---|---|
| Application is containerised | `Dockerfile` in the repo, image pushed to a registry |
| Infrastructure is defined as code | `.tf` files in the repo, `terraform apply` runs cleanly |
| Deployment is automated | `Jenkinsfile` in the repo, triggered by a real `git push` |
| Secrets are handled properly | No credentials committed to Git — Jenkins Credentials store used throughout |
| A human reviews before infrastructure changes apply | A working `input`/approval gate in the pipeline |
| The application is genuinely reachable | A live public IP or URL, demoed working on the day |
| They can explain their own architecture | Verbal walkthrough during the showcase, without notes |

Not a pass/fail checklist marked in isolation — it's what you're looking for while checking in at each checkpoint, and what students should be able to point to by the showcase.

<br>

---

## Stretch Goals

For anyone finishing early or wanting to push further:

- **Blue/green or canary deployment** — route traffic to a new version gradually rather than all at once
- **Split `plan`/`apply` on PRs vs merges** — `terraform plan` on a pull request, `apply` only on merge to `main`. Needs a Multibranch Pipeline and `when { branch 'main' }`
- **Real automated tests as a pipeline stage** — an actual test suite, gating the build before it ever reaches Build Image
- **A custom domain with HTTPS** — point a domain at the app and get a TLS certificate issued
- **Rebuild on the other infrastructure target** — VM → AKS or the reverse. One of the best ways to feel the trade-offs rather than be told them
- **Monitoring/alerting** — Azure Monitor, or a scheduled health-check script via `cron` that alerts if the app stops responding
- **Automated rollback** — a `post { failure { ... } }` block that reverts a bad deploy

<br>

---

## Trainer's Quick Triage Reference

The failures you will actually see, in roughly the order you'll see them. Worth having open on a second screen across both days.

| Symptom | Almost always |
|---|---|
| `docker run` works, `curl localhost` doesn't | App bound to `127.0.0.1` instead of `0.0.0.0`, or a port mismatch |
| Container exits immediately | Read `docker logs`. It's in there |
| `exec format error` on the VM/cluster | arm64 image on amd64. Rebuild with `--platform linux/amd64` |
| App unreachable at the public IP | NSG rule, or wrong port mapping. `curl localhost` **on the box** splits app problems from network problems |
| AKS `EXTERNAL-IP` stuck at `<pending>` | Wait two minutes. It's usually fine |
| `terraform: not found` in Jenkins | Stock Jenkins image. Needs the custom build |
| `docker: permission denied` in Jenkins | Socket not mounted, or `-u root` omitted |
| Pipeline plans to recreate everything | **Local state.** Needs a remote backend |
| Pipeline green on Build Now, not on push | Webhook can't reach localhost. Use Poll SCM |
| Deploy stage can't find the cluster | Hardcoded name, or missing `terraform output` |
| `kubectl apply` green but app broken | No `rollout status`. The pipeline never checked |
| Terraform state locked | A previous build still waiting at the approval gate |

<br>

---

## Conclusion

- **Inform** students this marks the end of the taught programme's core capstone
- **Run** the showcase as a group activity, not a private trainer review — seeing several valid approaches to the same brief is a large part of the value
- **Collect** a short written retrospective from each student/pair: what they'd do differently, and what they're proudest of
- **Confirm** everyone has torn down or deliberately decided to keep their infrastructure — run `az resource list -o table` one final time as a room
- **Direct** students towards the stretch goals as self-directed follow-up if they want to keep building after the course ends

---

[Back](./README.md)

---
