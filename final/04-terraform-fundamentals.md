# Session 4 — Terraform & Infrastructure as Code with Azure (Part 1 of 2) — Trainer Script

Taking trainees from "I've clicked resources into existence in the Azure Portal" to "I can describe infrastructure as code, understand what Terraform's state file is actually for, and create many resources from one configuration block". This is written as a script — read it out, adapt where it feels natural, but the intent is that you could deliver this cold from the page.

**Part 2** covers real infrastructure — networks, VMs, load balancers — and moving state to a remote backend.

---

## 📦 STARTER CODE — put this in the repo before training

Everything here goes into **`/infrastructure-as-code-terraform/intro-to-infrastructure-as-code/starter-code`** before the session.

**Almost nothing is provided as working code, deliberately.** Typing your first `resource` block is the single most valuable ten seconds of the day. What the repo gives you is a **reference sheet**, a **notes file**, and a `.gitignore` that must exist before anyone runs `terraform apply` — because state files contain secrets and we are not putting those in Git.

<br>

**`README.md`**
```markdown
# Terraform & IaC — Starter Code

## What's here

- **commands.md** — every Terraform command we use today, with a
  one-line explanation. Use it as a reference.

- **my-terraform-notes.md** — you fill this in. Includes the four
  Azure values you recorded in Session 1.

- **.gitignore** — COPY THIS to the root of your terraform folder
  BEFORE your first `terraform apply`. State files contain secrets
  in plain text and must never reach GitHub.

- **completed-code/** — the finished .tf files for each exercise.
  Catch-up only. Writing them yourself IS the session.

## Before we start

    terraform version     # if this fails, we install it at 10:15
    az account show       # you must be signed in
    echo $ARM_CLIENT_ID   # will be empty — we set this up today
```

<br>

**`.gitignore`** *(students copy this into their working folder immediately)*
```gitignore
# Terraform state — contains secrets IN PLAIN TEXT. Never commit.
*.tfstate
*.tfstate.*

# The downloaded provider plugins — hundreds of MB, and re-downloadable
**/.terraform/

# Variable files often hold environment-specific or sensitive values
*.tfvars
!example.tfvars

# Crash logs
crash.log
```

<br>

**`commands.md`**
```markdown
# Terraform CLI Reference — Session 4

## The core loop

    terraform init        Download the providers this project needs.
                          Run once per project, and again if you add a provider.

    terraform plan        Work out what WOULD change. Changes nothing.
    terraform apply       Actually make the changes. Asks you to type 'yes'.
    terraform destroy     Delete everything this project manages.

## Checking your work

    terraform validate    Is the config syntactically valid? Doesn't call Azure.
    terraform fmt         Auto-format all .tf files to the standard style.
    terraform show        Print the current state in readable form.
    terraform console     Interactive REPL for querying your config and state.

## Useful flags

    terraform plan -out=tfplan        Save the plan to a file
    terraform apply "tfplan"          Apply exactly that saved plan (no prompt)
    terraform apply -refresh=false    Skip asking Azure what really exists
    terraform apply -target=TYPE.NAME Only touch one resource
    terraform apply -var="name=value" Set a variable on the command line

## Inspecting state (safer than opening the JSON)

    terraform state list              Every resource Terraform manages
    terraform state show ADDRESS      All attributes of one resource

## Referring to things

    <resource_type>.<internal_name>              a resource
    <resource_type>.<internal_name>.<attribute>  one of its attributes
    var.<variable_name>                          a variable
    data.<type>.<name>                           a data source
```

<br>

**`my-terraform-notes.md`** *(students fill this in)*
```markdown
# My Terraform Notes

## From Session 1 — copy these across from my-azure-details.md

| What | Value |
|---|---|
| Subscription ID | |
| Tenant ID | |
| **Tenant domain** | e.g. `jbloggs.onmicrosoft.com` |
| My storage prefix | e.g. `stjbloggs` |

**The tenant domain matters today.** When we create an Azure AD user,
its `user_principal_name` MUST be on a domain your tenant owns.
Using someone else's will fail.

## Service Principal for Terraform

Created today. These become the four ARM_ environment variables.

| Returned by az | Terraform calls it | Environment variable | Value |
|---|---|---|---|
| `appId` | Client ID | `ARM_CLIENT_ID` | |
| `password` | Client Secret | `ARM_CLIENT_SECRET` | **never commit** |
| `tenant` | Tenant ID | `ARM_TENANT_ID` | |
| *(from az account show)* | Subscription ID | `ARM_SUBSCRIPTION_ID` | |

## The three states — in my own words

| State | What it is |
|---|---|
| Desired | |
| Known | |
| Actual | |
```

<br>

**`completed-code/`** — put finished `.tf` files for each folder (`01-terraform-basics`, `02-count`, `03-lists-and-sets`, `04-maps`) here. Catch-up only.

<br>

---

## 💬 SLACK SNIPPETS — queue these before you start

**Post reactively**, at the point they're needed. Terraform blocks are long and unforgiving — a missing brace or a mistyped resource type costs a student ten minutes, and there's no learning in the transcription.

| # | When | What to post |
|---|---|---|
| 1 | 09:05 | The pre-flight check |
| 2 | 10:15 | Install commands + `az ad sp create-for-rbac` |
| 3 | 10:40 | The four `export ARM_*` lines |
| 4 | 10:50 | The `terraform`/`provider` block for `main.tf` |
| 5 | 11:15 | The resource group + storage account blocks |
| 6 | 12:20 | The `azuread` provider + user block |
| 7 | 14:00 | The `.gitignore` reminder + variable block |
| 8 | 15:15 | The `03-lists-and-sets` starting config |
| 9 | 16:45 | The destroy commands |

Each is marked **💬 SLACK** in the body below.

---

## Organisation

### Audience

Trainees who have completed **Session 1 (Azure fundamentals)**, **Session 2 (bash scripting)** and **Session 3 (Jenkins & CI/CD)**.

They are experienced developers: **Node, Express, REST, MVC, SQL, Git, GitHub, Docker, frontend, unit and integration testing**. Several things they already know map almost perfectly onto today and should be used constantly:

- **They know what declarative means**, even if they've never used the word — React's render output, or an SQL query, describes a *result* rather than steps
- **They know dependency resolution** — `npm install` builds a graph and installs in the right order. Terraform does the same for cloud resources
- **They know `package-lock.json`** — which is conceptually identical to `.terraform.lock.hcl`
- **They know code review** — and today the thing being reviewed becomes a server

What's genuinely new: **HCL syntax**, **the state file** (the single biggest conceptual hurdle), and the idea that infrastructure has a **lifecycle** you manage rather than a thing you set up once.

**NOTE FOR TRAINERS** <br>
The concept that trips up this room is **state**. Not the syntax — HCL is trivially readable to anyone who writes JSON. The hurdle is that Terraform maintains its own *memory* of what it created, that this memory can be wrong, and that being wrong has real consequences. Budget the time for the "delete the state file and watch what happens" demo at 12:15; it is the single highest-value ten minutes of the day, and skipping it means half the room never really gets it. <br>
**END OF NOTE**

### How this document is laid out

**Terraform is directory-scoped** — being in the wrong folder genuinely changes what happens, because each folder is an independent project with its own state. So every command block is labelled:

- *(Run from `~/terraform-training/01-terraform-basics`)* — a **terminal** command, in that exact folder
- *(In the Azure Portal — Resource groups)* — a **browser** action, from that screen

The folders we build today:

| Folder | What it's for |
|---|---|
| `~/terraform-training` | Parent folder. The `.gitignore` lives here |
| `~/terraform-training/01-terraform-basics` | Resource group, storage account, first AD user, state, outputs |
| `~/terraform-training/02-count` | Creating many resources with `count` |
| `~/terraform-training/03-lists-and-sets` | `length()`, `for_each`, sets vs lists |
| `~/terraform-training/04-maps` | Maps, nested maps, richer attributes |

**Every activity has a `**Solution**` block** immediately afterwards.

### Duration & schedule

Full day, **09:00–17:00**, with a one-hour lunch and two 15-minute breaks.

**Hands-on time today: ~3 hours 10 minutes** across seven activities, every one with a solution in this document.

| Time | Section | Activity time |
|---|---|---|
| 09:00–09:15 | Welcome & recap: the ClickOps problem, solved | |
| 09:15–10:00 | What Terraform is, and the three states | |
| **10:00–10:15** | **Break** | |
| 10:15–11:15 | Setup: install, Service Principal, credentials, `init` | **30 min** |
| 11:15–12:15 | Your first resources: resource group & storage account | **25 min** |
| 12:15–13:00 | State, the console, and outputs | **20 min + challenge** |
| **13:00–14:00** | **Lunch** | |
| 14:00–14:45 | Refactoring, `.gitignore`, and variables | **20 min** |
| 14:45–15:00 | Creating many resources with `count` | |
| **15:00–15:15** | **Break** | |
| 15:15–16:45 | Capstone: lists, sets, `for_each` and maps | **90 min** |
| 16:45–17:00 | Wrap-up, destroy, & Q&A | **10 min** |

### Set-up

#### For Trainers

**For the slidee presentation** <br>
**Open** the [Slidee](https://github.com/mklilley/slidee) presentation:

- Run `npx slidee` from within the `curriculum` folder to see the available slide decks (those with extension `.slides.md`)
- Click on `See your presentations`
- Open `Weeks 6 7 > CDO > Infrastructure as code with terraform > Terraform`
- Hit `s` to open the presenter notes
- Resource to be kept open on a tab throughout lecture to refer to

**Check out** [FAQs](/FAQs/README.md) for help with using Slidee

#### For Students
- **Direct** students to the starter code for this module
  - [Student Facing Repo](https://github.com/LaFosseAcademy/cloud-devops-student-repos/tree/main)
- **Use**
  - **/infrastructure-as-code-terraform/intro-to-infrastructure-as-code/starter-code**
- **Make sure**, *before the session starts*, every student has:
  - An **Azure account** with an active subscription
  - The **Azure CLI** installed — `az --version` responds and they can `az login`
  - **VS Code**, ideally with the HashiCorp Terraform extension
  - **Their `my-azure-details.md` from Session 1**, including their **tenant domain**
  - Permission to **create a Service Principal** in their tenant (see the trainer note below)

**NOTE FOR TRAINERS — the three things that will eat your morning** <br>
**(1) Service Principal permissions.** On a personal free-trial subscription students are the tenant's Global Administrator and `az ad sp create-for-rbac` just works. On a **corporate** tenant it very often does not — creating app registrations may be restricted. Test this yourself in the target environment beforehand. Fallback: students authenticate with plain `az login` and let the `azurerm` provider use that session, which works for everything today except demonstrating the Service Principal pattern — which you can then demo once from your own machine. <br>
**(2) Tenant domains.** Everyone's differs. Your demo will use yours; theirs must use their own. See the callout at 12:20 — if they copy your domain, the AD user creation fails. <br>
**(3) Globally-unique storage account names.** Have them settle on a personal prefix at the start and use it all day. <br>
**END OF NOTE**

## Learning objectives

- **Explain** what Infrastructure as Code is and why it beats clicking in a portal
- **Distinguish** Terraform's three states — Desired, Known and Actual — and explain what `terraform.tfstate` is for
- **Write** HCL resource blocks, reference one resource from another, and define outputs
- **Run** the core workflow: `init`, `plan`, `apply`, `destroy`, plus `validate`, `fmt`, `show` and `console`
- **Authenticate** Terraform to Azure using a Service Principal and environment variables
- **Refactor** a configuration into `main.tf`, `resources.tf` and `outputs.tf`, and protect state with `.gitignore`
- **Use** variables, and understand the precedence order of the ways to set them
- **Create** many resources from one block using `count`, and explain why `for_each` is usually better
- **Compare** Terraform against the other IaC tools they'll meet in the wild

<br>

## Sequence

### 09:00–09:15 — Welcome & Recap: The ClickOps Problem, Solved

Morning. Today is the payoff for a complaint we've been making since day one.

Cast your mind back to **Session 1**. We built a resource group and a storage account by hand in the Portal, and then I asked you to recreate it identically in a fresh subscription. The honest answer was "from memory, slowly, probably getting something slightly wrong." We called that **ClickOps** and listed its problems: it wasn't repeatable, no record of *why*, environments drift, no review process, and the knowledge lives in one person's head.

We saw that the answer to these problems was **Git**.

In **Session 2** started writing that knowledge down as scripts. **Session 3** had those scripts run automatically, gated by tests, on every push.

The missing piece is this: **your scripts told the computer the steps. Terraform lets you describe the result.** You don't write "create a storage account, then check whether it already exists, then maybe update it" — you write "there should be a storage account, configured like this", and Terraform works out what needs doing to match that description.

Today you'll go from an empty folder to a configuration that provisions real Azure resources, and you'll understand the one concept that trips up nearly everyone learning Terraform: which is **state**.

**Two housekeeping things.**

First, a standing rule: **anything we create, we destroy before we leave.** Cloud resources cost money.

Second — **have your `my-azure-details.md` from Session 1 open.** You'll need your subscription ID, and critically your **tenant domain**, later this morning. If you didn't record it, we'll get it again in a moment, but this is why we wrote it down.

**Everyone set up now:**

*(Run from `~/`)*
- Run: `mkdir -p ~/terraform-training` → **cd inside** with `cd ~/terraform-training`

**💬 SLACK — snippet 1**, post at 09:05:
```bash
mkdir -p ~/terraform-training && cd ~/terraform-training

az --version                                         # Azure CLI installed?
az account show --query id -o tsv                    # signed in? subscription ID?
az account show --query tenantDefaultDomain -o tsv   # YOUR tenant domain — you need this today
terraform version                                    # probably fails — we install at 10:15
```

- `tsv` is **"tab-seperated values"** removes quotes from output

<br>
<br>

### 09:15–10:00 — What Terraform Is, and the Three States

**Terraform** is an **Infrastructure as Code** tool. Its job is to create, change and delete infrastructure — virtual machines, load balancers, storage, databases, users — based on configuration files you write and keep in Git.

- `SLIDE ACROSS`

**Where it sits in the toolchain.** Terraform lives at the **Provision Server** step. It's excellent at *bringing infrastructure into existence*. It offers basic tooling for configuring what's *inside* a server, but that's not its job — installing and configuring software is the territory of **configuration management** tools like Ansible, Chef and Puppet.

**ASK** <br>
Why do you think there maybe a split? Why not one tool that creates a VM *and* installs everything on it? <br>
**ANSWER** <br>
Different-shaped problems on different timescales. Infrastructure changes rarely and needs careful, reviewable, all-or-nothing changes. What's installed *inside* a machine changes constantly. Keeping them separate means you can redeploy an application fifty times a day without touching the VM definition — and the two concerns can be owned, reviewed and versioned independently. It's the same instinct as separating your database schema from your application code.

**ASK** <br>
Another question to solidify the reason we use Terraform. We provisioned a storage account by hand in Session 1. Why is doing it in Terraform better? <br>
**ANSWER** <br>
- Removes human error — written down and reviewed, rather than re-clicked from memory
- Fast to stand resources up in a new region or subscription — change one value, re-run
- Just as fast to tear them *down*, which directly controls cost
- It's a text file, so you work in the editor and Git workflow you already know, with history and pull requests

#### The concept everything depends on: three states

- `SLIDE ACROSS`

This is the most important ten minutes of the day. Terraform constantly reconciles **three** different pictures of your infrastructure.

**1. Desired State** — what you *want*. Your `.tf` files. "There should be a storage account called X, in resource group Y, with versioning on."

**2. Actual State** — what genuinely exists in Azure right now. If someone clicked something in the Portal, this is where that change lives.

**3. Known State** — what Terraform *believes* it created last time. Stored in a file called `terraform.tfstate`.

When you run `terraform apply`, Terraform reads Desired State, checks Known State to find which real Azure resources it's responsible for, refreshes against Actual State, works out the difference, and makes the minimum changes needed.


That third one seems redundant. If Terraform can just go and *look* at what's really in Azure, why remember what it did last time? <br>

Three reasons, and we'll see all three today. **(1) Mapping** — your config calls something `my_storage_account`, a name we provide for a resource, but Azure calls it a long resource ID. The state file is the **lookup table** joining those two names; without it Terraform has no idea which Azure resource your block refers to, so it assumes it must create a new one. **(2) Metadata and dependencies** — it records the order things were created in, so it can destroy them in the right order. Then we have **(3) Performance** — asking Azure about every resource on every run is slow. State acts as a cache.

Hold onto that. We'll break it deliberately later this morning and watch what happens.

#### Declarative, not imperative

One more framing. Your bash scripts in Session 2 were **imperative** — a list of steps, in order, that you executed. Terraform is **declarative** — you describe the end result and it figures out the steps.

The practical difference: tell Terraform "I want 3 users" when 2 exist, and it creates **one**. You never wrote "add one more". You stated the desired total, and Terraform did the subtraction.




Terraform is not the only IaC tool.<br>

We also have <br>
- **ARM templates** — Azure's original, JSON-based and famously verbose
- **Bicep** — Microsoft's much nicer DSL (which is Domain Specific Language) that compiles to ARM. Azure-only, but excellent if you're Azure-only
- **AWS CloudFormation** — the AWS equivalent
- **Pulumi** — same idea, but you write it in real TypeScript/Python/Go rather than a DSL
- **Ansible** — primarily configuration management, but can provision too

Terraform's advantage is that it's **cloud-agnostic** and has by far the widest provider ecosystem — the same tool and the same mental model for Azure, AWS, GCP, GitHub, Cloudflare and Datadog. That's why it's the most transferable one to learn.

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>

### 10:15–11:15 — Setup: Install, Service Principal, Credentials, `init`
*(Activity: 30 min)*

Four steps: install Terraform, create a Service Principal, expose its credentials as environment variables, and initialise a project.

#### Step 1 — Install Terraform

*(Run from `~/terraform-training`)*

**Mac:**
```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
terraform --version
```

`brew tap hashicorp/tap` adds HashiCorp's own package repository to Homebrew — a "tap" is a third-party source of formulae. Without it, Homebrew doesn't know Terraform exists.

**Windows:**
```bash
choco install terraform
terraform --version
```

Full instructions: [Install Terraform](https://developer.hashicorp.com/terraform/tutorials/azure-get-started/install-cli).

**ASK** <br>
From Session 2 — where will Homebrew or Chocolatey put that binary, and how will your shell find it? <br>
**ANSWER** <br>
Somewhere on your **`PATH`** — `/usr/local/bin` or `/opt/homebrew/bin` on Mac, a Chocolatey folder on Windows. Confirm it with `which terraform`. It's exactly what you did by hand with `~/bin` last session, just done for you by the installer. If `terraform --version` fails immediately after a successful install, it's almost always that the terminal needs restarting to pick up the new PATH.

#### Step 2 — Create a Service Principal

We need Terraform to authenticate to Azure **without a human sitting there**. That's exactly what a **Service Principal** is — you made one in Session 1: an identity in Azure AD representing an application or automated process rather than a person.

*(Run from `~/terraform-training`)*
```bash
az login
az account show --query id -o tsv
```

Now create it:

*(Run from `~/terraform-training`)*
```bash
az ad sp create-for-rbac --name "terraform-training-sp" --role="Contributor" --scopes="/subscriptions/<your-subscription-id>"
```

To break down that command again:
- `az ad sp` — Azure AD, Service Principal
- `create-for-rbac` — create it *and* assign a role in one go
- `--role="Contributor"` — can create, change and delete resources, but **cannot grant permissions to others**. Exactly what Terraform needs
- `--scopes` — where that role applies. Here, the whole subscription


It returns JSON. Three fields matter:

| Returned field | What Terraform calls it |
|---|---|
| `appId` | Client ID |
| `password` | Client Secret |
| `tenant` | Tenant ID |

**Copy these into `my-terraform-notes.md` right now.** The `password` is shown **once and never again** — lose it and you generate a new secret, and the old one stops working.

**ASK** <br>
In Session 1 you gave a Service Principal `Reader`. Today it's `Contributor`. Why the upgrade, and why not `Owner`? <br>
**ANSWER** <br>
`Reader` can only look — Terraform must **create and delete** things, so it needs `Contributor`. But it deliberately stops short of `Owner`, because Owner can **grant permissions to other identities**. A compromised `Contributor` credential can wreck your resources; a compromised `Owner` credential can quietly give an attacker permanent access to everything and then cover its tracks. That's the least-privilege principle from Session 1, applied to a real decision: **pick the weakest role that still does the job.**


#### Step 3 — Expose the credentials as environment variables

It's no good have Terraform configuration without Azure knowing which acount to created resources for. So we need to provide a way to tell Terraform, I want these resources but for this specific account. 

The `azurerm` "Azure Resource Mananger" provider *can* read credentials directly from a `provider` block in your `.tf` file — and HashiCorp's docs explicitly warn you not to.

Let's look at some genuine HCL (HashiCorp Controller Language)

- *Scroll down to third warning*

[Credentials Directly Inserted](https://registry.terraform.io/providers/Azure/azapi/latest/docs/guides/service_principal_client_secret)


The first thing to say is we write HTML into a **.html** file, JavaScript into a **.js** file and our Terraform configuration into a **.tf** file. 

Where **index.js** is the normal entry point for JavaScript. **Main.tf** is the equivalent for Terraform. 

From the configuration we can see, inside the different curly braces we can provide a lot of base configuration. 
- Which cloud service provider
- Which version
- Also our credentials 

**ASK** <br>
Why is hard-coding those four values into `main.tf` genuinely dangerous, rather than just untidy? <br>
**ANSWER** <br>
Because `main.tf` goes into Git, and **Git history is permanent** — deleting the line later doesn't remove it from history. A client secret committed to a repository is a real security incident, and the standard advice once it's happened is to assume it's compromised and rotate it. It's the same lesson as the Jenkins credentials store in Session 3: **secrets live outside the code that uses them.** The mechanism differs — environment variables here, a credentials vault there — but the principle is identical.

So we use **environment variables**. The provider looks for four specific names automatically.


Run these commands in **VS Code's integrated terminal**,open up the `./terraform-training` folder in VSCode, open a terminal there and run Terraform from that same terminal. Environment variables set in one terminal don't exist in another<br>


- `SLIDE ACROSS`

So run these commands from your Terminal in VSCode, subbing in the placeholders with the information you saved earlier. 

*(Run from `~/terraform-training`, in VS Code's terminal — Mac/Linux)*
```bash
export ARM_CLIENT_ID=<appId>
export ARM_CLIENT_SECRET='<password>'
export ARM_SUBSCRIPTION_ID=<your-subscription-id>
export ARM_TENANT_ID=<tenant>
```

**Note the single quotes around the secret.** Client secrets frequently contain `~`, `!` or `$`, which bash would otherwise interpret — that's the quoting lesson from Session 2, biting for real.


To make them permanent on Mac/Linux, add those four lines to `~/.zshrc` or `~/.bashrc` — the file that runs every time a shell opens. Same trick you used to put `~/bin` on your `PATH`.

- `SLIDE ACROSS`

**.zshrc**
```
export ARM_CLIENT_ID="2b1e4c6a-8f3d-4a1b-9c7e-5d2f1a3b4c5d"
export ARM_CLIENT_SECRET="Qx9~vK2pL.8mN4rT7wZ1uY6bA3sC0dEf"
export ARM_SUBSCRIPTION_ID="7c3a9e21-1b4d-4f6a-9c8e-2d5f7a1b3c6e"
export ARM_TENANT_ID="9f4b1c2a-3d5e-4a6b-8c7d-1e2f3a4b5c6d"
```


#### Step 4 — Create a project and initialise it

Terraform uses configuration files ending in **`.tf` as we said**.

*(Run from `~/terraform-training`)*
- Run: `touch .gitignore` and copy the **.gitignore** from the resources folder int he student facing repo into it
- Run: `mkdir 01-terraform-basics` → **cd inside**

*(Run from `~/terraform-training/01-terraform-basics`)*
- Run: `touch main.tf`
- Then: `code main.tf`

**main.tf**
```tf
# Declares the beginning of the terraform configuration block
terraform {
  # Used to specify the required providers and their versions
  required_providers {
    # Defines the required provider is Azure
    azurerm = {
      # Defines the source of the Azure provider, which is HashiCorp
      source  = "hashicorp/azurerm"
      # Use any 3.x version
      version = "~> 3.0"
    }
  }
}

# Start of config block for the Azure provider
provider "azurerm" {
  # Required (but empty) features block for the azurerm provider
  features {}
}
```

Your first HCL, and every line does something:

- `terraform { }` — settings **about Terraform itself**, not about your infrastructure
- `required_providers { }` — which plugins this project needs. A **provider** is the piece that knows how to talk to a specific API. Azure, AWS, GitHub, Cloudflare, Datadog — all separate providers
- `azurerm = { ... }` — `azurerm` is the local name we'll use in resource types. Azure **R**esource **M**anager
- `source = "hashicorp/azurerm"` — where to download it from in the Terraform Registry
- `version = "~> 3.0"` — the **pessimistic constraint operator**: "3.0 or higher, but below 4.0"
- `provider "azurerm" { }` — configuration *for* that provider once downloaded
- `features {}` — an Azure-provider quirk: mandatory, and usually empty. It exists so Azure can add opt-in behaviours later



**Take a look at what else exists.** Open the [Terraform Registry](https://registry.terraform.io/browse/providers). Terraform isn't an Azure tool — it's a *provisioning* tool with hundreds of providers. Bookmark it; it's the documentation you'll live in.

Installing Terraform did **not** give you the Azure provider. You download it per-project:

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform init
```

The output says it has:
- **Initialised the backend** — where state will be stored. Right now, a local file
- **Initialised provider plugins** — downloaded `azurerm` at a version matching your constraint

Two new things appeared:

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
ls -la
du -sh .terraform # Shows Disk space it's using
```

- **`.terraform/`** — a hidden folder holding the downloaded provider binary. When I ran `du -sh`, notice it's **hundreds of megabytes**. Every Terraform project you init downloads its own copy
- **`.terraform.lock.hcl`** — the dependency lock file

**ASK** <br>
A hidden folder of hundreds of MB of downloaded dependencies, plus a lock file pinning exact versions. What does this remind you of? <br>
**ANSWER** <br>
**`node_modules` and `package-lock.json`** — and the parallel is essentially exact. `.terraform/` is `node_modules`: downloadable, disposable, gitignored, and enormous. `.terraform.lock.hcl` is `package-lock.json`: it pins the exact resolved versions so every developer and every CI pipeline run gets the same thing, and **it should be committed**. That's why our `.gitignore` excludes `.terraform/` but not the lock file.

**HANDS ON (30 min)** <br>

I want you to follow these steps yourself or make notes so you can repeat the process yourself independantly. 

Part A *(15 min)* — install and authenticate.
1. Install Terraform, confirm `terraform --version`, and run `which terraform`
2. `az login`, and note your **subscription ID**
3. Create a Service Principal named `terraform-training-sp` with `Contributor` on your subscription. **Save `appId`, `password` and `tenant` into `my-terraform-notes.md`**
4. Export all four `ARM_*` variables in VS Code's terminal, and confirm with `export | grep ARM`

Part B *(10 min)* — first project.
5. Copy the `.gitignore` into `~/terraform-training`
6. Create `01-terraform-basics/main.tf` with the provider config, and run `terraform init`
7. Run `ls -la` and `du -sh .terraform`. How big is it?

Part C *(5 min)* — research.
8. Open the **Terraform Registry** and find **three** providers that have nothing to do with Azure. Note what each one manages. We'll compare as a room
**END OF NOTE**

**💬 SLACK — snippet 2**, post at the start:
```bash
# Install (Mac)
brew tap hashicorp/tap && brew install hashicorp/tap/terraform

# Install (Windows)
choco install terraform

terraform --version

# Authenticate and get your subscription ID
az login
az account show --query id -o tsv

# Create the Service Principal — paste YOUR subscription id in
az ad sp create-for-rbac \
  --name "terraform-training-sp" \
  --role "Contributor" \
  --scopes "/subscriptions/<your-subscription-id>"
```

**💬 SLACK — snippet 3**, post once people have their Service Principal:
```bash
# Paste YOUR values in. Note the SINGLE QUOTES around the secret —
# it often contains ~ ! or $ which bash would otherwise interpret.

export ARM_CLIENT_ID=<appId>
export ARM_CLIENT_SECRET='<password>'
export ARM_SUBSCRIPTION_ID=<subscription-id>
export ARM_TENANT_ID=<tenant>

export | grep ARM        # confirm all four are set
```

**💬 SLACK — snippet 4**, the provider block:
```tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}
```

**Solution**

*(Run from `~/terraform-training`)*
```bash
# 1
brew tap hashicorp/tap && brew install hashicorp/tap/terraform
terraform --version
which terraform          # -> /opt/homebrew/bin/terraform  (Apple Silicon)
                         #    /usr/local/bin/terraform      (Intel Mac)

# 2
az login
az account show --query id -o tsv

# 3
az ad sp create-for-rbac --name "terraform-training-sp" --role "Contributor" \
  --scopes "/subscriptions/a755c4aa-edd0-4c67-83d9-bc7adc18bb18"
```

Which returns:
```json
{
  "appId": "2b1e4c6a-8f3d-4a1b-9c7e-5d2f1a3b4c5d",
  "displayName": "terraform-training-sp",
  "password": "Qx9~vK2pL.8mN4rT7wZ1uY6bA3sC0dEf",
  "tenant": "bff07bc0-bc49-46ca-9f9f-6ce7c451f2a7"
}
```

*(Run from `~/terraform-training`, in VS Code's terminal)*
```bash
# 4
export ARM_CLIENT_ID=2b1e4c6a-8f3d-4a1b-9c7e-5d2f1a3b4c5d
export ARM_CLIENT_SECRET='Qx9~vK2pL.8mN4rT7wZ1uY6bA3sC0dEf'
export ARM_SUBSCRIPTION_ID=a755c4aa-edd0-4c67-83d9-bc7adc18bb18
export ARM_TENANT_ID=bff07bc0-bc49-46ca-9f9f-6ce7c451f2a7
export | grep ARM

# 5, 6
cp <starter-repo>/.gitignore .
mkdir 01-terraform-basics && cd 01-terraform-basics
```

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
touch main.tf
# ...paste the provider config...
terraform init

# 7
ls -la
du -sh .terraform        # typically 300-700 MB
```

**Step 8 — providers worth surfacing from the room.** Ask two or three people what they found:

| Provider | Manages |
|---|---|
| `hashicorp/aws` | Every AWS service |
| `integrations/github` | Repos, teams, branch protection rules, webhooks |
| `cloudflare/cloudflare` | DNS records, firewall rules, CDN config |
| `datadog/datadog` | Monitors, dashboards, alerts |
| `hashicorp/kubernetes` | Kubernetes objects (Session 6) |
| `grafana/grafana` | Dashboards and data sources |

**The GitHub one is worth pausing on**, because it lands hardest with this room: you can manage your **repository settings** — who has access, which branches are protected, which reviews are required — as Terraform code, reviewed through a pull request. The rules governing your pull requests, themselves defined in a pull request. That's the "everything as code" idea taken to its logical end.

**Troubleshooting:**

| Symptom | Cause |
|---|---|
| `terraform: command not found` after install | Restart the terminal to pick up the new PATH |
| `az ad sp create-for-rbac` → authorisation error | Corporate tenant restricting app registrations. Fall back to `az login` auth |
| `terraform init` → provider download fails | Corporate proxy or firewall. Check `HTTP_PROXY` |
| Init succeeds but later `apply` says "building AzureRM client" | One of the four `ARM_*` variables is missing or set in a different terminal |

<br>
<br>

### 11:15–12:15 — Your First Resources: Resource Group & Storage Account
*(Activity: 25 min)*

Time to create something real.

#### Creating a Resource Group

Before you can create almost anything in Azure, you need somewhere to put it — a **Resource Group**.

**ASK** <br>
From Session 1 — why does Azure have this extra layer? <br>
**ANSWER** <br>
A single unit for managing lifecycle, permissions, cost tracking and clean-up for a related set of resources. Deleting a resource group deletes everything inside it — which is exactly why you group by **shared lifecycle** rather than convenience.

*(Run from `~/terraform-training/01-terraform-basics`)* — edit `main.tf`:

**main.tf**
```tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

# NEW CODE
resource "azurerm_resource_group" "my_resource_group" {
  name     = "rg-emilesherrott-devops"
  location = "uksouth"
}
```

**The resource block syntax — learn this shape; everything else today is a variation:**

- `SLIDE ACROSS`

```tf
resource "<provider>_<resource_type>" "<internal_name>" {
  <attribute> = <value>
}
```

Three parts, and the distinction between the last two matters enormously:

1. `resource` — the keyword. "I want to manage a thing in the cloud"
2. `"azurerm_resource_group"` — the **type**. Provider name, underscore, resource type. **Fixed vocabulary defined by the provider** — you can't invent it, you look it up in the registry docs
3. `"my_resource_group"` — the **internal name**, sometimes called the local name or label. **You choose this.** It exists only inside Terraform. Nothing in Azure will ever be called this

Then the attributes. Note `name = "rg-emilesherrott-devops"` — *that's* what Azure sees.

**ASK** <br>
So we have two names. What's the difference between `"my_resource_group"` and `name = "rg-emilesherrott-devops"`? <br>
**ANSWER** <br>
`my_resource_group` is Terraform's **internal handle** — how your *other configuration* refers to this resource. `name` is what actually gets created in Azure and appears in the Portal. This distinction becomes critical in about an hour when we look at the state file, because Terraform uses the **internal name as its lookup key**. Think of it as the difference between a variable name in your code and the value it holds.

#### Creating a Storage Account

**ASK** <br>
What's Azure's equivalent of an S3 bucket? No don't expect you to know what an S3 Bucket is but these are common terms in DevOps so give both a Google or use AI.<br>
**ANSWER** <br>
They're object storage, a bit similar to DropBox. <br>
Azure splits it into two: a **Storage Account**, the top-level container, and a **Blob Container** inside it, where the actual files (blobs) live.

Let's create one in the Portal first, just to see what we're automating.

*(In the Azure Portal — home)*
- **Create a resource → Storage account**
- Try the name `storage` — you'll be told it already exists

Storage account names must be **globally unique across all of Azure**, additionally they must be **lowercase letters and numbers only**, no hyphens, no underscores, 3–24 characters.

- Use something like `stemilesherrottdevops`
- Choose Primary Service: `Azure Blob Storage or Azure Data Lake Storage` leave the rest default, **Review + create** → **Create**


Click into it, then **Data storage -> Containers** — that's where files would go. If we were Netflix, we could upload a season of Squid Games or whatever and then our App could pull the files from there. <br>
Now let's do the same thing properly, in code.

*(Run from `~/terraform-training/01-terraform-basics`)* — add to `main.tf`:

**main.tf**
```tf
[ . . . ]

# NEW CODE
resource "azurerm_storage_account" "my_storage_account" {
  name                     = "stemilesherrottdevops01"
  resource_group_name      = azurerm_resource_group.my_resource_group.name
  location                 = azurerm_resource_group.my_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```

Two lines here are doing something genuinely new:

```tf
resource_group_name = azurerm_resource_group.my_resource_group.name
location            = azurerm_resource_group.my_resource_group.location
```

That's a **reference**. Instead of typing `"rg-emilesherrott-devops"` again, we point at the *other resource block* and read its attribute:

```
<resource_type>.<internal_name>.<attribute>
```

Note there are **no quotes** — quotes would make it a literal string. This is an expression, so it's bare.

**ASK** <br>
Why is referencing better than typing the name string? <br>
**ANSWER** <br>
Two reasons. **(1) Single source of truth** — rename the resource group in one place and everything following it updates. **(2) Dependency ordering**, which is the big one. By referencing it you've told Terraform "this storage account **depends on** that resource group", so Terraform builds a dependency graph and creates the resource group **first**. You never specify the order.


Remaining attributes: `account_tier = "Standard"` (versus Premium, SSD-backed and pricier), and `account_replication_type = "LRS"` — **L**ocally **R**edundant **S**torage, three copies within one datacentre. Cheapest, fine for training. `GRS` replicates to the paired region — the disaster-recovery idea from Session 1.

#### The two-step execution approach

Terraform has a sensible workflow: **check what would happen, then do it.**

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform plan
```

`plan` compares Desired against Known (refreshing against Actual) and tells you exactly what it *would* do, without doing anything. Read the output together:

- `+` create, `-` destroy, `~` update in place
- Lots of attributes show `(known after apply)` — `primary_access_key`, `primary_blob_endpoint`, `id`. Terraform can't know those until Azure has made the thing
- The only values it *can* show are the ones you wrote

Now execute:

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform apply
```


`apply` shows the plan again, then stops and asks for confirmation. At the bottom: **Plan: 2 to add, 0 to change, 0 to destroy**. The only input it accepts is the literal word `yes`.

- Type: `yes`

If that worked, your credentials are correct and Terraform has real access to your subscription.

*(In the Azure Portal — Resource groups)*
- Open **rg-emilesherrott-devops** and refresh. Your storage account is there


**NOTE FOR TRAINERS** <br>
Failures here are almost always one of two things: **credentials** (an `ARM_*` variable missing, or set in a different terminal) or a **non-unique storage account name**. Have students check `export | grep ARM` first, then try a more obscure name. <br>
**END OF NOTE**

- `SLIDE ACROSS`

**HANDS ON (25 min)** <br>
*(Run from `~/terraform-training/01-terraform-basics`)*
1. Add the resource group and storage account blocks — use **your own** name prefix, not mine
2. Run `terraform plan` and read it properly. Find **three** attributes marked `(known after apply)` and explain why Terraform can't know them yet
3. Run `terraform apply`, confirm with `yes`
4. *(In the Azure Portal)* Verify both resources exist
5. Run `terraform apply` a **second** time, with no changes. What does it say, and why?
6. **Break the reference on purpose:** change `resource_group_name` to the literal string `"rg-yourname-devops"` instead of the reference. Run `terraform plan`. Does it still work? Now think about what you've lost
**END OF NOTE**

**💬 SLACK — snippet 5**:
```tf
resource "azurerm_resource_group" "my_resource_group" {
  name     = "rg-CHANGEME-devops"
  location = "uksouth"
}

resource "azurerm_storage_account" "my_storage_account" {
  name                     = "stCHANGEMEdevops01"
  resource_group_name      = azurerm_resource_group.my_resource_group.name
  location                 = azurerm_resource_group.my_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```
*(Storage account names: lowercase + numbers only, 3–24 chars, globally unique.)*

**Solution**

**main.tf**
```tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "my_resource_group" {
  name     = "rg-jbloggs-devops"
  location = "uksouth"
}

resource "azurerm_storage_account" "my_storage_account" {
  name                     = "stjbloggsdevops01"
  resource_group_name      = azurerm_resource_group.my_resource_group.name
  location                 = azurerm_resource_group.my_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform plan
terraform apply
# type: yes

terraform apply     # a second time
```

**Step 2 answer:** `id`, `primary_access_key` and `primary_blob_endpoint` are all generated **by Azure** at creation time. Terraform is planning *ahead* of that happening, so it honestly reports it doesn't know them yet. Anything Azure generates — IDs, keys, endpoints, IP addresses — falls into this category.

**Step 5 answer:** **"No changes. Your infrastructure matches the configuration."** Desired and Known agree, and refreshing shows Actual agrees too — nothing to do. That's the declarative model working: **running `apply` repeatedly is safe.** That property has a name — **idempotency** — and it's the same property that made your bash scripts safe to re-run in Session 2.

**Step 6 answer — the important one.** The `plan` still works and produces the same result, so nothing *looks* broken. What you've lost is the **dependency edge**. Terraform no longer knows the storage account depends on the resource group, so:
- It may try to create them **in parallel**, and the storage account creation fails because its resource group doesn't exist yet
- On `destroy` it may try to delete the resource group **first**, while the storage account is still in it

The failure is **intermittent and order-dependent**, which makes it far worse than a clean error. Put the reference back. The lesson: **references aren't just about avoiding repetition — they're how you declare dependencies.**

<br>
<br>

### 12:15–13:00 — State, the Console, and Outputs
*(Activity: 20 min + challenge)*

Now the important bit.

#### Changing something that can't be changed

*(Run from `~/terraform-training/01-terraform-basics`)* — change the storage account name:

**main.tf**
```tf
resource "azurerm_storage_account" "my_storage_account" {
  # UPDATED CODE
  name                     = "stemilesherrottdevops02"
  [ . . . ]
}
```

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform apply
```


Read the output carefully: **destroy and then create replacement**, and **`must be replaced`**. Summary: **1 to add, 0 to change, 1 to destroy**.

Terraform knows a storage account's name **cannot be changed after creation** — an Azure constraint, encoded into the provider. The only way to reach your Desired State is to delete the old one and make a new one.

- Type: `yes`

**ASK** <br>
Why does it matter that Terraform knows *which* attributes force a replacement rather than an in-place update? <br>
**ANSWER** <br>
Because a replacement is **destructive**. On a storage account holding real data, or a database, "destroy and recreate" is potentially catastrophic — and the plan output is your only warning before it happens. This is a very strong argument for always running `plan` and actually **reading** it, especially in production. It's the same reason you read a migration before running it against a live database.

We can see this in the documentation as well. 

[Documentation](https://registry.terraform.io/providers/hashicorp/azurerm/1.43.0/docs/resources/storage_account)

If we scroll down we can see the **Argument Reference** and under **name** it has the text:
- `Required`, so we need to have that key value present
- Also *"Changing this forces a new resource to be created"* <br>
Which is what we've just experienced. 

#### Enabling versioning — a property, not a resource

*(In the Azure Portal — your storage account)* — look at **Data protection** under **Data management**. **Blob versioning** is disabled.

I'm showing you this purely to demo how terraform syntax is constructed but it may be something we want to enable if we want to retain older versions of the files we save. 

**main.tf**
```tf
resource "azurerm_storage_account" "my_storage_account" {
  [ . . . ]

  # NEW CODE
  blob_properties {
    versioning_enabled = true
  }
}
```

Notice `blob_properties { }` has **no `=`**. It's a **nested block**, not an attribute:
- `name = "value"` — an **attribute**: a single named value, uses `=`
- `blob_properties { ... }` — a **block**: a group of related settings, no `=`, uses braces

Using the documentation, can you tell me whether this will force a new resource or can we change the existing one. 

- *Share*: https://registry.terraform.io/providers/hashicorp/azurerm/1.43.0/docs/resources/storage_account

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform apply
# type: yes
```

This time it's an **update in place** — `~` rather than `-/+`. Versioning *can* be changed on a live account, so nothing gets destroyed.

Here's the declarative point again: **you never wrote "enable versioning".** You declared that versioning should be enabled, and Terraform worked out that the way to get there was an update call.

**NOTE FOR TRAINERS** <br>
Occasionally `terraform.tfstate` lags behind a configuration-only change. If Known State looks stale, `terraform refresh` pulls the latest Actual State from the Azure API. Worth mentioning so nobody panics. <br>
**END OF NOTE**

#### The terraform console

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform console
```

An interactive **REPL** which stands for **"Read-Evaluate-Print Loop"**, basically for querying your configuration and state — like if we run `node` in a terminal, but for Terraform expressions.

```
azurerm_storage_account.my_storage_account
```


Every attribute of the storage account. The syntax is a **reference**, same as the one linking your two resources:

```
<provider>_<resource-type>.<chosen_resource_name>
```

Critically, you use the **internal name** (`my_storage_account`), *not* the Azure name. **Terraform doesn't think in Azure names.**

This looks very similar to digging into a JavaScript object. 

Dig deeper:
```
azurerm_storage_account.my_storage_account.blob_properties
```


It comes back wrapped in `[ ]` — it's a **list**. Nested blocks can appear multiple times, so the provider models them as a list even when there's one. Index into it, exactly like a JavaScript array:

**ASK**<br>
How would we do that if it were an array? <br>
**ANSWER** <br>
```
azurerm_storage_account.my_storage_account.blob_properties[0]
```


**ASK** <br>
How would we get just the value of `versioning_enabled`? <br>
**ANSWER** <br>
`azurerm_storage_account.my_storage_account.blob_properties[0].versioning_enabled` — index into the list, then read the attribute. **Exactly the same chaining as JavaScript**, and `terraform console` is doing the same job as a browser console or `node` REPL: letting you poke at a live object graph to find out what's actually in it.

Exit with `Ctrl + C` (or `exit`).

#### Outputs

The console is great for exploring. For a value you want *every time*, define an **output**.

- `SLIDE ACROSS`

```tf
output "<output-name>" {
  value = <expression>
}
```

**main.tf**
```tf
# NEW CODE
output "my_storage_account_versioning" {
  value = azurerm_storage_account.my_storage_account.blob_properties[0].versioning_enabled
}
```

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform apply -refresh=false
```


`-refresh=false` says **don't ask Azure what's really there** — just compare Desired against Known. We know they match and we only added an output, so skipping the refresh makes it fast.

**Be careful with `-refresh=false` in production** — you're deliberately choosing not to check reality, and reality is where your applications live.

You're not limited to one:

**main.tf**
```tf
# NEW CODE
output "my_storage_account_complete_details" {
  value = azurerm_storage_account.my_storage_account
  sensitive = true
}
```

We add the attribute `sensitive` because it may reveal sensitive information and we're letting terraform now we're comfortable with that. 

**ASK** <br>
Outputs are handy for poking around. What do you think they're actually *for* in a real project? <br>
**ANSWER** <br>
Three things. **Passing information onward** — a blob endpoint or DNS name handed to another part of your infrastructure, or another Terraform project. **Debugging** — surfacing key facts to confirm things provisioned correctly. And the big one: **integration with external systems.** A pipeline stage can read `terraform output` and use those values.

#### Adding an Azure AD user

Let's add a resource from a **different provider**, to prove a project isn't limited to one.

**⚠️ THIS IS THE BIT WHERE YOUR VALUE MUST DIFFER FROM MINE.**

An Azure AD user's `user_principal_name` **must be on a domain your tenant owns.** In Session 1 you recorded your **tenant domain** — this is what it was for.

- Mine is `lafosse.com`, a verified custom domain, so my users look like `name@lafosse.com`
- Yours is almost certainly something like `jbloggs.onmicrosoft.com`, so yours look like `name@jbloggs.onmicrosoft.com`

If you copy my domain, you'll get **`Property userPrincipalName is invalid`**. Check your notes now; if you don't have it:

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
az account show --query tenantDefaultDomain -o tsv
```

**main.tf**
```tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
    # NEW CODE
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.0"
    }
  }
}

provider "azurerm" {
  features {}
}

# NEW CODE
provider "azuread" {}

# ... resource group and storage account as before ...

# NEW CODE — REPLACE THE DOMAIN WITH YOUR OWN
resource "azuread_user" "my_azuread_user" {
  user_principal_name = "my_iam_user_abc@emilesherrottgmail.onmicrosoft.com"
  display_name        = "my_iam_user_abc"
  mail_nickname       = "my_iam_user_abc"
  password            = "ChangeMe123!ChangeMe"
}
```

You can see domains which are available on your account with:

```bash
az rest --method get \
  --url "https://graph.microsoft.com/v1.0/domains" \
  --query "value[].id" \
  -o tsv
```

So this could be a process if someone is joining your organisation and you need to set them up with an account.

# ADD

**ASK**<br>
How are we authenticating ourselves in Terraform to provision these resources? <br>
**ANSWER**<br>
A Service Principal we created linked to our Tenant and Subscription

Our Service Principle at the moment has permissions as a Contributor within the AzureRM provider. We've added another privoder however the AzureAD provider which doesn't manage resources per-se but EntraID. 

Before we're able to execute our current configuration we'll need to add some new privilages. 

- In the Azure Portal go to: **EntraID -> App registrations -> All applications** and you should see the one we created earlier. Click into it

- Then go to **Manange -> API permissions -> Click 'Add a permission' -> select Microsoft Graph -> Select Application permissions -> Search for 'User.ReadWrite.All' -> Tick -> Click Add permissions**

Then on the same page we should be able to click: **"Grant admin consent for Default Directory"**


Also because we added a new provider, we also need to re-initialise:

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform init
```


Azure AD enforces password complexity (upper and lowercase, numbers, symbols, min 8 characters). If `apply` rejects it, tweak until it passes. In a real project you'd generate this with the `random_password` resource rather than hard-coding it in a file destined for Git. 


#### Saving a plan to a file

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform plan -out aduser.tfplan
```


`-out <file>` saves the plan. Try opening `aduser.tfplan` in your editor — it's **binary**, not readable text.

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform apply "aduser.tfplan"
```



Notice: **no `yes` prompt.** It just runs.

**ASK** <br>
Applying a saved plan skips the confirmation. Feature or hazard? <br>
**ANSWER** <br>
Both, depending entirely on context. It's a **feature** in a pipeline — and this is a good place to get to: run `plan` in one stage, a human reviews the output, then `apply` **that exact saved plan**, guaranteeing nothing changed between review and execution. It's a **hazard** if you saved a plan an hour ago and the world has moved on, because it won't re-check. The safety of a saved plan comes entirely from it being **fresh and reviewed**.

#### Updating in place, with a target

*(Run from `~/terraform-training/01-terraform-basics`)* — change `abc` to `def` across all three name fields, then:

```bash
terraform apply -target=azuread_user.my_azuread_user
```


- Type: `yes`

`-target` restricts the operation to one resource and its dependencies, saving a full refresh. Useful in a project with hundreds of resources — but **use it sparingly**: you're deliberately ignoring part of your configuration, so state can end up partially applied.

#### The state files, in depth

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
ls -la
```


Two files:
- **`terraform.tfstate`** — the current Known State
- **`terraform.tfstate.backup`** — the *previous* Known State

Open `terraform.tfstate`. Two top-level keys matter: **`outputs`** (your defined outputs) and **`resources`** (everything Terraform manages — type, internal name, current attributes, metadata).

If we compare the two files: `terraform.tfstate` has the AD user as `def`; the backup still says `abc`. Every successful `apply` moves the old state into `.backup`.

**You should never edit these by hand.** So naturally, let's break one on purpose.

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
mv terraform.tfstate terraform.tfstate.001
mv terraform.tfstate.backup terraform.tfstate.backup.001
ls
```

**ASK** <br>
What happens if we run a plan now? <br>
**ANSWER** <br>
Let them guess before revealing.

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform plan
```

It wants to **create everything from scratch**. With no Known State, Terraform believes none of these resources exist — even though they're sitting right there in Azure. The plan is entirely accurate *to Terraform's knowledge*; its knowledge is just gone.

Put them back:

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
mv terraform.tfstate.001 terraform.tfstate
mv terraform.tfstate.backup.001 terraform.tfstate.backup
terraform plan
```

Sanity restored.

**Why this happened — the mapping.**

**terraform.tfstate**
```tf
"resources": [
  {
    "mode": "managed",
    "type": "azuread_user",
    # HERE — your internal name
    "name": "my_azuread_user",
    "provider": "provider[\"registry.terraform.io/hashicorp/azuread\"]",
    "instances": [
      {
        "attributes": {
          # HERE — Azure's identifier for the real object
          "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
          "user_principal_name": "my_iam_user_def@emilesherrottgmail.onmicrosoft.com",
          "display_name": "my_iam_user_def"
        }
      }
    ]
  },
[ . . . ]
```

Terraform finds the resource by its **`name`** (your internal name), then reads the **`id`** to know which real Azure object it maps to. **That's the join.** Without it, `my_azuread_user` means nothing — no way to connect your config block to that GUID (Globally Unique Identifier), or even to know Terraform created it.


- `SLIDE ACROSS`

**HANDS ON (20 min)** <br>
*(Run from `~/terraform-training/01-terraform-basics`)*
1. Change your storage account name and `apply`. Confirm the plan says **must be replaced** and note the add/destroy counts
2. Add the `blob_properties` block with `versioning_enabled = true`, apply, and verify in the Portal
3. Open `terraform console` and retrieve `versioning_enabled` in a **single expression**
4. Add outputs for the versioning flag and the full storage account details, then `terraform apply -refresh=false`
5. **Add an Azure AD user — using YOUR tenant domain.** Re-run `terraform init` first
6. Rename both state files, run `terraform plan`, explain what you see, then rename them back
**END OF NOTE**

**💬 SLACK — snippet 6**:
```tf
# Add to required_providers:
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.0"
    }

# Add after the azurerm provider block:
provider "azuread" {}

# THE DOMAIN MUST BE YOURS. Get it with:
#   az account show --query tenantDefaultDomain -o tsv
resource "azuread_user" "my_azuread_user" {
  user_principal_name = "my_iam_user_abc@YOUR-DOMAIN-HERE"
  display_name        = "my_iam_user_abc"
  mail_nickname       = "my_iam_user_abc"
  password            = "ChangeMe123!ChangeMe"
}
```
*(Then run `terraform init` again — you've added a provider.)*

**Solution**

**main.tf** (steps 1, 2, 4 and 5):
```tf
resource "azurerm_storage_account" "my_storage_account" {
  name                     = "stjbloggsdevops02"          # step 1 -> forces replacement
  resource_group_name      = azurerm_resource_group.my_resource_group.name
  location                 = azurerm_resource_group.my_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "LRS"

  # step 2
  blob_properties {
    versioning_enabled = true
  }
}

# step 5 — jbloggs' OWN tenant domain
resource "azuread_user" "my_azuread_user" {
  user_principal_name = "my_iam_user_abc@jbloggs.onmicrosoft.com"
  display_name        = "my_iam_user_abc"
  mail_nickname       = "my_iam_user_abc"
  password            = "ChangeMe123!ChangeMe"
}

# step 4
output "my_storage_account_versioning" {
  value = azurerm_storage_account.my_storage_account.blob_properties[0].versioning_enabled
}

output "my_storage_account_complete_details" {
  value = azurerm_storage_account.my_storage_account
}
```

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
# Step 1 -> "Plan: 1 to add, 0 to change, 1 to destroy"
terraform apply

# Step 2 -> "Plan: 0 to add, 1 to change, 0 to destroy"  (update in place)
terraform apply

# Step 3
terraform console
```
Inside the console:
```
azurerm_storage_account.my_storage_account.blob_properties[0].versioning_enabled
# -> true
```
`Ctrl + C` to exit, then:
```bash
# Steps 4 and 5
terraform init          # needed because azuread is a new provider
terraform apply -refresh=false

# Step 6
mv terraform.tfstate terraform.tfstate.001
mv terraform.tfstate.backup terraform.tfstate.backup.001
terraform plan          # -> proposes creating EVERYTHING from scratch
mv terraform.tfstate.001 terraform.tfstate
mv terraform.tfstate.backup.001 terraform.tfstate.backup
terraform plan          # -> "No changes"
```

**Step 6 explanation:** with no state file, Terraform has no mapping between your internal names and the real Azure object IDs. It assumes nothing exists and plans to create it all. **The resources are still in Azure** — Terraform has simply forgotten it owns them. That gap between "reality" and "what Terraform believes" is the single most important thing to understand about this tool.

---

**Challenge**

- `SLIDE ACROSS`

*Direct* students, **in pairs**, to extend their configuration so that:

* A **second** storage account is created in the same resource group, with a different name
* It uses `account_replication_type = "GRS"` instead of `LRS`
* Both storage accounts **reference** the resource group rather than hard-coding its name
* An **output** called `all_blob_endpoints` returns a **list** containing the `primary_blob_endpoint` of *both* (see `terraform.tfstate` to see the keys)
* **OPTIONAL** — a second output returning only the **first** endpoint from that list

*Provide* this example output shape as an aid:

```
all_blob_endpoints = [
  "https://stjbloggsdevops02.blob.core.windows.net/",
  "https://stjbloggsdevops03.blob.core.windows.net/",
]
```

*Grant* ~10 minutes. Hint if stuck: a list in HCL uses square brackets, exactly like a JavaScript array — `[ thing_one, thing_two ]`. Indexing works the same way too.

**SOLUTION**

```tf
resource "azurerm_storage_account" "my_storage_account" {
  name                     = "stjbloggsdevops02"
  resource_group_name      = azurerm_resource_group.my_resource_group.name
  location                 = azurerm_resource_group.my_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "LRS"

  blob_properties {
    versioning_enabled = true
  }
}

# The second storage account — same resource group, different redundancy
resource "azurerm_storage_account" "my_backup_storage_account" {
  name                     = "stjbloggsdevops03"
  resource_group_name      = azurerm_resource_group.my_resource_group.name
  location                 = azurerm_resource_group.my_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "GRS"
}

# A list built from two references
output "all_blob_endpoints" {
  value = [
    azurerm_storage_account.my_storage_account.primary_blob_endpoint,
    azurerm_storage_account.my_backup_storage_account.primary_blob_endpoint,
  ]
}

# OPTIONAL
output "first_blob_endpoint" {
  value = azurerm_storage_account.my_storage_account.primary_blob_endpoint
}
```

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform plan     # -> 1 to add
terraform apply
# type: yes
```

Three points to draw out when revealing:

- The two resources have **different internal names** but the **same type**. The internal name is what distinguishes them — same as two variables of the same type in your code
- Building a list with `[ ]` in an output is **exactly** JavaScript array literal syntax. HCL borrows heavily from what you already know
- Both reference the resource group, so Terraform creates it first — and can safely create the two storage accounts **in parallel**, because neither depends on the other. It works that out from the graph, with no instruction from you

<br>
<br>

*(Take a 1 hour lunch break here.)*

<br>
<br>

### 14:00–14:45 — Refactoring, `.gitignore`, and Variables
*(Activity: 20 min)*

This morning everything went into one `main.tf`. Fine for five resources, unmanageable for five hundred. Let's tidy up, protect our secrets, and make the configuration dynamic.

#### Splitting the configuration

*(Run from `~/terraform-training/01-terraform-basics`)*
- Run: `touch outputs.tf`

Move all three `output` blocks out of `main.tf` into it:

**outputs.tf**
```tf
output "my_storage_account_versioning" {
  value = azurerm_storage_account.my_storage_account.blob_properties[0].versioning_enabled
}

output "my_storage_account_complete_details" {
  value = azurerm_storage_account.my_storage_account
}

output "my_azuread_user_complete_details" {
  value = azuread_user.my_azuread_user
}
```

*ALSO REMOVE THE OUTPUTS FROM `main.tf`*

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform plan -refresh=false
```


**No changes.** Exactly right — you moved text between files, you didn't change what you're asking for.

**The rule: Terraform reads *every* `.tf` file in the current directory and concatenates them.** File names are purely for humans. `main.tf`, `outputs.tf`, `banana.tf` — Terraform doesn't care, as long as the extension is `.tf`.

Two things follow, and both catch beginners:
- It reads the **current directory only**. It does **not** go into subfolders. That's why each folder we work in today is a completely separate project with its own state
- Because everything is concatenated, **order doesn't matter** — you can reference a resource defined in another file, or further down the same file

**ASK** <br>
"Every file is concatenated, order doesn't matter, filenames are convention only." How does that compare to how Node handles files? <br>
**ANSWER** <br>
It's the **opposite**. In Node, nothing exists until you `require` or `import` it, order matters, and the filename is the identity. In Terraform there are no imports — every `.tf` file in the folder is automatically part of one big configuration. Which means splitting files is **purely organisational**, with zero functional effect, and also means a stray `.tf` file you forgot about is silently part of your infrastructure.

Same for resources:

*(Run from `~/terraform-training/01-terraform-basics`)*
- Run: `touch resources.tf`

Move the resource group, storage account and AD user blocks across, and remove them from `main.tf`. `main.tf` now contains only the `terraform` and `provider` blocks.

The conventional layout you'll see in real projects: `main.tf` (providers and backend), `variables.tf`, `resources.tf`, `outputs.tf`.

#### Protecting state with .gitignore

**ASK** <br>
We've established the state file is important and the team needs it. So we should commit `terraform.tfstate` to GitHub… shouldn't we? <br>
**ANSWER** <br>
**No.** State files store attribute values **unencrypted** — including secrets. Your AD user's password is sitting in that file in plain text right now. Committing it, especially to a public repo, hands those secrets to anyone who can read it.

Have them actually check:

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
grep -i password terraform.tfstate
```

That's the `grep` from Session 2, finding a plaintext credential in a file people routinely commit by accident.

The proper solution is a **remote backend** — state in an Azure Storage Account blob container, so the team pulls the same authoritative state and it's never in Git. **That's Part 2.** Today's takeaway: **state must not be publicly available.**

You should already have the `.gitignore` in `~/terraform-training` from this morning. Confirm it:

*(Run from `~/terraform-training`)*
```bash
cat .gitignore
```

```gitignore
*.tfstate
*.tfstate.*
**/.terraform/
*.tfvars
```

- `*.tfstate` and `*.tfstate.*` — state and its backups
- `**/.terraform/` — the provider download folder at **any** depth. `**` means "any level of nesting". Hundreds of MB of binaries `terraform init` can re-download
- `*.tfvars` — variable files, which often hold environment-specific or sensitive values

#### Variables

Everything so far has been a hard-coded **constant**. Let's make it dynamic.

*(Run from `~/terraform-training/01-terraform-basics`)* — add to `main.tf`:

**main.tf**
```tf
# NEW CONFIG
variable "iam_user_name_prefix" {
  default = "my_iam_user"
}
```

And use it:

```tf
resource "azuread_user" "my_azuread_user" {
  # UPDATED — remember: YOUR domain
  user_principal_name = "${var.iam_user_name_prefix}_def@jbloggs.onmicrosoft.com"
  display_name        = "${var.iam_user_name_prefix}_def"
  mail_nickname       = "${var.iam_user_name_prefix}_def"
  password            = "ChangeMe123!ChangeMe"
}
```

Two pieces of new syntax:

**Declaring** — `variable "name" { default = "value" }`. The block declares it exists and gives a fallback.

**Using** — `${var.iam_user_name_prefix}`. The `var.` prefix is how Terraform knows you mean a variable rather than a resource. And `${ ... }` is **string interpolation**.

**ASK** <br>
Where have you seen `${ }` before? <br>
**ANSWER** <br>
**JavaScript template literals** — and in the Jenkins session, and bash uses `${VAR}` too. Same concept everywhere: drop the value of an expression into the middle of a string. HCL borrows the idea wholesale, which is why it needs about ten seconds of explanation rather than ten minutes.

One subtlety: you only need `${ }` when the value is **inside a larger string**. If the whole value is just the variable, write it bare:

```tf
display_name = var.iam_user_name_prefix          # correct
display_name = "${var.iam_user_name_prefix}"     # works, but redundant
```

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform apply -refresh=false
```

No changes — the interpolated result is identical to the constant it replaced.

#### Typing variables

```tf
variable "iam_user_name_prefix" {
  # NEW CONFIG
  type    = string
  default = "my_iam_user"
}
```

The available types:

| Type | Meaning |
|---|---|
| `any` | the default — anything goes |
| `string` | text |
| `number` | numeric |
| `bool` | `true` / `false` |
| `list` | ordered, indexed, duplicates allowed — like a JS array |
| `tuple` | like a list, but each position has a fixed type |
| `set` | unordered, unique values only |
| `map` | key/value pairs — like a JS object |

**ASK** <br>
`list`, `set` and `map` should all be familiar. What are they in JavaScript? <br>
**ANSWER** <br>
`list` is an **Array**, `set` is a **Set**, `map` is an **Object** (or a `Map`). And the distinctions matter for the same reasons: a Set enforces uniqueness and doesn't preserve order; an Object gives you key-based lookup. **This afternoon that distinction stops being academic** — choosing a list versus a set changes whether Terraform destroys and recreates your infrastructure.

Watch a mismatch:

```tf
variable "iam_user_name_prefix" {
  type    = number     # deliberately wrong
  default = "my_iam_user"
}
```

**ASK** <br>
How can we check whether our configuration is sound, without touching Azure? <br>
**ANSWER** <br>
`terraform validate`

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform validate
```
![intro-to-terraform-37](./resources/intro-to-terraform-37.png)

Set it back to `string`.

#### What if there's no default?

```tf
variable "iam_user_name_prefix" {
  type = string
  # default = "my_iam_user"
}
```

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform validate     # -> Valid! Interesting.
terraform apply -refresh=false
```
![intro-to-terraform-39](./resources/intro-to-terraform-39.png)

Terraform **prompts you interactively** for the value. A variable without a default isn't an error — it's just **required**.

- Type: `my_iam_user`

**ASK** <br>
That prompt is quite friendly. Where would it be a serious problem? <br>
**ANSWER** <br>
In a **pipeline**. A Jenkins build has no human to type an answer, so it hangs until it times out. Same trap as the `read -p` prompt in Session 2 and the missing `-y` on `apt install`: **convenient by hand, fatal in automation.** In CI you always supply variables non-interactively — which is exactly what we're about to cover.

*UNCOMMENT THE DEFAULT VALUE*

#### The four ways to set a variable

**1. The `default` in the block** — the fallback.

**2. An environment variable**, prefixed `TF_VAR_`:

*(Run from `~/terraform-training/01-terraform-basics`, in VS Code's terminal)*
```bash
export TF_VAR_iam_user_name_prefix=FROM_ENV_VARIABLE_IAM_PREFIX
export | grep TF_VAR
terraform plan -refresh=false
```
![intro-to-terraform-41](./resources/intro-to-terraform-41.png)

The rule is `TF_VAR_` followed by the **exact** variable name. Remove it again with `unset TF_VAR_iam_user_name_prefix`.

**NOTE FOR TRAINERS** <br>
Flag this hard: if a student ever sees a `plan` proposing changes they can't explain, **a forgotten `TF_VAR_` environment variable in that terminal is a very common culprit** — and it's invisible in the code. Teach `export | grep TF_VAR` as the first diagnostic. <br>
**END OF NOTE**

**3. A `terraform.tfvars` file:**

*(Run from `~/terraform-training/01-terraform-basics`)*
- Run: `touch terraform.tfvars`

**terraform.tfvars**
```tf
iam_user_name_prefix = "VALUE_FROM_TERRAFORM_TFVARS"
```

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform plan -refresh=false
```
![intro-to-terraform-42](./resources/intro-to-terraform-42.png)

Note there's **no `variable` keyword** in a `.tfvars` file — just `name = value`. Declared in `.tf`, assigned here.

**4. On the command line:**

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform plan -refresh=false -var="iam_user_name_prefix=VALUE_FROM_CLI"
```

**Precedence, lowest to highest:**
1. `default` in the variable block
2. `TF_VAR_` environment variables
3. `terraform.tfvars`
4. `-var` on the command line

There's also `-var-file`, for pointing at a specific file:
```bash
terraform apply -var-file="prod.tfvars"
```

**ASK** <br>
Why is that ordering the *right* way round? <br>
**ANSWER** <br>
Because **the more specific and deliberate the instruction, the higher it wins.** A `default` is "if nobody says otherwise". An environment variable is per-machine. A `.tfvars` file is per-project. A `-var` flag is someone typing an explicit override right now, so it beats everything. It's the same hierarchy as CSS specificity, or config files versus command-line flags in any tool you've used.

#### Why variables actually matter

The reason we spend time on this: **the same Terraform configuration gets used across multiple environments.** You'll provision Development, Test and Production — and you do *not* want three copies of the code that can drift apart.

A common pattern is an `environment` variable woven into resource names:

```tf
variable "environment" {
  default = "dev"
}

resource "azuread_user" "my_azuread_user" {
  user_principal_name = "${var.environment}_${var.iam_user_name_prefix}_def@jbloggs.onmicrosoft.com"
  display_name        = "${var.environment}_${var.iam_user_name_prefix}_def"
  [ . . . ]
}
```

More practically you'd also use it to vary **scale** — one small VM in test, six large in production — and **location**, standing up the same stack in a different region by changing one value. `dev.tfvars`, `test.tfvars`, `prod.tfvars`: one configuration, three sets of values.

*DELETE THE `environment` VARIABLE AND REVERT THE NAMES*, then `terraform validate`.

#### A few more commands worth knowing

**`terraform fmt`** — auto-formats every `.tf` file to the standard style. Run it before committing; it removes an entire category of pointless code-review comments. It's Prettier, for Terraform.

**`terraform show`** — prints Known State readably, far friendlier than raw JSON.

**`terraform validate`** — syntax and internal consistency, without contacting Azure. Fast, and pipeline-friendly.

**HANDS ON (20 min)** <br>
*(Run from `~/terraform-training/01-terraform-basics`)*
1. Split your configuration into `main.tf`, `resources.tf` and `outputs.tf`. Confirm with `terraform plan -refresh=false` that nothing changed
2. Run `grep -i password terraform.tfstate` and **look at what's in there in plain text**
3. Confirm the `.gitignore` in `~/terraform-training` covers state, `.terraform/` and `.tfvars`
4. Introduce a variable `iam_user_name_prefix` with a `type` and a `default`, and use it in the AD user's names
5. Override it **three ways in turn** — `TF_VAR_`, a `terraform.tfvars` file, and `-var` on the CLI — running `terraform plan -refresh=false` each time to watch precedence
6. Break something deliberately (misspell `var.` or a resource type) and confirm `terraform validate` catches it. Then run `terraform fmt`
**END OF NOTE**

**💬 SLACK — snippet 7**:
```bash
# Check what's in your state file. This is why it never goes in Git.
grep -i password terraform.tfstate

# The four ways to set a variable, lowest to highest precedence:
terraform plan -refresh=false                                    # 1. default
export TF_VAR_iam_user_name_prefix=FROM_ENV                      # 2. env var
terraform plan -refresh=false
echo 'iam_user_name_prefix = "FROM_TFVARS"' > terraform.tfvars   # 3. tfvars
terraform plan -refresh=false
terraform plan -refresh=false -var="iam_user_name_prefix=FROM_CLI"  # 4. CLI

# tidy up
unset TF_VAR_iam_user_name_prefix
rm terraform.tfvars
```

**Solution**

**main.tf**
```tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.0"
    }
  }
}

provider "azurerm" {
  features {}
}

provider "azuread" {}

variable "iam_user_name_prefix" {
  type    = string
  default = "my_iam_user"
}
```

**resources.tf**
```tf
resource "azurerm_resource_group" "my_resource_group" {
  name     = "rg-jbloggs-devops"
  location = "uksouth"
}

resource "azurerm_storage_account" "my_storage_account" {
  name                     = "stjbloggsdevops02"
  resource_group_name      = azurerm_resource_group.my_resource_group.name
  location                 = azurerm_resource_group.my_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "LRS"

  blob_properties {
    versioning_enabled = true
  }
}

resource "azuread_user" "my_azuread_user" {
  user_principal_name = "${var.iam_user_name_prefix}_def@jbloggs.onmicrosoft.com"
  display_name        = "${var.iam_user_name_prefix}_def"
  mail_nickname       = "${var.iam_user_name_prefix}_def"
  password            = "ChangeMe123!ChangeMe"
}
```

**outputs.tf**
```tf
output "my_storage_account_versioning" {
  value = azurerm_storage_account.my_storage_account.blob_properties[0].versioning_enabled
}

output "my_storage_account_complete_details" {
  value = azurerm_storage_account.my_storage_account
}

output "my_azuread_user_complete_details" {
  value = azuread_user.my_azuread_user
}
```

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform plan -refresh=false          # step 1 -> No changes
grep -i password terraform.tfstate     # step 2 -> your password, in plain text
terraform validate                     # step 6
terraform fmt
```

**Step 2 is the one to dwell on.** The output shows the AD user's password sitting unencrypted in a JSON file. Ask: *"who here has ever committed a file without looking at what was in it?"* Everyone. That's the entire argument for the `.gitignore`, and for the remote backend in Part 2.

<br>
<br>

### 14:45–15:00 — Creating Many Resources with `count`

Everything so far has been one block, one resource. Let's create many.

*(Run from `~/terraform-training/01-terraform-basics`)*
- Run: `cd ..`

*(Run from `~/terraform-training`)*
- Run: `mkdir 02-count` → **cd inside**

*(Run from `~/terraform-training/02-count`)*
- Run: `touch main.tf`

**02-count/main.tf**
```tf
terraform {
  required_providers {
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.0"
    }
  }
}

provider "azuread" {}

resource "azuread_user" "my_azuread_users" {
  # NEW CONFIG
  count               = 2
  user_principal_name = "my_iam_user_${count.index}@jbloggs.onmicrosoft.com"
  display_name        = "my_iam_user_${count.index}"
  mail_nickname       = "my_iam_user_${count.index}"
  password            = "ChangeMe123!ChangeMe"
}
```

Note the internal name is now **plural** — `my_azuread_users`. It represents many things, so the name should say so.

`count = 2` is a **meta-argument** — an argument Terraform itself understands, available on *any* resource type regardless of provider. "Make this many copies of this block."

`count.index` is then available inside the block, holding the current iteration number, **starting at 0**. Without something varying per copy, you'd be creating two identical users — and `user_principal_name` must be unique.

**ASK** <br>
What do we need to run first in this new folder, and why? <br>
**ANSWER** <br>
`terraform init`. Each project directory is **independent** and needs its own provider plugins in its own `.terraform/` folder. Same as running `npm install` in a new project — the dependencies don't come with you.

*(Run from `~/terraform-training/02-count`)*
```bash
terraform init
terraform apply
# type: yes
```

*(In the Azure Portal — Microsoft Entra ID → Users)* — both users are there.

Now change `count = 2` to `count = 3`:

*(Run from `~/terraform-training/02-count`)*
```bash
terraform apply
```
![intro-to-terraform-26](./resources/intro-to-terraform-26.png)

**This is the declarative model in one screenshot.** You didn't say "add one more". You said "there should be 3". Terraform checked state, found 2, and worked out that **one creation** gets you there.

- Type: `yes`

Look in the console:

*(Run from `~/terraform-training/02-count`)*
```bash
terraform console
```
```
azuread_user.my_azuread_users
azuread_user.my_azuread_users[0].user_principal_name
```
![intro-to-terraform-27](./resources/intro-to-terraform-27.png)

The reference now returns a **list** — because `count` produced many. So you index into it and chain attributes, exactly like a JavaScript array.

Exit with `Ctrl + C`.

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>
### 15:15–16:45 — Capstone: Lists, Sets, `for_each` and Maps
*(Activity: 90 min)*

The main event. You're going to discover a genuine, well-known problem with `count`, fix it with `for_each`, then build up to configurations rich enough to describe real infrastructure.

Work individually or in pairs. **Work through Parts 1–4 in order** — each one motivates the next. Part 5 is stretch.

**NOTE FOR TRAINERS** <br>
Do not skip ahead to `for_each` because it's "the right answer". **The whole value of this capstone is that students hit the `count` problem themselves and feel it.** Part 2 asks them to predict what will happen, then shows them being wrong. That five seconds of surprise is worth more than any amount of explaining, and it's what makes the `for_each` rule stick permanently. <br>
**END OF NOTE**

---

#### Part 1 (≈20 min) — Driving resources from a list

Let's move away from `my_iam_user_0` and use real names.

*(Run from `~/terraform-training/02-count`)*
- Run: `terraform destroy` → type `yes`
- Run: `cd ..`

*(Run from `~/terraform-training`)*
- Run: `mkdir 03-lists-and-sets` → **cd inside**

*(Run from `~/terraform-training/03-lists-and-sets`)*
- Run: `touch main.tf`

**03-lists-and-sets/main.tf**
```tf
terraform {
  required_providers {
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.0"
    }
  }
}

provider "azuread" {}

variable "names" {
  default = ["emile", "romeo", "sarah"]
}

resource "azuread_user" "my_azuread_users" {
  count               = length(var.names)
  user_principal_name = "${var.names[count.index]}@jbloggs.onmicrosoft.com"
  display_name        = var.names[count.index]
  mail_nickname       = var.names[count.index]
  password            = "ChangeMe123!ChangeMe"
}
```

New syntax:

- `default = ["emile", "romeo", "sarah"]` — a **list**, square brackets. A JavaScript array, indexed from 0
- `length(var.names)` — a **built-in function**. In JavaScript you'd write `names.length`; HCL uses function-call syntax: `length(thing)`. It evaluates to 3, so `count.index` runs 0, 1, 2
- `var.names[count.index]` — index into the list using the current iteration number

Notice `display_name = var.names[count.index]` has **no** `${ }` — the whole value *is* the expression. But `user_principal_name` needs it, because the value is embedded in a larger string alongside `@domain`.

*(Run from `~/terraform-training/03-lists-and-sets`)*
```bash
terraform init
terraform apply
```

*WHILST IT'S RUNNING, SAY:*

Worth explaining why we keep making new project folders. It's **Terraform best practice** — a single application commonly has several separate Terraform projects, because different resources have different **lifecycles**.

A storage account might live for years, holding logs, assets and backups. A deployment might change daily. Grouping resources by how often they change, and managing each group as its own project with its own state, means a daily deployment change can never accidentally destroy the storage everything depends on. It also keeps `plan` fast and limits blast radius.

![intro-to-terraform-44](./resources/intro-to-terraform-44.png)

- Type: `yes`

Explore in the console:

*(Run from `~/terraform-training/03-lists-and-sets`)*
```bash
terraform console
```

```
var.names
azuread_user.my_azuread_users[0].display_name
```

While you're here, meet the **collection functions**:

```
length(var.names)                     # 3
reverse(var.names)                    # reverses the order
distinct(var.names)                   # removes duplicates
toset(var.names)                      # converts to a set
concat(var.names, ["tom", "astha"])   # joins two lists
contains(var.names, "simon")          # true / false
sort(var.names)                       # alphabetical
```

**ASK** <br>
Look at that list of functions. What do they remind you of? <br>
**ANSWER** <br>
**JavaScript array methods** — `.length`, `.reverse()`, `.concat()`, `.includes()`, `.sort()`, and `new Set()`. Nearly one-for-one. The only real difference is **syntax**: HCL puts the collection *inside* the function call rather than calling a method on it. So `names.sort()` becomes `sort(var.names)`. If you know the array methods, you already know most of this.

Note `distinct` returns `tolist([...])` — Terraform being explicit that it produced a list.

And `range` for sequences:
```
range(10)          # 0 to 9
range(1, 12)       # 1 up to but not including 12
range(1, 12, 3)    # third argument is the step: 1, 4, 7, 10
```

Exit with `Ctrl + C`. Get into the habit of checking the [function documentation](https://developer.hashicorp.com/terraform/language/functions) — there are dozens more.

---

#### Part 2 (≈20 min) — The problem with lists

**This is the exercise that makes the whole afternoon click.** Add a name to the **front** of the list:

**main.tf**
```tf
variable "names" {
  # UPDATED
  default = ["simon", "emile", "romeo", "sarah"]
}
```

**Before running anything — predict what will happen.** We added one name to a list of three. Write down what you expect the plan to say. Ask two or three people out loud.

*(Run from `~/terraform-training/03-lists-and-sets`)*
```bash
terraform apply
```
![intro-to-terraform-57](./resources/intro-to-terraform-57.png)

**1 to add… but 3 to change.**

**ASK** <br>
We added exactly one name. Why is Terraform changing three existing users? <br>
**ANSWER** <br>
Because `count` tracks resources by **position**, not identity. Everything shifted: `simon` now occupies index 0 where `emile` was, `emile` moved to 1, `romeo` to 2 — so Terraform sees each slot as having *changed value*, and index 3 as brand new. It has no concept that "emile" is the same person who was there before; it only knows "the resource at index 0 used to be emile and should now be simon".

Look in the state file to see exactly why:

**terraform.tfstate**
```tf
"instances": [
  {
    "index_key": 0,                    # <-- A NUMBER
    "attributes": { "display_name": "emile" }
  },
  {
    "index_key": 1,
    "attributes": { "display_name": "romeo" }
  },
  {
    "index_key": 2,
    "attributes": { "display_name": "sarah" }
  }
]
```

The `index_key` is a **number**. That number is the only thing joining your configuration to the real Azure object. Reorder the list and every mapping is wrong.

**ASK** <br>
Where have you seen exactly this bug before, in a completely different context? <br>
**ANSWER** <br>
**React's `key` prop.** Using an array index as a key, then inserting an item at the front — React re-renders and mismatches every element after the insertion point, because it identifies children by position rather than identity. React's official advice is "don't use the index as a key, use a stable ID". **It's the identical bug with identical reasoning**, and `for_each` is Terraform's version of "use a stable ID".

**ASK** <br>
On Azure AD users this is annoying. Where would this be genuinely dangerous? <br>
**ANSWER** <br>
Anywhere the resource holds **state** or takes time to build — databases, VMs with data on them, storage accounts. Inserting one item at the top of a list could trigger the destruction and recreation of **every** resource after it. In React the cost is a visual glitch. Here the cost is data loss and downtime. This is the single most common way people get burned by `count`.

Destroy before we fix it:

*(Run from `~/terraform-training/03-lists-and-sets`)*
```bash
terraform destroy
# type: yes
```

---

#### Part 3 (≈25 min) — Fixing it with `for_each`

**main.tf**
```tf
variable "names" {
  default = ["simon", "emile", "romeo", "sarah"]
}

resource "azuread_user" "my_azuread_users" {
  # OLD — commented out
  #   count               = length(var.names)
  #   user_principal_name = "${var.names[count.index]}@jbloggs.onmicrosoft.com"

  # NEW CONFIG
  for_each            = toset(var.names)
  user_principal_name = "${each.value}@jbloggs.onmicrosoft.com"
  display_name        = each.value
  mail_nickname       = each.value
  password            = "ChangeMe123!ChangeMe"
}
```

The changes:

- `for_each` replaces `count`. The other iteration meta-argument, working over a **set or a map** rather than a number
- `toset(var.names)` converts our list to a **set**. `for_each` requires unique values, and a list can contain duplicates — so we convert. (A set is unordered and unique-only, exactly like a JavaScript `Set`)
- `each.value` replaces `var.names[count.index]`. Inside a `for_each` block you get `each.value` (the current item) and `each.key`. For a set, key and value are the same thing

*(Run from `~/terraform-training/03-lists-and-sets`)*
```bash
terraform apply
```
![intro-to-terraform-58](./resources/intro-to-terraform-58.png)

Four resources — and notice **the order doesn't match your list**.

**Sets prioritise uniqueness, not order.** They're implemented with a **hash table**, which doesn't preserve insertion order.

**OPTIONAL SIDE TANGENT**
- **Hashing** — each element runs through a hash function, turning it into a numeric fingerprint
- **Storage** — that fingerprint is used as an address in a table
- **Lookup** — to check membership, hash it and jump straight to that address

Extremely fast lookups; the price is that ordering isn't retained. Exactly the same trade-off as a JavaScript `Set` or `Object` versus an `Array`.
**END SIDE TANGENT**

- Type: `yes`

Now look at the state file again:

**terraform.tfstate**
```tf
"instances": [
  {
    "index_key": "emile",              # <-- A STRING. The value itself.
    "attributes": { "display_name": "emile" }
  },
  {
    "index_key": "romeo",
    "attributes": { "display_name": "romeo" }
  },
  {
    "index_key": "simon",
    "attributes": { "display_name": "simon" }
  }
]
```

**`index_key` is now a string — the value itself.** That's the whole fix. Each resource is tracked by **what it is**, not **where it sat**.

Prove it. Add a name at the front:

**main.tf**
```tf
variable "names" {
  default = ["astha", "simon", "emile", "romeo", "sarah"]
}
```

*(Run from `~/terraform-training/03-lists-and-sets`)*
```bash
terraform apply
```
![intro-to-terraform-59](./resources/intro-to-terraform-59.png)

**1 to add, 0 to change.** Exactly what a human would expect.

- Type: `yes`

And removal behaves properly too:

**main.tf**
```tf
variable "names" {
  default = ["emile", "sarah"]
}
```

*(Run from `~/terraform-training/03-lists-and-sets`)*
```bash
terraform apply
```

It identifies precisely which users to remove and leaves the rest alone.

- Type: `yes`

**The rule to take away: prefer `for_each` over `count` whenever the things you're creating have a meaningful identity.** Reserve `count` for genuinely interchangeable resources, or for conditionally creating something:

```tf
count = var.enable_backup ? 1 : 0     # a ternary — same as JavaScript
```

---

#### Part 4 (≈25 min) — Maps: richer data per resource

A set gives one value per resource. What if each user needs a country *and* a department?

*(Run from `~/terraform-training/03-lists-and-sets`)*
- Run: `terraform destroy` → type `yes`
- Run: `cd ..`

*(Run from `~/terraform-training`)*
- Run: `mkdir 04-maps` → **cd inside**

*(Run from `~/terraform-training/04-maps`)*
- Run: `touch main.tf`

Copy the config across from `03-lists-and-sets`, then convert the variable from a list to a **map**:

**04-maps/main.tf**
```tf
terraform {
  required_providers {
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.0"
    }
  }
}

provider "azuread" {}

# UPDATED — square brackets [ ] become curly braces { }
variable "users" {
  default = {
    emile : "England",
    sarah : "France"
  }
}

resource "azuread_user" "my_azuread_users" {
  for_each            = var.users
  user_principal_name = "${each.key}@jbloggs.onmicrosoft.com"
  display_name        = each.key
  mail_nickname       = each.key
  password            = "ChangeMe123!ChangeMe"
  country             = each.value
}
```

*(Run from `~/terraform-training/04-maps`)*
```bash
terraform init
```

What changed:

- **`[ ]` became `{ }`** — the syntactic difference between a list and a map
- A map is **key/value pairs**, like a JavaScript object. Key is the username, value the country
- `for_each = var.users` — **no `toset()` needed**. Map keys are already unique by definition
- `each.key` is now the username, `each.value` the country. With a set they were the same; with a map they're properly distinct
- Renamed from `names` to `users`, because it holds more than names. **Name things for what they contain**

Explore it:

*(Run from `~/terraform-training/04-maps`)*
```bash
terraform console
```

```
var.users
var.users.sarah            # dot notation
var.users["sarah"]         # bracket notation — identical result
keys(var.users)            # just the keys
values(var.users)          # just the values
lookup(var.users, "emile") # find a value by key
```

**ASK** <br>
Both `.sarah` and `["sarah"]` work. Where else is that true? <br>
**ANSWER** <br>
**JavaScript objects** — `obj.sarah` and `obj["sarah"]` are interchangeable, with the same caveat that bracket notation is required for keys that aren't valid identifiers. `keys()` and `values()` are `Object.keys()` and `Object.values()`. HCL is borrowing your existing mental model almost wholesale.

Exit with `Ctrl + C`, then apply:

*(Run from `~/terraform-training/04-maps`)*
```bash
terraform apply
# type: yes
```

*(In the Azure Portal — Microsoft Entra ID → Users)* — both users exist, with countries set.

Check the state: `index_key` is the username, same as with a set.

**Now nest a map inside the map**, so each user carries multiple attributes:

**main.tf**
```tf
variable "users" {
  default = {
    # UPDATED — the value is now ANOTHER MAP
    emile : { country : "England" },
    sarah : { country : "France" }
  }
}
```

**ASK** <br>
The resource says `country = each.value`. How do we reach the country now? <br>
**ANSWER** <br>
`each.value.country` — `each.value` is now an object, so you chain onto it, exactly as in JavaScript.

```tf
resource "azuread_user" "my_azuread_users" {
  [ . . . ]
  # UPDATED
  country             = each.value.country
}
```

*(Run from `~/terraform-training/04-maps`)*
```bash
terraform apply
```

**No changes** — the resulting values are identical, even though the route to them changed. **Terraform compares results, not the expressions that produced them.**

Now the payoff — add another attribute:

**main.tf**
```tf
variable "users" {
  default = {
    emile : { country : "England", department : "Training" },
    sarah : { country : "France", department : "Training" }
  }
}

resource "azuread_user" "my_azuread_users" {
  [ . . . ]
  country             = each.value.country
  # UPDATED
  department          = each.value.department
}
```

Both `country` and `department` are **native, typed attributes** on `azuread_user` — first-class fields, not entries stuffed into a generic tags map.

*(Run from `~/terraform-training/04-maps`)*
```bash
terraform apply
# type: yes
```

An in-place update adding the attribute.

---

#### Part 5 — Stretch goals

1. **A third user with different values.** Add someone with a different country and department, apply, confirm only one resource is created
2. **Optional attributes.** Give one user a `job_title` and the others none. *(Hint: `lookup(each.value, "job_title", null)` returns a default when the key is absent)*
3. **An output over the map.** List every created user's `user_principal_name`. *(Hint: `values(azuread_user.my_azuread_users)[*].user_principal_name` — `[*]` is the **splat operator**, pulling one attribute from every element)*
4. **Drift detection.** In the Portal, manually change one user's department. Then run `terraform plan` and see Terraform detect that Actual no longer matches Desired

**Solution**

**Stretch 1 and 2:**
```tf
variable "users" {
  default = {
    emile : { country : "England", department : "Training", job_title : "Trainer" },
    sarah : { country : "France", department : "Training" }
    romeo : { country : "Italy", department : "Engineering" }
  }
}

resource "azuread_user" "my_azuread_users" {
  for_each            = var.users
  user_principal_name = "${each.key}@jbloggs.onmicrosoft.com"
  display_name        = each.key
  mail_nickname       = each.key
  password            = "ChangeMe123!ChangeMe"
  country             = each.value.country
  department          = each.value.department

  # lookup(map, key, default) — returns the default when the key is missing,
  # so users without a job_title don't error
  job_title           = lookup(each.value, "job_title", null)
}
```

`lookup(map, key, default)` is the same idea as `obj.key ?? default` in JavaScript.

**Stretch 3:**
```tf
output "all_user_principal_names" {
  # values() turns the map of resources into a list,
  # then [*] pulls one attribute from every element
  value = values(azuread_user.my_azuread_users)[*].user_principal_name
}
```

`[*]` is doing the job of `.map(u => u.userPrincipalName)`.

*(Run from `~/terraform-training/04-maps`)*
```bash
terraform apply     # -> 1 to add (romeo), 1 to change (emile gains a job_title)
# type: yes
terraform output all_user_principal_names
```

**Stretch 4 — drift detection.**

*(In the Azure Portal — Microsoft Entra ID → Users → sarah)* — change **Department** to `Marketing` and save.

*(Run from `~/terraform-training/04-maps`)*
```bash
terraform plan
```

The plan reports **1 to change**, showing `department: "Marketing" -> "Training"`. Terraform refreshed Actual State, found it no longer matches Desired State in your code, and proposes putting it back. Run `terraform apply` and the manual change is reverted.

**This is the single strongest argument for Infrastructure as Code.** Your `.tf` files aren't just a record of what you built — **they're the authority.** Anything anyone does by hand gets detected and corrected. That's the ClickOps drift problem from Session 1, solved.

**ASK** <br>
Terraform just silently reverted someone's manual change. Is that always what you want? <br>
**ANSWER** <br>
Mostly yes — that's the point. But it has a real consequence: **the Portal effectively becomes read-only** for anything Terraform manages. Someone making an emergency fix at 3am in the Portal will find it silently undone by the next pipeline run. Which means the team has to *agree* that Terraform is the authority, and that emergency fixes go through the code. It's a cultural commitment as much as a technical one — the same commitment as "we don't push directly to `main`".

<br>
<br>

### 16:45–17:00 — Wrap-up, Destroy, & Q&A
*(Activity: 10 min)*

#### Clean up first — this is not optional

*(Run from `~/terraform-training/04-maps`)*
```bash
terraform destroy
# type: yes
```

*(Run from `~/terraform-training/03-lists-and-sets`)*
```bash
terraform destroy
# type: yes
```

*(Run from `~/terraform-training/02-count`)*
```bash
terraform destroy
# type: yes
```

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform destroy
# type: yes
```

Then verify nothing is lingering:

*(Run from anywhere)*
```bash
az resource list -o table
az group list -o table
```

**💬 SLACK — snippet 9**, post now:
```bash
# Run this in EACH of your four project folders:
terraform destroy      # then type: yes

# Then confirm from anywhere:
az resource list -o table
az group list -o table
```

`terraform destroy` works exactly like `apply` in reverse: it refreshes state, identifies every resource Terraform manages, and deletes them **in reverse dependency order** — so the storage account goes before the resource group containing it.

**ASK** <br>
Why is the fact that destroying is *this easy* a headline feature, not just a convenience? <br>
**ANSWER** <br>
Because it changes what you can afford to do. If tearing down is one command, you can spin up a full test environment for an afternoon and destroy it before you leave. Before cloud computing an organisation had to buy office space, order server components and physically install them. Now the whole lifecycle — **create, use, destroy** — is a habit. It's what keeps cloud spend under control, and it's why "just build a fresh environment for this feature branch" is a realistic sentence rather than a fantasy.

#### Where today sits

You started with the ClickOps complaint from Session 1 and ended with the answer. You can describe infrastructure declaratively, you understand what the state file is for and what happens when it's missing, and you can create many resources from one block — knowing why `for_each` beats `count`.

**ASK** <br>
Think back to Session 3. We built a Jenkins pipeline that checked out code, ran tests, then built and pushed a Docker image — with credentials from the vault and a `post` block. What would we change to make that pipeline run **Terraform** instead? <br>
**ANSWER** <br>
Structurally, **almost nothing**. You'd store the four `ARM_*` values in Jenkins' credentials vault instead of a Docker Hub token. The `sh` steps would run `terraform init`, then `terraform plan -out=tfplan` in one stage, then — after a human approval gate — `terraform apply tfplan`. And **that's precisely why we saved a plan to a file this morning**: the reviewed plan is the thing that gets applied, so nothing can change between review and execution.

There's exactly **one blocker**, and it's the thing Part 2 fixes: your state file is on your laptop. A Jenkins agent starts with an empty workspace, so it would believe nothing exists and try to create everything. **Remote state isn't a nice-to-have for pipeline Terraform — it's a prerequisite.**

Where this goes:
- **Today** — infrastructure as declarative, version-controlled code
- **Part 2** — real infrastructure (networks, VMs, load balancers) and **remote state** in an Azure Storage Account so a team can share it safely
- **Kubernetes** — running and scaling containers on infrastructure Terraform provisions
- **Integration** — a Git push flowing through test, build, `terraform plan`, review, `terraform apply`, deploy

**Before you leave:**
1. `my-terraform-notes.md` complete, including your Service Principal details
2. `az resource list -o table` shows nothing you don't want to pay for
3. Your `.gitignore` is in place, and **no state file has been committed anywhere**

**Q&A**

<br>
<br>

### Exercise (take-home / reinforcement)

Individually or in pairs. **Do at least one practical and one research task.**

**Practical**

1. From an **empty folder**, get to a working resource group and storage account **from memory** — no notes
2. Add a `location` variable defaulting to `uksouth`, use it on your resource group, then override it to `ukwest` on the command line and **read the plan**. Don't apply it
3. Convert a `count`-based configuration to `for_each`, and write a short paragraph explaining to a colleague why you did
4. Add a **blob container** (`azurerm_storage_container`) inside your storage account, referencing it properly so Terraform builds the dependency
5. Use `terraform state list` and `terraform state show <address>` to inspect state without opening the JSON

**Research**

6. **Compare IaC tools.** Pick **two** of Bicep, Pulumi, ARM templates and CloudFormation. Write half a page: what would make you choose each over Terraform, and Terraform over them?
7. **Read about remote backends** and write down, in your own words, what problem they solve that a local `terraform.tfstate` doesn't. *(This is Part 2's opening question — arrive with an answer)*
8. **(Stretch)** Research the `random_password` resource and rework your AD user so the password isn't hard-coded. Then work out why that only *half* solves the problem
9. **(Stretch)** Find three resource types in the [azurerm provider docs](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs) you haven't used. For each, find one attribute marked "Changing this forces a new resource to be created"

**Solutions** *(for the guided ones — 2, 4, 5, 8)*

**Take-home 2** — a location variable:
```tf
variable "location" {
  type        = string
  default     = "uksouth"
  description = "The Azure region resources are created in"
}

resource "azurerm_resource_group" "my_resource_group" {
  name     = "rg-jbloggs-devops"
  location = var.location
}
```

```bash
terraform plan -var="location=ukwest"
```

The plan shows the resource group **must be replaced** — location is another attribute that can't change in place. A very good reason to read plans carefully.

**Take-home 4** — a blob container:
```tf
resource "azurerm_storage_container" "my_container" {
  name                  = "uploads"
  # The reference creates the dependency, so Terraform builds
  # the storage account FIRST, then the container inside it
  storage_account_name  = azurerm_storage_account.my_storage_account.name
  container_access_type = "private"
}
```

**Take-home 5** — inspecting state safely:
```bash
terraform state list
# azurerm_resource_group.my_resource_group
# azurerm_storage_account.my_storage_account

terraform state show azurerm_storage_account.my_storage_account
```

`state list` gives the addresses; `state show` gives the detail for one. Far safer and faster than opening `terraform.tfstate` in an editor — and it removes any temptation to edit it.

**Take-home 8** — `random_password`, and why it only half-solves the problem:
```tf
resource "random_password" "user_password" {
  length  = 20
  special = true
}

resource "azuread_user" "my_azuread_user" {
  user_principal_name = "..."
  password            = random_password.user_password.result
}
```

Better: no password in your source code, so nothing sensitive reaches Git. **But** the generated value is still written into `terraform.tfstate` **in plain text** — you can prove it with the `grep` from this afternoon. So it solves the *source code* problem and not the *state* problem. The full answer is a remote backend with encryption and restricted access, which is Part 2. Worth landing, because it shows that "don't hard-code secrets" is necessary but not sufficient.

<br>
<br>

### Conclusion

- **Inform** students this marks the end of Terraform Part 1
- **Reinforce** the three states — Desired, Known, Actual — and that the state file is what **maps their code to real cloud resources**
- **Reinforce** the `count` versus `for_each` rule: track resources by **identity**, not position
- **Confirm** everyone has run `terraform destroy` in all four folders and checked `az resource list -o table`
- **Confirm** nobody has committed a state file
- **Preview** Part 2: real infrastructure — virtual networks, VMs, load balancers — and moving state to a **remote backend**, which is the one thing standing between them and running Terraform from a pipeline
- **Direct** students to the take-home exercises and the [azurerm provider docs](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs), which is the reference they'll use daily

---

[Back](./README.md)

---
