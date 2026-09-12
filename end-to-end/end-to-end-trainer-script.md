# End-to-End Project — Trainer Script

Three hours taking trainees from an empty folder to a two-container application running on Azure infrastructure, built and deployed by a pipeline they wrote.

Written as a script — read it out, adapt where it feels natural, but the intent is that you could deliver this cold from the page.

---

## Organisation

### The shape of the session

**We build the pipeline incrementally, and get it green at every step.**

The scaffold script generates a **stub `Jenkinsfile` with four empty stages** — `Docker Login`, `Build Docker Images`, `Push Docker Images`, `Terraform`. We fill those in one at a time and prove each one before moving on. By the end students have seen **five successful builds**, not one reveal.

The application grows the same way. The script is **generic** — run it with `books Book` and you get a books API. What comes out is correctly wired but meaningless: two example rows and a model that knows nothing. Over the session that becomes a planets API.

### Duration & schedule

| Time | Section | Activity |
|---|---|---|
| 00:00–00:10 | Kickoff | |
| 00:10–00:35 | Permissions — what automation needs, and why | **25 min** |
| 00:35–01:00 | The scaffold script — read it, then run it | **25 min** |
| 01:00–01:25 | A Jenkins that can do the job | **25 min** |
| 01:25–01:50 | **Pipeline v1** — fill in the stubs. Get it green | **25 min** |
| **01:50–02:05** | **Break** | |
| 02:05–02:25 | Make it a planets app | **20 min** |
| 02:25–02:45 | **Pipeline v2** — somewhere to keep state | **20 min** |
| 02:45–03:10 | **Pipeline v3** — plan, then apply | **25 min** |
| 03:10–03:30 | cloud-init, verify, tear down | **20 min** |

**NOTE FOR TRAINERS — where to cut if you're behind** <br>
**Make it a planets app** compresses safely — hand out the two files, let them paste and push. The learning there is application code they already have. <br>
**Pipeline v1 must not be cut.** If they leave without one green build, they leave with nothing. <br>
**END OF NOTE**

### Set-up

Before the session every student needs:

- **Docker Desktop** running
- **Node 20+**, **Git**, the **Azure CLI**, and the **GitHub CLI** (`gh`)
- A **GitHub** account, a **Docker Hub** account, an **Azure** account with credit
- The **scaffold script** from the starter repo

**NOTE FOR TRAINERS — check `gh` before the session** <br>
The script calls `gh repo create`. If `gh` isn't installed, or the student isn't authenticated, the script fails **after** generating all the files — leaving them halfway. <br>
`brew install gh` on Mac, `winget install GitHub.cli` on Windows. Put it in the pre-session email and check it in the first five minutes. <br>
**END OF NOTE**

---
---

## 00:00–00:10 — Kickoff

### Pre Delete
- Azure:
  - SSH Keys
  - EntraId Service Principal Apps
  - `~/azure` directory
- Docker:
  - (All images, containers, volumes)
  - Personal Access Token
- Repo planetary-app if exists
- Check your GitHub Username and Docker username are the same


Morning. Three hours, and by the end you'll have an application running on a virtual machine in Azure that you never once logged into, built by a pipeline you never once triggered by hand. That's the plan. 

Here's the shape of it.

```
git push ──▶ Jenkins ──▶ build two images
                     ──▶ push to Docker Hub
                     ──▶ terraform plan
                     ──▶ [ you click Apply ]
                     ──▶ terraform apply ──▶ Azure
                                                │
                                                ▼
                                          VM builds itself
                                          pulls your images
                                          starts them
```

**Two things about how we're doing this.**

**First — we build it in layers, and get it green at every layer.** The script you're about to run generates a `Jenkinsfile` with four empty stages in it. We fill them in one at a time. You'll have a pipeline that genuinely builds and publishes images about an hour from now, long before we touch Azure.

That's not caution for your benefit — it's how you'd do it at work. A pipeline with five stages added one at a time is debuggable. Five stages written at once and pushed is forty minutes of guessing which one broke.

**Second — the app starts generic and becomes specific.** The script takes a resource name as an argument. Run it with `books Book` and you'd get a books API. What comes out is a correct skeleton with two example rows in the database and a model that knows nothing about anything. We'll turn it into planets after the break.

**ASK** <br>
Should we try and get the application working in the cloud or locally firsT?? <br>
**ANSWER** <br>
**Locally** Not the pipeline or Azure. We'll start with a container on our own machines that responds to a request. <br>
The most reliable way to lose a session like this is debugging something in the cloud that was never working on your laptop. Every layer you add — a second container, a registry, a pipeline, a VM — is another place the failure could be. **Start with zero layers and add one at a time.**

Right. Before we do that though... let's think wholistically about what we're going to try and do and the permissions we'll need. 

---
---

## 00:10–00:35 — Permissions: what automation needs, and why

*(Activity: 25 min)*

We need four credentials today. I want to do the **why** first, in broad strokes, because otherwise this is twenty-five minutes of clicking with no thread through it.

### Three different jobs, three different identities

**Something is going to push Docker images.** Today that's the script first, then Jenkins. Docker Hub needs to know it's us. So we need credentials for Docker. 

**Something is going to create infrastructure in Azure.** That's Terraform, running inside Jenkins. Azure needs to know it's us. Think about how we can authenticate ourselves there. 

**We'll want to log into the virtual machine afterwards to see what happened.** That's a different thing entirely — that's **you personally**, not a pipeline. So we need an SSH key.

**And the script is going to create a GitHub repository for us**, which needs GitHub to know it's us.

Three machines and one human, and **none of them uses your password.**

**ASK** <br>
Why not? It would be simpler — we already have passwords. <br>
**ANSWER** <br>
Four reasons, and they compound: <br>
**It works headlessly.** There's no browser for an interactive login and no human at 3am to approve an Multi Factor Authentication prompt. <br>
**It's scoped.** A token that can push to one registry can't read your email. <br>
**It's revocable independently.** If Jenkins is compromised you kill one credential rather than changing your password and locking yourself out of everything. <br>
**It's attributable.** By which I mean, when something creates a resource at 2am, you can tell whether it was a person or a pipeline.


We've seen this before regarding **the Service Principal in the Azure session.** Same reasoning exactly — today is where a machine genuinely uses it for the first time. <br>


### Why an SSH key, specifically

**ASK** <br>
Azure would let us set a password on a VM. Why insist on a key pair? <br>
**ANSWER** <br>
A password can be guessed, and a public SSH port gets probed by bots within minutes of existing. A key can't realistically be brute-forced. <br>
But the more interesting reason is the **asymmetry**. There are two halves: <br>
The **public** key goes on the server and is designed to be published — commit it, email it, put it on a poster. <br>
The **private** key never leaves your machine. <br>
So you can hand the public half to Azure, to a colleague, to a repository, with **no risk at all**. That's a property a password simply doesn't have — and it's why later today we can commit one to a public repo without flinching.

### HANDS ON (25 min)

**1. Docker Hub access token** *(6 min)*

*(In your browser — [hub.docker.com](https://hub.docker.com))*
- **Account Settings → Personal Access Token → Generate Access Token**
- Description: `planetary-app`
- Permissions: **Read, Write & Delete**
- **Generate**, then **copy it now** — it's shown once

I'm going to create a folder: `~/Desktop/planetary-app`

- *(Run from `~/Desktop/planetary-app`: `touch credentials.md`)* <br>
I'll paste the password into there. 

**credentials.md**
```md
Docker PAT - <personal-access-token>
```

*(Run from anywhere)*
```bash
docker login -u <your-dockerhub-username>
```
Paste the token as the password.

**2. GitHub CLI** *(3 min)*

Let's authenticate ourselves to GitHub via the CLI.

```bash
gh auth login
```
Choose **GitHub.com → HTTPS → Yes → Login with a browser**, and follow the prompts.

```bash
gh auth status
```

**credentials.md**
```md
Docker PAT - <personal-access-token>
GH CLI Authorised
```

**3. An SSH key pair** *(8 min)*

*(In the Azure Portal)*
- Search **SSH keys** → **Create**
- **Resource group**: create `rg-ssh-keys`
- **Key pair name**: `default-vm-ssh`
- **Generate new key pair** → **Review + create** → **Create**
- Your browser downloads `default-vm-ssh.pem`

- *Download it to `~/Desktop/planetary-app`

*(Run from `~/Desktop/planetary-app`)* <br>
- `ls -l`

**ASK** <br>
I only need permission to read this key to, how do I check what permissions I currently have? <br>
**ANSWER** <br>
`ls -l`


*(Run from `~/Desktop/planetary-app`)*
```bash
mkdir -p ~/azure/azure_ssh_keys
chmod 400 default-vm-ssh.pem
ls -l default-vm-ssh.pem
mv default-vm-ssh.pem ~/azure/azure_ssh_keys/
```

We've only been given the private, which is what stays with us. We need to generate a public key which gets added to the Virtual Machines as we create them. 

*(Run from `~/azure/azure_ssh_keys`)*
```bash
ssh-keygen -y -f default-vm-ssh.pem > default-vm-ssh.pub
ls -l
```

**credentials.md**
```md
Docker PAT - <personal-access-token>
GH CLI Authorised
Public & Private Key Created
```

**4. A Service Principal** *(8 min)*

```bash
az login
az account show --query id -o tsv
```
**credentials.md**
```md
Docker PAT - <personal-access-token>
GH CLI Authorised
Public & Private Key Created
Azure Subscription ID - <subscription-id>
```

```bash
az ad sp create-for-rbac \
  --name "terraform-planetary-app" \
  --role "Contributor" \
  --scopes "/subscriptions/<your-subscription-id>"
```

**Keep the output open.** The password is shown once.

**credentials.md**
```md
Docker PAT - <personal-access-token>
GH CLI Authorised
Public & Private Key Created
Azure Subscription ID - <subscription-id>
Azure SP: appId - <app-id>
Azure SP: displayName - <display-name>
Azure SP: password - <password>
Azure SP: tenant - <tenant>
```

**ASK** <br>
`--role "Contributor"`. Why not `Owner`? <br>
**ANSWER** <br>
`Contributor` can create and delete resources. **`Owner` can additionally grant permissions to others** — so a compromised pipeline could create new identities and persist after you'd locked it out. <br>
Least privilege: what the job needs and nothing more. We're also scoping to one subscription rather than the whole tenant.

---
---

## 00:35–01:00 — The scaffold script

*(Activity: 25 min)*

Back in the bash session we wrote a script that scaffolds an application. This is a grown-up version of it, and it does a little more than ours did before.

I want you to **read it properly before you run it**, because everything in it is something you'll recognise — and the parts you recognise are the parts you'll be modifying all afternoon.

I'm going to share the script with you over Slack. 

### Make it executable

*(Run from `~/bin`)*
- `touch scaffold-extended`
- `chmod 700 scaffold-extended`
- Copy script inside
- Make it executable from anywhere
  - `export PATH="$HOME/bin:$PATH"`



### HANDS ON (10 min) — read it

**Read the whole script. Don't run it yet.** I'd like you to be able to find and point out:

1. How many **arguments** it takes, and what happens if you leave one out
2. The line that makes `$2` **optional**, and what it defaults to
3. A **quoted** heredoc and an **unquoted** one — and why they differ
4. The place it creates a **GitHub repository**
5. The place it **builds and pushes Docker images** — and why there are four builds but only two pushes
6. Anything you don't recognise — write it down

**END OF NOTE**


### Talking through what they found

**Three arguments, and two guards:**

```bash
resource="$1"
model="${2:-${resource^}}"
username="$3"
```

**ASK** <br>
`${2:-${resource^}}` — unpack that for me. <br>
**ANSWER** <br>
Two things at once. `${2:-something}` means **"use `$2`, but if it's missing, use `something` instead"**. And `${resource^}` **capitalises the first letter**. <br>
So `scaffold planets` gives you a model called `Planets`, while `scaffold planets Planet` gives you exactly `Planet`. It's a default parameter, like you'd write in a JavaScript function.


**The guard clauses:**

```bash
if [ -z "$resource" ]; then
  echo "Usage: scaffold <resource> [ModelName] <github/docker-username>" >&2
  exit 1
fi
```

**ASK** <br>
Why `>&2` rather than plain `echo`? <br>
**ANSWER** <br>
Every program has **two** output channels — `1` for results and `2` for complaints. `>&2` sends the  message to standard error. <br>
That matters the moment something consumes your output. `./scaffold > files.txt` shouldn't put an error message in `files.txt`. And a pipeline can tell them apart.

**ASK** <br>
And `exit 1`? <br>
**ANSWER** <br>
It's how a script says **"I failed"**. Zero means success; anything else means failure. <br>
Hold onto that — in about an hour Jenkins will be reading those exit codes to decide whether your build is green or red. **That's the entire contract between your scripts and every automated system that will ever run them.**

**The heredocs:**

**ASK** <br>
Find `cat > server/index.js << 'EOF'` — the marker is quoted. Now find `cat > server/app.js << EOF` — it isn't. Why the difference? <br>
**ANSWER** <br>
`app.js` contains `${resource}Router`, which bash **must** substitute — that's how the file gets the right name in it. <br>
`index.js` contains `` `Listening on port ${PORT}` `` — and that `${PORT}` belongs to **Node**, not bash. Unquoted, bash would substitute its own non-existent `PORT` variable and write an empty string, giving you `Listening on port ` and a bug that looks like a Node problem. <br>
**Quote the marker by default.** An accidental substitution silently corrupts a file rather than erroring, which is the worst kind of bug to have.

**The four builds, two pushes:**

```bash
docker build -t "${username}"/"${resource}"-db:latest ./db
docker build -t "${username}"/"${resource}"-mvc:latest ./server

docker build --platform linux/amd64 -t "${username}"/"${resource}"-db-cloud:latest ./db
docker build --platform linux/amd64 -t "${username}"/"${resource}"-mvc-cloud:latest ./server

docker push "${username}"/"${resource}"-db-cloud:latest
docker push "${username}"/"${resource}"-mvc-cloud:latest
```

**ASK** <br>
Why build everything twice? <br>
**ANSWER** <br>
**Architecture.** On an Apple Silicon Mac, a plain `docker build` produces an **arm64** image. That runs beautifully on your laptop and **fails on a standard Azure VM** with `exec format error`. <br>
So the script builds a native pair for local use and an `amd64` pair for the cloud. Only the cloud ones get pushed, because those are the ones that need to run somewhere else. <br>
**If you're on an Intel Mac or Windows, the first two builds are redundant** — comment them out and save yourself a couple of minutes.


- Run: `uname -m` to find out. 


### HANDS ON (15 min) — run it

*(Run from `~/Desktop/planetary-app`)*
```bash
mkdir backend
```

I'm doing this because our script will create and push a repo and I want to keep our credentials outside of that. 


```bash
scaffold-extended planets Planet <your-dockerhub-username>
```

**Your Docker Hub username and your GitHub username must be the same**, because the script uses the third argument for both. If they differ, run it with your GitHub name and re-tag the images afterwards.

Then look at what you got:

```bash
tree
cat Jenkinsfile
cat docker-compose.yml
```

**END OF NOTE**

You should have this:

```
.
├── db
│   ├── Dockerfile
│   ├── planets.sql
│   └── .dockerignore
├── docker-compose.yml
├── Jenkinsfile
├── README.md
├── .gitignore
├── server
│   ├── app.js
│   ├── controllers/planets.js
│   ├── db/connect.js
│   ├── Dockerfile
│   ├── index.js
│   ├── models/Planet.js
│   ├── package.json
│   └── routers/planets.js
└── terraform
    ├── backend/main.tf
    └── infrastructure/*.tf
```

Plus a **GitHub repository already created and pushed**, and **two images already on Docker Hub**.


Look at `db/Dockerfile`. It copies the SQL file into `/docker-entrypoint-initdb.d/`. That's because the official Postgres image runs **anything** in that folder on first startup. It's a convention the image author built in. <br>
Which means your database image isn't just Postgres — it's **Postgres with your schema already in it**. That's a deliberate choice, and it's why we can deploy a database with no separate setup step later.


Now look at `docker-compose.yml`. It references `image:` and not `build:`. That tells us it's written for **deployment, not development**. It pulls finished images from Docker Hub rather than building from source. <br>
Which means it works on a machine that has **no source code on it at all** — which is exactly the machine we're going to create in Azure. The script wrote a deployment file before we had anywhere to deploy to.


The **Jenkinsfile** has four stages that each `echo` a string and do nothing. **It's a stub** — the shape of the pipeline with the work left out. <br>
That's what we're filling in, one stage at a time, and it's why we'll have something green within the hour.

### Prove it locally

Let us run the application locally to make sure everything behaving itself. 

In the **docker-compose.yml** I'm going to remove the `-cloud` section on both images. Those images as we saw in the script was built for **amd architecture** which the cloud has but my laptop does not. 

**docker-compose.yml**
```yml

services:
  planets-mvc:
    # UPDATED
    image: emilesherrott/planets-mvc:latest
    ports:
      - "80:80"
    restart: always
    depends_on:
      - planets-db
    networks:
      - planets-network

  planets-db:
    # UPDATED
    image: emilesherrott/planets-db:latest
    ports:
      - "5432:5432"
    restart: always
    networks:
      - planets-network

networks:
  planets-network:
```

*(Run from `~/planetary-app/backend`)*
```bash
docker compose up -d
docker ps
curl localhost:80/planets
```

You should get two example rows back as JSON.

**ASK** <br>
What did you get? <br>
**ANSWER** <br>
`[{"id":1,"name":"Example one"},{"id":2,"name":"Example two"}]` <br>
**That's the generic skeleton.** The plumbing is correct — Express is routing, the controller is calling the model, the model is querying Postgres, and the two containers are talking to each other over a Docker network. **It just doesn't mean anything yet.** <br>
That's after the break. For now it only has to start, because what we build next is the machinery that ships it.

If `curl` fails here, **stop and fix it** before we go near Jenkins. A pipeline that ships a broken app is far harder to debug than an app that's broken on your desk. <br>

- *RUN `docker compose down`*

---
---

## 01:00–01:25 — A Jenkins that can do the job

*(Activity: 25 min)*

Jenkins runs in a container. But there's a problem.

**ASK** <br>
Our pipeline needs capability for `docker build` and `terraform plan`. The official `jenkins/jenkins` image has neither. What happens when a stage calls them? <br>
**ANSWER** <br>
`docker: not found`, a non-zero exit code, a red stage. Which is **correct behaviour** — the pipeline failed honestly. <br>
But it means our first job is building a Jenkins that has the tools it's meant to orchestrate. Obvious in hindsight, invisible until it bites.

### The Dockerfile

*(Run from `~/planetary-app`)* — create `jenkins-image/Dockerfile`:

```dockerfile
FROM jenkins/jenkins:lts-jdk17

USER root

RUN apt-get update \
    && apt-get install -y docker.io curl gnupg lsb-release \
    && rm -rf /var/lib/apt/lists/*

RUN curl -fsSL https://apt.releases.hashicorp.com/gpg \
      | gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" \
      > /etc/apt/sources.list.d/hashicorp.list \
    && apt-get update \
    && apt-get install -y terraform \
    && rm -rf /var/lib/apt/lists/*

USER jenkins
```

Every line is Docker you already have.

**`USER root`** — installing packages needs admin rights; we drop back at the end.

**`gpg --dearmor`** imports HashiCorp's signing key, so apt can verify the Terraform package genuinely came from them rather than from someone who'd compromised a mirror.

**`$(lsb_release -cs)`** is command substitution — it prints the Debian codename so the repository line matches this base image.

**ASK** <br>
Why `&&` chaining inside one `RUN`, rather than six separate `RUN` lines? <br>
**ANSWER** <br>
Because **each `RUN` is a layer, and layers are additive.** So deleting a file in a later layer doesn't reclaim the space from an earlier one — it just hides it. <br>
So `apt-get update` in one layer and `rm -rf /var/lib/apt/lists/*` in another leaves the package index in the image anyway. Chaining puts the download and the cleanup in the same layer, so the cleanup actually shrinks the result.

### HANDS ON (25 min)

# CONTINUE

*(Run from `~/planetary-app/jenkins-image`)*
```bash
docker build -t jenkins-docker .
```

*(Run from `~/planetary-app`)*
```bash
docker run -d --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -u root \
  jenkins-docker

docker ps
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

*(In your browser — `http://localhost:8080`)*
- Paste the password
- **Install suggested plugins**
- Create your admin user — **write it down**

**Then verify. Nobody moves on until both of these respond:**

```bash
docker exec jenkins terraform version
docker exec jenkins docker --version
```

This verification step is the highest-value thirty seconds of the session. **Enforce it.** Every `command not found` in the next two hours traces back to here, and it's far cheaper to catch now than inside a build log. <br>

# EXPLAIN MORE DETAIL & HOW DONE IN REAL LIFE

**ASK** *(while the build runs)* <br>
We mounted `/var/run/docker.sock` into the container. What does that actually let Jenkins do? <br>
**ANSWER** <br>
Talk to **your machine's** Docker daemon. So when the pipeline runs `docker build`, the image is built by your Docker and appears in your `docker images`. <br>
There's no Docker *running* inside the Jenkins container — only the client. The socket is the pipe to the real thing. <br>
It's also a large amount of trust. Anything that can reach that socket can start a privileged container and own your machine. Fine on a laptop; a genuine security decision on a shared server.


### Store the credentials

*(In the Jenkins UI)* **Manage Jenkins → Credentials → System → Global credentials → Add Credentials**

| Kind | ID | Value |
|---|---|---|
| Username with password | `dockerhub-credentials` | your Docker Hub **username** + the token |
| Secret text | `azure-client-id` | the `appId` |
| Secret text | `azure-client-secret` | the `password` |
| Secret text | `azure-tenant-id` | the `tenant` |
| Secret text | `azure-subscription-id` | from `az account show --query id -o tsv` |

**The IDs must match exactly** — your `Jenkinsfile` refers to them by these strings.

**ASK** <br>
Why is Docker Hub a Username-with-password but Azure is four separate Secret texts? <br>
**ANSWER** <br>
It's due to how we'll access them in the pipeline, we'll access them through `credentials()`, a method and **username-with-password** behaves differently to **secret text**. A **Secret text** injects one string into one variable. A **Username with password** injects *three* — `$VAR`, `$VAR_USR` and `$VAR_PSW`. <br>
Terraform's provider wants four discrete values, so four Secret texts maps cleanly. Docker genuinely wants a username **and** a password as a pair, so there the combined type is right.

**ASK** <br>
And why not just write them into the `Jenkinsfile`, after all it's our repo. <br>
**ANSWER** <br>
Because the `Jenkinsfile` is **committed, and Git history is permanent.** Deleting the line tomorrow doesn't remove it from history. <br>
A cloud credential with Contributor rights in a repository is a security incident, not an untidiness problem. The credential store also **masks these in build logs automatically** — an accidental `echo` prints `****`.

---
---

## 01:25–01:50 — Pipeline v1: fill in the stubs

*(Activity: 25 min)*

Open the `Jenkinsfile` the script generated:

```groovy
pipeline {
    agent any
    stages {
        stage('Docker Login')        { steps { echo "Docker Login" } }
        stage('Build Docker Images') { steps { echo "Build Docker Images" } }
        stage('Push Docker Images')  { steps { echo "Push Docker Images" } }
        stage('Terraform')           { steps { echo "Terraform" } }
    }
}
```

**Four stages, all doing nothing.** We're going to fill in the first three now and leave Terraform stubbed until after the break.

### The filled-in version

```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME_DB  = 'yourname/planets-db-cloud'
        IMAGE_NAME_MVC = 'yourname/planets-mvc-cloud'
        IMAGE_TAG      = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Building ${IMAGE_NAME_MVC}:${IMAGE_TAG}"
            }
        }

        stage('Build Docker Images') {
            steps {
                dir('db') {
                    sh 'docker build --platform linux/amd64 -t $IMAGE_NAME_DB:$IMAGE_TAG .'
                    sh 'docker build --platform linux/amd64 -t $IMAGE_NAME_DB:latest .'
                }
                dir('server') {
                    sh 'docker build --platform linux/amd64 -t $IMAGE_NAME_MVC:$IMAGE_TAG .'
                    sh 'docker build --platform linux/amd64 -t $IMAGE_NAME_MVC:latest .'
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $IMAGE_NAME_DB:$IMAGE_TAG'
                    sh 'docker push $IMAGE_NAME_MVC:$IMAGE_TAG'
                    sh 'docker push $IMAGE_NAME_DB:latest'
                    sh 'docker push $IMAGE_NAME_MVC:latest'
                }
            }
        }

        stage('Terraform') {
            steps { echo "Terraform — coming after the break" }
        }
    }

    post {
        success { echo "Pushed ${IMAGE_TAG}" }
        failure { echo "FAILED — see ${BUILD_URL}console" }
    }
}
```

**Replace `yourname`** with your Docker Hub username.

**Notice the `Docker Login` stub disappeared.** Logging in isn't really a stage — it's something the push stage needs, scoped tightly around it with `withCredentials`. Separating them would mean the credential was in scope for longer than necessary.

**`IMAGE_TAG = "${BUILD_NUMBER}"`** tags every image with the build that produced it, so you can trace any running container back to an exact build, and that build to an exact commit.

**`dir('db')`** runs the enclosed steps inside that folder — `cd db` for a block, putting you back afterwards.

**`withCredentials`** scopes the secret to a block. Inside the braces you can use `$DOCKER_USER` and `$DOCKER_PASS`; **outside, they don't exist.**

**`--password-stdin`** pipes the password in rather than putting it on the command line, where it would appear in process listings.

### HANDS ON (25 min)

*(Run from `~/planetary-app`)*
```bash
git add Jenkinsfile
git commit -m "Pipeline v1: build and push"
git push origin main
```

*(In the Jenkins UI)*
1. **New Item** → `planetary-app-pipeline` → **Pipeline** → **OK**
2. **Pipeline** → **Definition**: `Pipeline script from SCM`
3. **SCM**: `Git`, your repo URL, **Branch**: `*/main`, **Script Path**: `Jenkinsfile`
4. **Build Triggers** → tick **Poll SCM** → `H/2 * * * *`
5. **Save** → **Build Now**

Then check [hub.docker.com](https://hub.docker.com) — both images should have a new tag, numbered `1`.

**END OF NOTE**

**That's your first green build.** Not a toy one — it genuinely rebuilt and republished your application.

**ASK** <br>
The script already pushed those images half an hour ago. So what's different now? <br>
**ANSWER** <br>
**You didn't do it.** And more importantly, **you can't accidentally not do it**. <br>
The script was a one-off you had to remember to run, from a machine with the right tools, with the right arguments. The pipeline happens on every push, the same way, whether or not anyone remembers. <br>
That's the difference between a script and automation.

**ASK** <br>
Poll SCM checks every two minutes. Why not a webhook, which would be instant? <br>
**ANSWER** <br>
Because GitHub is on the public internet and your Jenkins is on `localhost`. **There's no route for GitHub to reach you.** <br>
A webhook is what you'd use for real, and you could get there today with a tunnel like ngrok. Polling demonstrates the identical principle with no networking involved.

**NOTE FOR TRAINERS** <br>
**Make them push a change and wait**, rather than clicking Build Now again. A build starting on its own, two minutes after a `git push`, is the moment the session lands for most people. Worth the two minutes of apparent nothing. <br>
**END OF NOTE**

---
---

*(Take a 15 minute break here.)*

---
---

## 02:05–02:25 — Make it a planets app

*(Activity: 20 min)*

We have a pipeline that ships a skeleton. Let's give it something worth shipping.

Everything here is ordinary application code. **The pipeline doesn't change at all**, which is the point worth noticing.

### The SQL

**`db/planets.sql`** — replace the generated contents:

```sql
DROP TABLE IF EXISTS planets;

CREATE TABLE planets (
  id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  name VARCHAR(50) NOT NULL UNIQUE,
  planet_type VARCHAR(50) NOT NULL,
  diameter_km INT NOT NULL,
  mass_earths DECIMAL(10,4),
  distance_from_sun_m_km DECIMAL(10,3) NOT NULL,
  moons INT NOT NULL DEFAULT 0,
  has_rings BOOLEAN NOT NULL DEFAULT FALSE
);

INSERT INTO planets
  (name, planet_type, diameter_km, mass_earths, distance_from_sun_m_km, moons, has_rings)
VALUES
  ('Mercury', 'Terrestrial',   4879,   0.0553,   57.910,   0, FALSE),
  ('Venus',   'Terrestrial',  12104,   0.8150,  108.200,   0, FALSE),
  ('Earth',   'Terrestrial',  12742,   1.0000,  149.600,   1, FALSE),
  ('Mars',    'Terrestrial',   6779,   0.1074,  227.900,   2, FALSE),
  ('Jupiter', 'Gas Giant',   139820, 317.8000,  778.500,  95, TRUE),
  ('Saturn',  'Gas Giant',   116460,  95.2000, 1432.000, 146, TRUE),
  ('Uranus',  'Ice Giant',    50724,  14.5000, 2867.000,  28, TRUE),
  ('Neptune', 'Ice Giant',    49244,  17.1000, 4515.000,  16, TRUE);
```

### The model

**`server/models/Planet.js`** — add a second method:

```javascript
const db = require("../db/connect");

class Planet {
  static async findAll() {
    const result = await db.query("SELECT * FROM planets ORDER BY id");
    return result.rows;
  }

  static async findById(id) {
    const result = await db.query("SELECT * FROM planets WHERE id = $1", [id]);
    return result.rows[0];
  }
}

module.exports = Planet;
```

**ASK** <br>
Look at `server/db/connect.js` — every value reads an environment variable. Why not just hardcode them? <br>
**ANSWER** <br>
Because **the same image has to run in two different places.** Locally the database is a container called `planets-db`; in a different environment it might be a managed Postgres with a completely different hostname. <br>
Hardcode it and you need two images for two environments — which means **the thing you tested isn't the thing you shipped.** <br>
**Build once, configure at run time.** Same argument as Terraform variables, one layer up.

### HANDS ON (20 min)

*(Run from `~/planetary-app`)*
```bash
docker compose down -v
docker compose up -d --build
curl localhost:80/planets
```

**`down -v` matters** — the `-v` removes the volume. Without it, Postgres keeps the old database and **never re-runs your SQL**, because `/docker-entrypoint-initdb.d/` only executes on a fresh data directory.

Then push it and **watch the pipeline run on its own**:

```bash
git add .
git commit -m "Make it a planets API"
git push origin main
```

**END OF NOTE**

**ASK** *(when the build goes green)* <br>
You changed the application substantially. How many changes did the pipeline need? <br>
**ANSWER** <br>
**None.** It built, it pushed, it went green. <br>
That separation is worth a moment: **the pipeline cares about *how* you ship, not *what* you ship.** It's why one pipeline can serve a project for years while the application inside it is rewritten twice.

---
---

## 02:25–02:45 — Pipeline v2: somewhere to keep state

*(Activity: 20 min)*

Time to fill in that fourth stub. Before we can, there's a problem to solve.

**ASK** <br>
Your Terraform state has been a local file all course. A Jenkins build runs in a fresh workspace that's thrown away afterwards. What happens the first time that pipeline runs `terraform apply`? <br>
**ANSWER** <br>
**Disaster.** The build starts with **no state at all**, so Terraform believes nothing exists and plans to create everything — duplicating resources or erroring on name collisions. Then the workspace is destroyed and whatever state it wrote vanishes, so the **next** build starts from nothing again. <br>
**Remote state isn't a nice-to-have for pipeline Terraform. It's a hard prerequisite.**

So we create a storage account to hold it. And there's a chicken-and-egg: the storage account holding your state can't be stored in that storage account. So this project keeps **local** state, is created once by hand, and then left alone.

The script gave you `terraform/backend/main.tf` with a provider block. Add the resources:

```tf
resource "azurerm_resource_group" "backend_rg" {
  name     = "rg-tfstate-planetary-app"
  location = "swedencentral"
}

resource "azurerm_storage_account" "backend_state" {
  name                     = "stplanets<yourinitials>"
  resource_group_name      = azurerm_resource_group.backend_rg.name
  location                 = azurerm_resource_group.backend_rg.location
  account_tier             = "Standard"
  account_replication_type = "LRS"

  blob_properties {
    versioning_enabled = true
  }
}

resource "azurerm_storage_container" "tfstate" {
  name                  = "tfstate"
  storage_account_name  = azurerm_storage_account.backend_state.name
  container_access_type = "private"
}
```

**Storage account names are globally unique across all of Azure** — lowercase letters and numbers only, so add your initials.

**ASK** <br>
The resource group is `rg-tfstate-planetary-app` — deliberately different from the one our app will use. Why? <br>
**ANSWER** <br>
Two reasons. <br>
**Different lifecycles.** The backend is created once and never touched; the app changes constantly. <br>
**Blast radius.** `terraform destroy` on the app must not be able to delete the storage account holding its own state. <br>
And a practical one: if two Terraform projects both declare a resource group with the same name, each has its own state, **neither knows about the other**, and you get a very confusing `already exists` error.

### HANDS ON (20 min)

*(Run from `~/planetary-app/terraform/backend`)*
```bash
export ARM_CLIENT_ID=<appId>
export ARM_CLIENT_SECRET='<password>'
export ARM_SUBSCRIPTION_ID=<subscription-id>
export ARM_TENANT_ID=<tenant>

terraform init
terraform apply
```

Then look at what you just created:

```bash
grep -i key terraform.tfstate
```

**END OF NOTE**

**ASK** <br>
What did you find? <br>
**ANSWER** <br>
Your **storage account access keys**, in plaintext. Unencrypted, in a file people commit by accident every day. <br>
The script's `.gitignore` already covers `*.tfstate` — go and confirm it. That's not tidiness, it's the reason the container is `private` too. **State files aren't metadata; they contain real secrets.**

**ASK** <br>
Those four `ARM_` names — did I make them up? <br>
**ANSWER** <br>
No. The `azurerm` provider looks for **exactly** those names automatically. You never mention them in a `.tf` file. <br>
Which is why in a minute, putting them in the Jenkins `environment` block is all it takes. **We're inventing nothing** — Jenkins will do for a machine exactly what you just did with `export`.

---
---

## 02:45–03:10 — Pipeline v3: plan, then apply

*(Activity: 25 min)*

The script gave you six empty `.tf` files in `terraform/infrastructure/`. **I'm giving you their contents** rather than writing them together — they're all resource types you built by hand in the Terraform sessions, and our time is better spent on the pipeline.

*(Full contents in the accompanying student walkthrough: `main.tf`, `network-card.tf`, `network-security-group.tf`, `variables.tf`, `data-providers.tf`, `outputs.tf`.)*

The one addition:

*(Run from `~/planetary-app/terraform/infrastructure`)*
```bash
mkdir -p keys
cp ~/azure/azure_ssh_keys/default-vm-ssh.pub keys/
git check-ignore -v keys/default-vm-ssh.pub
```

**No output from that last command is what you want** — it means the file is tracked.

**ASK** <br>
We're committing an SSH key to a public repository. Is that alright? <br>
**ANSWER** <br>
Yes — because it's the **public** half. It's designed to be published; that's the entire point of asymmetric cryptography. <br>
Committing it means the pipeline has it automatically, with no credential to manage. **The private key never goes near the repo.**

### Replace the Terraform stub

```groovy
        stage('Terraform Init') {
            steps {
                dir('terraform/infrastructure') {
                    sh 'terraform init -reconfigure'
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                dir('terraform/infrastructure') {
                    sh 'terraform plan -out=tfplan'
                    sh 'terraform show -no-color tfplan > tfplan.txt'
                }
                archiveArtifacts artifacts: 'terraform/infrastructure/tfplan.txt',
                                 fingerprint: true
            }
        }
```

And add the four Azure credentials to `environment`:

```groovy
        ARM_CLIENT_ID       = credentials('azure-client-id')
        ARM_CLIENT_SECRET   = credentials('azure-client-secret')
        ARM_SUBSCRIPTION_ID = credentials('azure-subscription-id')
        ARM_TENANT_ID       = credentials('azure-tenant-id')
```

**Push it, and get a green plan before adding apply.**

**`terraform init -reconfigure`** — a Jenkins workspace persists between builds and can hold a cached copy of a previous backend configuration. If that changes, Terraform stops and asks whether you meant to migrate state or start fresh, and **a pipeline can't answer a question**. `-reconfigure` makes init deterministic.

**ASK** <br>
`terraform plan -out=tfplan` saves the plan to a file. Why not just let `apply` re-plan? <br>
**ANSWER** <br>
Because then `apply` **replays a decision already made**, rather than making a new one. Nothing can drift between what a human reviewed and what actually executes. <br>
That's the entire point of the two-stage split, and it's what makes the approval gate meaningful rather than theatre.

**`terraform show -no-color tfplan > tfplan.txt`** converts the binary plan to readable text. **`archiveArtifacts`** attaches it permanently to the build — so six months from now you can answer *"what exactly did we change on 14 March?"*, **an audit trail the Azure Activity Log can't give you.**

**NOTE FOR TRAINERS** <br>
`archiveArtifacts` **must be inside `steps`**. A `stage` can only contain declarative directives — `agent`, `when`, `environment`, `options`, `post`, `steps`. Anything that *does work* goes in `steps`. <br>
Put it outside and you get `Unknown stage section "archiveArtifacts"`, reported against the line where the **stage opens**, not where the mistake is. Mention it pre-emptively; several will hit it. <br>
**END OF NOTE**

### Now the apply

```groovy
        stage('Terraform Apply') {
            steps {
                script {
                    timeout(time: 15, unit: 'MINUTES') {
                        input message: 'Apply this plan?', ok: 'Apply'
                    }
                }
                dir('terraform/infrastructure') {
                    sh 'terraform apply -auto-approve tfplan'
                }
            }
        }
```

**ASK** <br>
`input` pauses the pipeline and waits for a human to click Apply. The whole point of CI/CD is removing manual steps — so why are we deliberately adding one back? <br>
**ANSWER** <br>
Because **infrastructure changes can be destructive and irreversible** in a way application deploys usually aren't. You've seen `must be replaced` in plan output — on a storage account holding data, that's catastrophic. <br>
A human reading the actual plan immediately before it executes is a cheap, valuable safety net, **especially while a team is still building trust in a new pipeline.** <br>
Fully unattended apply is common and legitimate. But it should be **a deliberate decision to remove the gate**, not something that happened by default.

**ASK** <br>
So when would you remove it? <br>
**ANSWER** <br>
**When the review has genuinely happened somewhere else** — on a pull request, where a teammate already read the plan and approved the merge. Gating again is just friction: you're asking someone to approve a decision already made. <br>
The gate belongs where a human is actually exercising judgement, and only there.

**ASK** <br>
And `-auto-approve` — that's the third time on this course we've removed an interactive prompt. Where were the others? <br>
**ANSWER** <br>
**`apt-get install -y`** and **`read -p`** in your bash scripts. Same lesson each time: interactive prompts are helpful by hand and fatal in automation, because there's nobody there and the job hangs until it times out. <br>
When you meet a new tool, *"how do I make this non-interactive?"* is one of the first questions worth asking.

**And `timeout` matters more than it looks.** Without it an un-approved build holds a Jenkins executor open indefinitely — and, worse, **holds the Terraform state lock**, blocking everyone else's pipeline. That's Session 5's locking mechanism biting in a way you'd never predict.

---
---

## 03:10–03:30 — cloud-init, verify, tear down

*(Activity: 20 min)*

One piece left. We'll have a virtual machine, but nothing on it.

**ASK** <br>
The obvious answer is a `remote-exec` provisioner — Terraform SSHes in and types the install commands. What's wrong with that from a pipeline? <br>
**ANSWER** <br>
Several things, and they compound: <br>
**The Jenkins container** needs to reach port 22 on that VM — so your firewall has to allow SSH from wherever Jenkins happens to be running, not just from you. <br>
**The private key** has to be inside the Jenkins container — another secret to manage and `chmod`. <br>
**It runs once, at creation, and is invisible to the plan.** Change a command inside it, re-apply, and nothing happens. <br>
HashiCorp themselves call provisioners **"a last resort"**. This is why.

**So we don't use one.** We hand the VM a configuration document at creation time and let it **build itself on first boot**. Nobody logs in. There's no SSH session. **So there's no private key in the pipeline at all.**

**`terraform/infrastructure/cloud-init.yaml`**
```yaml
#cloud-config
package_update: true
packages:
  - ca-certificates
  - curl

write_files:
  - path: /opt/app/docker-compose.yml
    permissions: '0644'
    content: |
      services:
        planets-mvc:
          image: yourname/planets-mvc-cloud:latest
          ports:
            - "80:80"
          restart: always
          depends_on:
            - planets-db
          networks:
            - planets-network

        planets-db:
          image: yourname/planets-db-cloud:latest
          restart: always
          networks:
            - planets-network

      networks:
        planets-network:

runcmd:
  - install -m 0755 -d /etc/apt/keyrings
  - curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
  - chmod a+r /etc/apt/keyrings/docker.asc
  - echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" > /etc/apt/sources.list.d/docker.list
  - apt-get update -y
  - apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
  - systemctl enable --now docker
  - usermod -aG docker azureuser
  - docker compose -f /opt/app/docker-compose.yml up -d
```

Wired in with one line on the VM resource:

```tf
  custom_data = base64encode(file("${path.module}/cloud-init.yaml"))
```

**`#cloud-config` on line one is required** — without it the file is ignored entirely.

**ASK** <br>
Compare that compose file to the one the script generated in your repo root. What's different? <br>
**ANSWER** <br>
**The database publishes no ports.** The script's version maps `5432:5432`, which is convenient locally for connecting a GUI client — and on a public VM would **expose Postgres to the internet**, with a password of `docker` baked into the image. <br>
The two containers reach each other by service name over the shared network. **They never needed the published port; you did.**

**ASK** <br>
So what does the pipeline now need the private SSH key for? <br>
**ANSWER** <br>
**Nothing.** That's the whole win. <br>
The **public** key still goes on the VM so *you* can log in and debug. But Terraform never opens an SSH session, so it never needs the private half. No secret file in Jenkins, no `chmod`, no firewall rule for the agent. <br>
Which means you can lock port 22 down to your own IP, because the only thing that needs it is you.

**One honest trade-off.** cloud-init runs **asynchronously after boot**, so Terraform reports the VM created and moves on while apt is still installing Docker. There's a two-to-four minute window where the machine exists but the app isn't up. `remote-exec` blocked until it finished, which is the one thing it was better at.

### HANDS ON (20 min)

```bash
git add .
git commit -m "cloud-init and the apply stage"
git push origin main
```

Watch the build. When it pauses at **Terraform Apply**, **read the plan**, then click Apply.

*(Run from `~/planetary-app/terraform/infrastructure`)*
```bash
terraform output public_ip
```

**Wait two to four minutes**, then visit `http://<public-ip>/planets`.

If it doesn't load:
```bash
ssh -i ~/azure/azure_ssh_keys/default-vm-ssh.pem azureuser@<ip>
cloud-init status
sudo cat /var/log/cloud-init-output.log
docker ps
```

**END OF NOTE**

### Tear it down

**Not optional. Azure charges by the hour whether you use it or not.**

```bash
cd terraform/infrastructure && terraform destroy
cd ../backend && terraform destroy

az group list -o table
az resource list -o table
```

**Both should come back empty.**

---
---

## Wrap-up

Count what you built:

- A generated application, from one command with three arguments
- Two Docker images, republished automatically on every push
- A storage account holding Terraform state, shared and locked
- A network, firewall, public IP and virtual machine, all described in code
- A VM that configures itself with no SSH access required
- A pipeline tying it together, gated by one human decision

**And you built it in five green increments**, filling in four empty stages one at a time.

**ASK** *(the last one)* <br>
Go back to the beginning. We ran a script that generated an application. Then a pipeline that ships it. Then infrastructure that runs it. What's common to all three? <br>
**ANSWER** <br>
**All of it is code, in one repository, reviewed the same way.** <br>
The application, the process that ships it, and the infrastructure it runs on. There's no Dev artifact thrown over a wall to an Ops process — there's one repo and one pipeline. <br>
That's not a slogan any more. **You can point at the files.**

### What's still missing

Worth being honest about, because it's where a real project goes next:

- **No monitoring or alerting** — you find out it's broken when someone tells you
- **No staging environment** — production is the first place a change runs
- **No automated rollback** — a bad deploy stays bad until you push a fix
- **Secrets in plaintext** — the database password is in the Dockerfile
- **`:latest` never redeploys** — the VM runs cloud-init once, so a new image changes nothing until the VM is recreated

**That last one is the interesting one.** Provisioning infrastructure and continuously deploying to it are **different problems**, and a single VM is a poor fit for the second.

Which is a decent argument for why container orchestration exists — and a good place to stop.

---

[Back](./README.md)

---
