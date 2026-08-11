# Integrating DevOps Toolchains: Git, Jenkins, Terraform & Azure

A half-day session bringing together everything covered separately so far — version control, CI/CD orchestration, Infrastructure as Code, and the cloud — into one real, working pipeline.

## Organisation

### Duration

Half day, **09:30 - 13:30**, split into two sessions with a break.

| Time | Session |
|---|---|
| 09:30 - 11:30 | Session 1: The Toolchain and Setting Up Jenkins |
| 11:30 - 11:45 | Break |
| 11:45 - 13:30 | Session 2: Automating Terraform Against Azure |

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
- **Make sure**
  - Docker is installed and running (we'll run Jenkins as a container)
  - Students have a GitHub account and a fork of the starter repo
  - Students have their **Service Principal** credentials from the Azure Fundamentals / Terraform sessions (`ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, `ARM_SUBSCRIPTION_ID`, `ARM_TENANT_ID`) — if not, we'll recreate one at the start of Session 2
  - Terraform is installed locally (used earlier, not strictly required for this session since Jenkins will run it, but handy for troubleshooting)

## Learning objectives

- **Understand** where Git, Jenkins, Terraform, and Azure each sit in a DevOps toolchain, and why no single tool does everything
- **Configure** Jenkins to automatically pull code from a Git repository on push
- **Write** a Jenkins Pipeline (`Jenkinsfile`) that runs Terraform against Azure
- **Authenticate** Jenkins to Azure using a Service Principal, without hardcoding credentials anywhere
- **Understand** how Pull Request review and pipeline automation combine to make infrastructure changes both safe and repeatable
- **Trigger and observe** a full pipeline run end to end — from a `git push` to a real change in Azure

<br>

---

## Session 1: The Toolchain and Setting Up Jenkins
### 09:30 - 11:30

### Sequence

#### Why do we need a "toolchain" at all?

Over the last few sessions we've used **Git** (version control), **Terraform** (Infrastructure as Code), and **Azure** (the cloud itself). 

**ASK** <br>
Every time we've run `terraform apply` so far, who's actually typed that command? <br>
**ANSWER** <br>
Us — a human, sat at a laptop.

That's the gap today closes. We've talked about the *idea* of a pipeline — git push, a PR gets reviewed alongside a `terraform plan`, then merge triggers an automatic `terraform apply` — back in the Azure Fundamentals session. Today we actually build it.

*REFER TO RESOURCE 1 - SLIDEE* <br>
![toolchain-1](./resources/toolchain-1.png)

**Jenkins** is the piece we're introducing today — an open-source automation server. Its job is to watch our Git repository and, when something changes, automatically run whatever we've told it to: tests, builds, or — what we care about today — Terraform.

**ASK** <br>
Jenkins isn't the only tool that does this job. What are some others you might have heard of? <br>
**ANSWER** <br>
GitHub Actions, GitLab CI, Azure Pipelines, CircleCI — there are many. Jenkins is one of the oldest and most widely adopted, it's highly extensible via plugins, and — importantly for us today — it's **self-hosted**, which means every piece of the pipeline is something we set up and can see, rather than a black box running on someone else's SaaS platform.

**ASK** <br>
What's the trade-off of self-hosting Jenkins compared to using a SaaS CI/CD tool like GitHub Actions? <br>
**ANSWER** <br>
Full control, no vendor lock-in, and nothing running outside infrastructure we manage ourselves — but we're also on the hook for patching, scaling, and maintaining the Jenkins server itself, which a SaaS tool would otherwise handle for us.

<br>
<br>

#### Installing and running Jenkins

The fastest way to get a working Jenkins instance for training purposes is via Docker.

* Run:
```bash
docker run -d --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

* `-p 8080:8080` — the Jenkins web UI
* `-p 50000:50000` — used for communication with any build agents we attach later (not needed today, but standard to expose)
* `-v jenkins_home:/var/jenkins_home` — a named Docker volume so our Jenkins configuration survives a container restart

**NOTE FOR TRAINERS** <br>
This mirrors a conversation worth having explicitly: we're running Jenkins itself as a container, which is a good moment to connect back to why Docker mattered in the first place — Jenkins doesn't care what's installed on the host machine, it brings everything it needs with it. <br>
**END OF NOTE**

Jenkins takes a minute or two to start. Once it's up:

* Grab the initial admin password:
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

* Visit `http://localhost:8080`, paste in the password
* Choose **Install suggested plugins**
* Create your first admin user when prompted

<br>
<br>

#### A tour of the Jenkins UI

Once logged in, let's orient ourselves:

* **Dashboard** — lists all our Jenkins "jobs" (called **Items**)
* **New Item** — where we create a new job
* **Manage Jenkins** — plugins, credentials, system configuration
* **Credentials** — a secure store for secrets Jenkins needs (tokens, passwords, keys) — we'll be using this heavily today

Jenkins supports a few different job types. The one that matters to us is a **Pipeline**.

**ASK** <br>
Given everything we've said so far about writing infrastructure as text files rather than clicking through a Portal, what do you think a Jenkins "Pipeline" is defined as? <br>
**ANSWER** <br>
Code, not clicks — a **Pipeline** is defined in a text file called a `Jenkinsfile`, checked into the same Git repository as everything else. This is the exact same philosophy as Terraform's `.tf` files: the pipeline itself becomes version-controlled, reviewable, and repeatable, rather than a fragile set of manual UI configuration.

There's also an older job type called **Freestyle project**, configured entirely through the UI. We won't use it today — it's worth knowing it exists, but it has all the same "ClickOps" downsides we talked about with manually building infrastructure through the Azure Portal.

<br>
<br>

#### Connecting Jenkins to our Git repository

Let's create our first Pipeline job.

* **New Item** > name it `terraform-pipeline` > choose **Pipeline** > **OK**
* Scroll down to **Pipeline**, and change **Definition** from `Pipeline script` to `Pipeline script from SCM`
* **SCM**: `Git`
* **Repository URL**: your fork of the starter repo, e.g. `https://github.com/<your-username>/devops-toolchain-starter.git`
* **Branch**: `*/main`
* **Script Path**: `Jenkinsfile` (we'll write this shortly)

Because our fork is private (or even if it's public and we want to avoid rate limits), we'll want Jenkins to authenticate to GitHub properly.

* On GitHub: **Settings > Developer settings > Personal access tokens** — generate a token with `repo` scope
* Back in Jenkins: **Manage Jenkins > Credentials > (global) > Add Credentials**
  * **Kind**: Username with password
  * **Username**: your GitHub username
  * **Password**: the token you just generated
  * **ID**: `github-credentials`
* Back on our Pipeline job configuration, under **Repository URL**, select these credentials

**ASK** <br>
Why store this token in Jenkins' Credentials store, rather than just pasting it straight into the Pipeline configuration or the `Jenkinsfile` itself? <br>
**ANSWER** <br>
The exact same reasoning as everywhere else we've handled secrets this course — a `Jenkinsfile` lives in Git, and Git history is essentially permanent and often shared with a whole team. Jenkins' Credentials store keeps the secret value out of version control entirely, automatically masks it in console output (so it can't leak into build logs), and lets us rotate or revoke it centrally without touching any code.

<br>
<br>

#### Triggering builds automatically with webhooks

Right now, Jenkins will run our pipeline if we click **Build Now** — but that's not automation, that's just a different button to click. We want Jenkins to notice a `git push` automatically.

* On GitHub: **Settings > Webhooks > Add webhook**
  * **Payload URL**: `http://<your-jenkins-url>:8080/github-webhook/`
  * **Content type**: `application/json`
  * **Which events**: Just the push event

* Back in Jenkins, on our Pipeline job: **Configure > Build Triggers > GitHub hook trigger for GITScm polling**

**NOTE FOR TRAINERS** <br>
If Jenkins is running locally (as it is via our Docker command), GitHub's webhook can't reach `localhost` directly — a tool like [ngrok](https://ngrok.com/) is the usual workaround for training environments to expose the local Jenkins instance to the internet temporarily. Mention this alternative if students hit connection issues, or fall back to **Poll SCM** with a short interval (e.g. `* * * * *` for every minute) as a simpler, if less elegant, substitute. <br>
**END OF NOTE**

Let's test it — push a trivial change (like a comment in the `README.md`) to our repo, and watch the Jenkins Dashboard. A new build should kick off automatically, without anyone clicking anything.

**ASK** <br>
What have we actually built so far, even before Terraform enters the picture? <br>
**ANSWER** <br>
A working trigger — every `git push` now automatically causes Jenkins to do *something*. The "something" is what we'll define properly in Session 2.

<br>

---

## Session 2: Automating Terraform Against Azure
### 11:45 - 13:30

### Sequence

#### Authenticating Jenkins to Azure

Just like our GitHub token, Jenkins needs a way to authenticate to Azure — and just like everything else we've automated this course, that means a **Service Principal**, not a human login.

If you don't already have one from earlier sessions:

```bash
az ad sp create-for-rbac --name "jenkins-terraform" --role="Contributor" --scopes="/subscriptions/<your-subscription-id>"
```

This returns `appId`, `password`, and `tenant` — exactly as it did back in the Terraform sessions.

Let's store all four required values as Jenkins credentials:

* **Manage Jenkins > Credentials > Add Credentials**, repeated four times, each as **Kind: Secret text**:
  * ID `azure-client-id` — value: the `appId`
  * ID `azure-client-secret` — value: the `password`
  * ID `azure-subscription-id` — value: your subscription ID
  * ID `azure-tenant-id` — value: the `tenant`

**ASK** <br>
Why do these need to be Jenkins Secret text credentials, rather than just defined as plain environment variables inside the `Jenkinsfile` itself? <br>
**ANSWER** <br>
Exactly the same reasoning as our GitHub token — a `Jenkinsfile` is committed to Git and visible to anyone with repo access; Jenkins' credential store keeps the actual secret values out of that file entirely, and masks them automatically if they ever appear in console output.

<br>
<br>

#### Writing a Jenkinsfile

A **declarative Pipeline** has a predictable shape: an `agent` (where it runs), optional `environment` variables, and a series of named `stages`, each containing `steps`.

Let's build ours up piece by piece. Create a file called `Jenkinsfile` at the root of our repo, alongside our existing `.tf` files from the Terraform sessions.

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

We're pulling each credential in via the `credentials()` helper, referencing the IDs we just set up — Jenkins injects the actual secret value as an environment variable at run time, without it ever appearing in the pipeline definition itself.

**ASK** <br>
Where have we seen this exact set of four environment variable names before? <br>
**ANSWER** <br>
`ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, `ARM_SUBSCRIPTION_ID`, `ARM_TENANT_ID` — this is precisely what we `export`ed by hand at our terminal in the very first Terraform session. Terraform's `azurerm` provider automatically looks for these specific variable names — we're not inventing anything new, just letting Jenkins set them instead of us typing `export` ourselves.

Let's add the actual Terraform stages:

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

        stage('Terraform Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
            }
        }

        stage('Terraform Apply') {
            steps {
                input message: 'Apply this Terraform plan?'
                sh 'terraform apply -auto-approve tfplan'
            }
        }
    }
}
```

A couple of things worth pausing on:

* `terraform plan -out=tfplan` — we've done this before, back in the very first Terraform session. Saving the plan to a file means the exact plan we review is the exact plan that gets applied — nothing can silently drift between the two steps.
* `input message: '...'` — this pauses the pipeline and waits for a human to click **Proceed** in the Jenkins UI before continuing.

**ASK** <br>
The whole point of CI/CD is removing manual steps — so why deliberately add one back in here, right before `apply`? <br>
**ANSWER** <br>
Infrastructure changes can be destructive — deletions, forced replacements, unexpected scope. A human sanity-check on the actual `plan` output, immediately before it's applied for real, is a valuable safety net — especially while a team is still building trust in a new pipeline. Fully unattended `apply` is common too, once a team is confident enough in their testing and review process, but it's a deliberate choice to remove that gate, not a default.

<br>
<br>

#### Running the full pipeline

Let's push this. Add the `Jenkinsfile` alongside our existing Terraform configuration (for example, the resource group + storage account example from earlier sessions) and push to `main`.

* `git add Jenkinsfile`
* `git commit -m "Add Jenkins pipeline for Terraform"`
* `git push`

Our webhook should fire, and a new build should start automatically on the Jenkins Dashboard.

* Click into the running build, then **Console Output** to watch it live
* You'll see `Checkout`, then `terraform init` downloading the `azurerm` provider — exactly the same output we've seen before, just running on Jenkins' machine instead of ours
* `terraform plan` will run and pause, waiting on our **Terraform Apply** stage's `input` step

Click **Proceed** when you're happy with the plan, and watch `terraform apply` run to completion.

* Head to the **Azure Portal** and confirm the resource actually exists — the same verification step we did by hand throughout the earlier Terraform sessions, except this time nobody ran `terraform apply` themselves.

<br>
<br>

#### Tying it back to Pull Requests

We've built the mechanics, but let's connect this explicitly to the fuller picture from the Azure Fundamentals session.

*REFER TO RESOURCE 2 - SLIDEE* <br>
![toolchain-2](./resources/toolchain-2.png)

A more complete real-world pattern separates **plan** from **apply** based on *what kind* of Git event triggered the pipeline:

* **Pull Request opened or updated** → pipeline runs `terraform plan` only, and posts the output as a comment on the PR for reviewers to read (commonly done via a Jenkins plugin, or a small script calling the GitHub API directly)
* **Pull Request merged into `main`** → pipeline runs `terraform apply` automatically, with no manual gate needed — because the review *already happened*, as part of the PR approval itself

**ASK** <br>
How does this map onto the diagram we drew in the Azure Fundamentals session, before we'd built any of this for real? <br>
**ANSWER** <br>
It's the literal implementation of it. "Terraform plan posted for review before merge, apply happens automatically after merge" was a diagram then — today, for a simpler case, it's genuinely running.

We won't fully build the PR-triggered variant today given time, but it's a natural extension of exactly what we've just built — the same `Jenkinsfile` concepts, just triggered on a different Git event, and with the `input` gate replaced by "the PR review already was the gate."

<br>
<br>

#### Failure scenarios and troubleshooting

**ASK** <br>
If our `Terraform Plan` stage fails outright — a typo in a `.tf` file, say — does anything happen to our real Azure infrastructure? <br>
**ANSWER** <br>
No. `terraform apply` never runs. Splitting `plan` and `apply` into separate pipeline stages is a deliberate safety boundary, not just tidiness — a broken `plan` stops the pipeline dead before anything real is touched.

**ASK** <br>
What if `apply` itself fails halfway through — say, three resources are created successfully and a fourth fails? <br>
**ANSWER** <br>
This connects straight back to the idempotency conversation from the Linux and Automation session — Terraform's state file will reflect exactly what *did* get created, and running the pipeline again (once the underlying problem is fixed) will only attempt to create the resources that are still missing, not recreate everything from scratch.

<br>
<br>

#### Wrap-up: Seeing the whole toolchain

Let's recap the full loop we've built, end to end:

*REFER TO RESOURCE 3 - SLIDEE* <br>
```
engineer edits .tf files
        |
        v
git push
        |
        v
GitHub webhook fires
        |
        v
Jenkins pipeline triggered
        |
        v
Checkout -> terraform init -> terraform plan
        |
        v
human approval (or PR review, in the fuller pattern)
        |
        v
terraform apply
        |
        v
Azure resources updated
        |
        v
change verified in the Azure Portal
```

And where everything we've covered across the whole course plugs into that loop:

* **Linux & shell scripting** — the `sh` steps inside our `Jenkinsfile` are just bash, same as every provisioner and script we've written all along
* **Azure Fundamentals** — the Service Principal and RBAC concepts from that session are exactly what authenticates Jenkins to Azure today
* **Terraform** — the `.tf` files themselves are unchanged from earlier sessions; only *who* runs them has changed
* **Git** — the source of truth for both our infrastructure code and our pipeline definition, and the trigger for the whole thing
* **Jenkins** — the glue holding all of the above together, watching for changes and acting on them without a human needing to remember every step

<br>
<br>

### Exercise

Working individually or in pairs:
1. Set up your own Jenkins Pipeline job against your fork of the starter repo
2. Write a `Jenkinsfile` that runs `terraform init`, `plan`, and `apply` against a simple resource (a resource group and storage account is plenty)
3. Get a full pipeline run green end to end — from `git push` to a verified change in the Azure Portal
4. **Stretch**: split the pipeline so a Pull Request only triggers `terraform plan`, and only a merge to `main` triggers `terraform apply`

<br>
<br>

### Conclusion

- **Inform** students this marks the end of the DevOps Toolchain Integration session
- **Recap** that every tool from the course so far — Linux, Git, Terraform, Azure — has now been seen working together inside one real, automated pipeline
- **Tell** students what's coming next — extending this pipeline to include Kubernetes deployments, or adding automated testing stages before the Terraform stages run
- **Direct** students to the exercises for further practice, and to the [Jenkins Pipeline documentation](https://www.jenkins.io/doc/book/pipeline/) for anyone wanting to go deeper

---

[Back](./README.md)

---

