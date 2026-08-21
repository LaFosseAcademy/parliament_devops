# Terraform & Infrastructure as Code with Azure — Trainer Script

A full day taking trainees from "I've clicked resources into existence in the Azure Portal" to "I can describe infrastructure as code, understand what Terraform's state file is actually for, and create many resources from one configuration block". This is written as a script — read it out, adapt where it feels natural, but the intent is that you could deliver this cold from the page.

## Organisation

### Audience

Trainees who have already covered **Azure cloud fundamentals** (resource groups, subscriptions, storage accounts, RBAC, Service Principals), **Docker**, **bash scripting & automation**, and **Jenkins & CI/CD pipelines**. Assume they can navigate the Azure Portal, use `az` CLI commands, and read a config file — but assume **no prior Terraform** and **no prior HCL**. The syntax is explained from first principles, block by block.

They already know from the Azure session that "ClickOps is fragile". Today is the answer to that.

### How this document is laid out — read before delivering

Terraform days go wrong when people run a command in the wrong folder — Terraform is **directory-scoped**, so being in the wrong place genuinely changes what happens. Every command block is therefore labelled:

- *(Run from `~/terraform-training/01-terraform-basics`)* — a **terminal** command, in that exact folder
- *(In the Azure Portal — Resource groups)* — a **browser** action, starting from that screen

Navigation is written as breadcrumbs, e.g. **Resource groups → rg-yourname-devops → Data protection**.

The folders we build today:

| Folder | What it's for |
|---|---|
| `~/terraform-training` | Parent folder. The `.gitignore` lives here |
| `~/terraform-training/01-terraform-basics` | Resource group, storage account, first AD user, state, outputs |
| `~/terraform-training/02-count` | Creating many resources with `count` |
| `~/terraform-training/03-lists-and-sets` | `length()`, `for_each`, sets vs lists |
| `~/terraform-training/04-maps` | Maps, nested maps, richer attributes |

**Every activity has a `**Solution**` block** immediately afterwards.

Set the parent folder up before you start:

*(Run from `~/`)*
- Run: `mkdir -p ~/terraform-training` → **cd inside** with `cd ~/terraform-training`
- Confirm with `pwd`

### Duration & schedule

Full day, **09:00–17:00**, with a one-hour lunch and two 15-minute breaks. Roughly **4 hours of instruction and 3 hours of hands-on**, weighted towards a large end-of-day capstone.

| Time | Section | Shape |
|---|---|---|
| 09:00–09:15 | Welcome & recap: the ClickOps problem, solved | Talk |
| 09:15–10:00 | What Terraform is, and the three states | Talk + discussion |
| **10:00–10:15** | **Break** | |
| 10:15–11:15 | Setup: install, Service Principal, credentials, `init` | **Exercise** |
| 11:15–12:15 | Your first resources: resource group & storage account | **Exercise** |
| 12:15–13:00 | State, the console, and outputs | **Exercise + Challenge** |
| **13:00–14:00** | **Lunch** | |
| 14:00–14:45 | Refactoring, `.gitignore`, and variables | Talk + exercise |
| 14:45–15:00 | Creating many resources with `count` | Talk + demo |
| **15:00–15:15** | **Break** | |
| 15:15–16:45 | Capstone: lists, sets, `for_each` and maps | **Exercise (1 hr 30 min)** |
| 16:45–17:00 | Wrap-up, destroy, & Q&A | Talk |

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
  - An **Azure account** with an active subscription (the free trial from the Azure Fundamentals session is fine)
  - The **Azure CLI** installed and working — `az --version` responds, and they can `az login`
  - **VS Code** (or another text editor), ideally with the HashiCorp Terraform extension
  - Permission to **create a Service Principal** in their tenant (see the trainer note in Section 3)
  - Their own **Azure AD domain name** to hand — they'll need it for user principal names

**NOTE FOR TRAINERS — the two things that will eat your morning** <br>
**(1) Service Principal permissions.** On a personal free-trial subscription students are usually the tenant owner and `az ad sp create-for-rbac` just works. On a *corporate* tenant it very often does not — creating app registrations may be restricted. Test this yourself in the target environment beforehand. Fallback: students authenticate with plain `az login` and let the `azurerm` provider use that session, which works fine for everything today except demonstrating the Service Principal pattern — which you can then demo once from your own machine. <br>
**(2) Globally-unique storage account names.** Every student needs a different one. Get them to settle on a personal prefix (e.g. `stjbloggs`) at the start and use it all day. <br>
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

<br>

## Sequence

### 09:00–09:15 — Welcome & Recap: The ClickOps Problem, Solved

Morning everyone. Today is the payoff for a complaint we've been making since day one of this course.

Cast your mind back to the **Azure Fundamentals** session. We built a resource group and a storage account by hand in the Portal, and then I asked you to recreate it identically in a fresh subscription. The honest answer was "from memory, slowly, probably getting something slightly wrong." We called that **ClickOps**, and we listed its problems: not repeatable, no audit trail of *why*, environments drift apart, no review process, and the knowledge lives in one person's head.

Then in the **bash** session you started writing that knowledge down as scripts. And in the **Jenkins** session you saw those scripts run automatically, gated by tests, on every push.

The missing piece is this: **your scripts told the computer the steps. Terraform lets you describe the result.** You don't write "create a storage account, then check whether it already exists, then maybe update it" — you write "there should be a storage account, configured like this", and Terraform works out what needs doing to make reality match.

Today you'll go from an empty folder to a configuration that provisions real Azure resources, and you'll understand the single concept that trips up nearly everyone learning Terraform: **state**.

Lunch is at 1 for an hour, breaks mid-morning and mid-afternoon. And a standing rule for today: **anything we create, we destroy before we leave**. Cloud resources cost money.

**Everyone make the working folder now, so we're all in the same place:**

*(Run from `~/`)*
- Run: `mkdir -p ~/terraform-training` → **cd inside** with `cd ~/terraform-training`
- Run: `az --version` — confirm the Azure CLI responds. Flag it now if it doesn't.

<br>
<br>

### 09:15–10:00 — What Terraform Is, and the Three States

**Terraform** is an **Infrastructure as Code** tool. Its job is to create, change and delete infrastructure — virtual machines, load balancers, storage, databases, users — based on configuration files you write and keep in Git.

*REFER TO RESOURCE 1 - SLIDEE* <br>

![intro-to-terraform-1](./resources/intro-to-terraform-1.png)

**Where it sits in the toolchain.** Look at that diagram: Terraform lives at the **Provision Server** step. It's excellent at *bringing infrastructure into existence*. It offers some basic tooling for configuring what's *inside* a server, but that's not really its job — installing and configuring software is the territory of **configuration management** tools like Ansible, Chef and Puppet.

**ASK** <br>
Why is that split useful — why not have one tool that both creates a VM *and* installs everything on it? <br>
**ANSWER** <br>
They solve different-shaped problems on different timescales. Infrastructure changes rarely and needs careful, reviewable, all-or-nothing changes. What's installed *inside* a machine changes constantly. Keeping them separate means you can redeploy an application fifty times a day without ever touching the VM definition — and it means the two concerns can be owned, reviewed and versioned independently.

**ASK** <br>
We provisioned a storage account by hand in the Portal weeks ago. Why is doing it in Terraform better? <br>
**ANSWER** <br>
- Removes human error — the configuration is written down and reviewed, rather than re-clicked from memory each time
- Fast to stand resources up in a new region or a new subscription — change one value, re-run
- Just as fast to tear them *down* again, which directly controls cost
- It's a text file, so developers can work in the editor and workflow they already know, with Git history and pull requests

#### The concept that everything depends on: three states

This is the most important 10 minutes of the day. Terraform is constantly reconciling **three** different pictures of your infrastructure.

**1. Desired State** — what you *want*. This is your `.tf` files. "There should be a storage account called X, in resource group Y, with versioning on."

**2. Actual State** — what genuinely exists in Azure right now. If someone went into the Portal and clicked something, this is where that change lives.

**3. Known State** — what Terraform *believes* it created last time it ran. This is stored in a file called `terraform.tfstate`.

When you run `terraform apply`, Terraform reads your Desired State, checks its Known State to find out which real Azure resources it's responsible for, refreshes against the Actual State, works out the difference, and makes the minimum set of changes needed.

**ASK** <br>
That third one seems redundant. If Terraform can just go and *look* at what's really in Azure, why does it need to remember what it did last time? <br>
**ANSWER** <br>
Three reasons, and we'll see all of them today. **(1) Mapping** — your config calls something `my_storage_account`, but Azure calls it a long resource ID. The state file is the lookup table joining those two names; without it Terraform has no idea which Azure resource your block refers to, so it assumes it must create a new one. **(2) Metadata and dependencies** — it records the order things were created in, so it can destroy them in the right order. **(3) Performance** — asking Azure about every resource on every run is slow. State acts as a cache.

Hold onto that. We'll break it deliberately later this morning and watch what happens.

#### Declarative, not imperative

One more framing. Your bash scripts were **imperative** — a list of steps, in order, that you executed. Terraform is **declarative** — you describe the end result and it figures out the steps.

The practical difference: if you tell Terraform "I want 3 users" and 2 already exist, it creates **one**. You never wrote "add one more". You stated the desired total, and Terraform did the subtraction. We'll see that in action this afternoon.

**NOTE FOR TRAINERS** <br>
If students have done any React, the analogy that lands hardest is: writing Terraform is like writing a React component's render output — you describe what the UI should look like for a given state, and React diffs it against the DOM and patches the difference. You don't write "remove this div, then add that one". Terraform is that idea, applied to cloud infrastructure. <br>
**END OF NOTE**

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>

### 10:15–11:15 — Setup: Install, Service Principal, Credentials, `init`
*(Exercise)*

Let's get a working Terraform installation talking to your Azure subscription. There are four steps: install Terraform, create a Service Principal, expose its credentials as environment variables, and initialise a project.

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

Full instructions if anything goes wrong: [Install Terraform](https://developer.hashicorp.com/terraform/tutorials/azure-get-started/install-cli).

#### Step 2 — Create a Service Principal

We need Terraform to be able to authenticate to Azure without a human sitting there. That's exactly what a **Service Principal** is — you met these in the Azure Fundamentals session: an identity in Azure AD representing an application or automated process rather than a person.

First, sign in:

*(Run from `~/terraform-training`)*
```bash
az login
```

That opens a browser, and afterwards prints the subscriptions your account can see. Grab your subscription ID:

*(Run from `~/terraform-training`)*
```bash
az account show --query id -o tsv
```

`--query id` pulls just the `id` field out of the JSON response (it's a JMESPath query), and `-o tsv` prints it as a bare string rather than a quoted JSON value — handy for copying.

Now create the Service Principal:

*(Run from `~/terraform-training`)*
```bash
az ad sp create-for-rbac --name "terraform-training-sp" --role="Contributor" --scopes="/subscriptions/<your-subscription-id>"
```

Breaking that down — most of it is revision:
- `az ad sp` — Azure AD, Service Principal
- `create-for-rbac` — create it *and* assign it a role in one go
- `--name` — a human-friendly display name
- `--role="Contributor"` — Contributor can create, change and delete resources, but **cannot** grant permissions to others. That's what Terraform needs
- `--scopes="/subscriptions/<id>"` — where that role applies. Here, the whole subscription

![intro-to-terraform-2](./resources/intro-to-terraform-2.png)

It returns a JSON block. Three fields matter:

| Returned field | What Terraform calls it |
|---|---|
| `appId` | Client ID |
| `password` | Client Secret |
| `tenant` | Tenant ID |

**Copy these somewhere safe right now.** The `password` is shown **once and never again** — if you lose it you must generate a new secret, and the old one stops working.

**ASK** <br>
We gave this Service Principal `Contributor` across the *entire subscription*. Thinking back to the least-privilege discussion in the Azure session — is that the right call? <br>
**ANSWER** <br>
For a training subscription, it's pragmatic. For anything real, no — you'd scope it to the specific resource group(s) Terraform manages, so leaked credentials can't touch anything else. It's worth being explicit that we're taking a shortcut here, so nobody copies this into production.

#### Step 3 — Expose the credentials as environment variables

The `azurerm` provider can read credentials directly from a `provider` block in your `.tf` file — and the HashiCorp docs explicitly warn you not to.

![intro-to-terraform-5](./resources/intro-to-terraform-5.png)
![intro-to-terraform-6](./resources/intro-to-terraform-6.png)

**ASK** <br>
Why is hard-coding those four values into `main.tf` a genuinely bad idea? <br>
**ANSWER** <br>
Because `main.tf` goes into Git. A client secret committed to a repository is a real security incident — and it's permanent, because Git keeps history even after you "delete" the line. This is the same lesson as the Jenkins credentials vault: secrets live *outside* the code that uses them.

So we use **environment variables**. The provider looks for four specific names automatically.

**NOTE FOR STUDENTS** <br>
Run these from **VS Code's integrated terminal**, and run Terraform from that same terminal. Environment variables set in one terminal don't exist in another — this catches people out constantly. <br>
**END OF NOTE**

*REFER TO RESOURCE 2 - SLIDEE* <br>

*(Run from `~/terraform-training`, in VS Code's terminal — Mac/Linux)*
```bash
export ARM_CLIENT_ID=<appId-from-the-az-command>
export ARM_CLIENT_SECRET=<password-from-the-az-command>
export ARM_SUBSCRIPTION_ID=<your-subscription-id>
export ARM_TENANT_ID=<tenant-from-the-az-command>
```

*(Windows — use `set` for this session, or `setx` to persist)*
```bash
set ARM_CLIENT_ID=<appId-from-the-az-command>
```

To make them permanent on Mac/Linux, add those four lines to `~/.zshrc` (or `~/.bashrc`) — the file that runs every time a new shell opens. That's the same trick you used to put `~/bin` on your `PATH` in the bash session.

**.zshrc**
```
export ARM_CLIENT_ID="2b1e4c6a-8f3d-4a1b-9c7e-5d2f1a3b4c5d"
export ARM_CLIENT_SECRET="Qx9~vK2pL.8mN4rT7wZ1uY6bA3sC0dEf"
export ARM_SUBSCRIPTION_ID="7c3a9e21-1b4d-4f6a-9c8e-2d5f7a1b3c6e"
export ARM_TENANT_ID="9f4b1c2a-3d5e-4a6b-8c7d-1e2f3a4b5c6d"
```

Quick reference for managing environment variables:

**Mac / Linux**
- `export` on its own — list every environment variable
- `export NAME=value` — set one
- `unset NAME` — delete one

**Windows**
- `printenv` (Git Bash) or `set` — list them
- `setx NAME value` — set one persistently
- `setx NAME ""` — clear one

#### Step 4 — Create a project and initialise it

Terraform uses configuration files ending in **`.tf`**.

*(Run from `~/terraform-training`)*
- Run: `mkdir 01-terraform-basics` → **cd inside** with `cd 01-terraform-basics`

*(Run from `~/terraform-training/01-terraform-basics`)*
- Run: `touch main.tf`
- Then open it: `code main.tf`

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

Let's read that properly, because it's your first HCL and every line is doing something:

- `terraform { }` — settings **about Terraform itself**, not about your infrastructure
- `required_providers { }` — which plugins this project needs. A **provider** is the piece that knows how to talk to a specific API. Azure, AWS, GitHub, Cloudflare, Datadog — all separate providers
- `azurerm = { ... }` — `azurerm` is the local name we'll use in resource types. It's Azure **R**esource **M**anager
- `source = "hashicorp/azurerm"` — where to download it from in the Terraform Registry
- `version = "~> 3.0"` — the **pessimistic constraint operator**. `~> 3.0` means "3.0 or higher, but stay below 4.0". It lets you take bug fixes automatically without being broken by a major version that changes the syntax
- `provider "azurerm" { }` — configuration *for* that provider once it's downloaded
- `features {}` — a quirk of the Azure provider specifically: it's mandatory, and it's usually empty. It exists so Azure can add opt-in behaviours later. Just include it

Note that unlike some providers, `location` is **not** set here. In Azure, location is set on each individual resource, starting with the resource group.

**Take a look at what else exists.** Search for "terraform providers" and open the [Terraform Registry](https://registry.terraform.io/browse/providers). Terraform isn't an Azure tool — it's a *provisioning* tool with hundreds of providers. The registry describes them as being responsible for understanding API interactions and exposing resources. Bookmark it; it's the documentation you'll live in.

Installing Terraform did **not** give you the Azure provider. You have to download it per-project:

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform init
```

The output tells you it has:
- **Initialised the backend** — where state will be stored. Right now, a local file. (We'll mention remote backends later)
- **Initialised provider plugins** — downloaded the `azurerm` plugin at a version matching your constraint

Two new things appeared in your folder:

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
ls -la
du -sh .terraform
```

- **`.terraform/`** — a hidden folder holding the downloaded provider binary. Run that `du -sh` and notice it's *hundreds of megabytes*. Every Terraform project you init downloads its own copy, so they add up fast on your laptop
- **`.terraform.lock.hcl`** — the dependency lock file. Conceptually identical to `package-lock.json`: it pins the exact provider versions so everyone on the team, and your CI pipeline, resolves to the same thing

**HANDS ON (remaining time)** <br>
1. Install Terraform and confirm `terraform --version` works.
2. `az login`, and note down your **subscription ID**.
3. Create a Service Principal named `terraform-training-sp` with `Contributor` on your subscription. Save the `appId`, `password` and `tenant` somewhere safe.
4. Export all four `ARM_*` environment variables in VS Code's terminal, and confirm with `export | grep ARM`.
5. Create `~/terraform-training/01-terraform-basics/main.tf` with the provider config above, and run `terraform init`.
6. Look inside `.terraform/` and check how large it is.
**END OF NOTE**

**Solution**

*(Run from `~/terraform-training`)*
```bash
# 1. Install (Mac)
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
terraform --version

# 2. Sign in and get the subscription ID
az login
az account show --query id -o tsv

# 3. Create the Service Principal (paste your own subscription ID)
az ad sp create-for-rbac --name "terraform-training-sp" --role="Contributor" --scopes="/subscriptions/7c3a9e21-1b4d-4f6a-9c8e-2d5f7a1b3c6e"
```

*(Run from `~/terraform-training`, in VS Code's terminal)*
```bash
# 4. Export the credentials, then confirm
export ARM_CLIENT_ID=2b1e4c6a-8f3d-4a1b-9c7e-5d2f1a3b4c5d
export ARM_CLIENT_SECRET='Qx9~vK2pL.8mN4rT7wZ1uY6bA3sC0dEf'
export ARM_SUBSCRIPTION_ID=7c3a9e21-1b4d-4f6a-9c8e-2d5f7a1b3c6e
export ARM_TENANT_ID=9f4b1c2a-3d5e-4a6b-8c7d-1e2f3a4b5c6d

export | grep ARM
```

*(Note the **single quotes** around the secret — client secrets frequently contain `~`, `!` or `$`, which bash would otherwise try to interpret. This is the quoting lesson from the bash session, biting for real.)*

*(Run from `~/terraform-training`)*
```bash
# 5. Create the project and initialise
mkdir 01-terraform-basics && cd 01-terraform-basics
```

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
touch main.tf
# ...paste the provider config into main.tf...
terraform init

# 6. Inspect what appeared
ls -la
du -sh .terraform
```

<br>
<br>
### 11:15–12:15 — Your First Resources: Resource Group & Storage Account
*(Exercise)*

Time to create something real.

#### Creating a Resource Group

Before you can create almost anything in Azure, you need somewhere to put it. Azure organises resources into **Resource Groups** — a logical container grouping related resources together.

**ASK** <br>
From the Azure Fundamentals session — why does Azure have this extra layer? What does it give you? <br>
**ANSWER** <br>
A single unit for managing lifecycle, permissions, cost tracking and clean-up for a related set of resources. Deleting a resource group deletes everything inside it, which is exactly why you group things by *shared lifecycle* rather than by convenience.

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

**The resource block syntax — learn this shape, everything else today is a variation of it:**

```tf
resource "<provider>_<resource_type>" "<internal_name>" {
  <attribute> = <value>
}
```

Three parts, and the distinction between the last two matters enormously:

1. `resource` — the keyword. "I want to manage a thing in the cloud"
2. `"azurerm_resource_group"` — the **type**. Provider name, underscore, resource type. This is fixed vocabulary defined by the provider — you can't invent it, you look it up in the registry docs
3. `"my_resource_group"` — the **internal name**, sometimes called the local name or label. **You choose this.** It only exists inside Terraform. Nothing in Azure will ever be called this

Then inside the braces, the attributes. Note `name = "rg-emilesherrott-devops"` — *that's* the name Azure sees.

**ASK** <br>
So we have two names. What's the difference between `"my_resource_group"` and `name = "rg-emilesherrott-devops"`? <br>
**ANSWER** <br>
`my_resource_group` is Terraform's internal handle — how *your other configuration* refers to this resource. `name` is what actually gets created in Azure and shows up in the Portal. This distinction becomes critical in about ten minutes when we look at the state file, because Terraform uses the internal name as its lookup key.

#### Creating a Storage Account

**ASK** <br>
What's Azure's equivalent of an S3 bucket? <br>
**ANSWER** <br>
Azure splits it into two things: a **Storage Account**, the top-level container for storage resources, and a **Blob Container** inside it, where the actual files (blobs) live.

Think of a blob container as a file store — provide a **key** (the blob name) and store a **file** as the value. Azure Storage gives very high **durability** (once data's in, you're very unlikely to lose it) and high **availability** (you can reach it easily).

Let's create one in the Portal first, just to see what we're automating.

*(In the Azure Portal — home)*
- **Create a resource → Storage account**
- Try the name `storage` — you'll be told it already exists

Storage account names must be **globally unique across all of Azure**, and are more restrictive than S3: **lowercase letters and numbers only**, no hyphens, no underscores, 3–24 characters.

- Use something like `stemilesherrottdevops`
- Leave everything else default (Standard performance, LRS redundancy)
- **Review + create** → **Create**

![intro-to-terraform-9](./resources/intro-to-terraform-9.png)

Click into it and then **Containers** — that's where files would go. Fine. Now let's do the same thing properly, in code.

*(Run from `~/terraform-training/01-terraform-basics`)* — add to `main.tf`:

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
  name     = "rg-emilesherrott-devops"
  location = "uksouth"
}

# NEW CODE
resource "azurerm_storage_account" "my_storage_account" {
  name                     = "stemilesherrottdevops01"
  resource_group_name      = azurerm_resource_group.my_resource_group.name
  location                 = azurerm_resource_group.my_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```

Two lines here are doing something genuinely new and important:

```tf
resource_group_name = azurerm_resource_group.my_resource_group.name
location            = azurerm_resource_group.my_resource_group.location
```

That's a **reference**. Instead of typing `"rg-emilesherrott-devops"` again, we're pointing at the *other resource block* and reading its `name` attribute. The syntax is:

```
<resource_type>.<internal_name>.<attribute>
```

Notice there are **no quotes** around it — quotes would make it a literal string. This is an expression, so it's bare.

**ASK** <br>
Why is referencing the resource group better than just typing the name string again? <br>
**ANSWER** <br>
Two reasons. **(1) Single source of truth** — rename the resource group in one place and everything referring to it follows. **(2) Dependency ordering**, which is the big one. By referencing it, you've told Terraform "this storage account depends on that resource group", so Terraform automatically builds a dependency graph and creates the resource group **first**. You never have to specify the order. Break the reference and hard-code the string, and Terraform might try to create the storage account before the container exists to hold it.

The remaining attributes: `account_tier = "Standard"` (versus Premium, which is SSD-backed and pricier) and `account_replication_type = "LRS"` — Locally Redundant Storage, meaning three copies within one datacentre. The cheapest option, and fine for training. `GRS` would replicate to the paired region, which is the disaster-recovery idea from the Azure session.

#### The two-step execution approach

Terraform has a sensible workflow: **check what would happen, then do it.**

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform plan
```

![intro-to-terraform-10](./resources/intro-to-terraform-10.png)

`plan` compares Desired State against Known State (refreshing against Actual State) and tells you exactly what it *would* do — without doing anything. Read the output together:

- `+` means create, `-` means destroy, `~` means update in place
- It'll create `azurerm_resource_group.my_resource_group` and `azurerm_storage_account.my_storage_account`
- Lots of attributes show `(known after apply)` — `primary_access_key`, `primary_blob_endpoint`, `id` and so on. Terraform can't know those until Azure has actually made the thing
- The only values it *can* show you are the ones you wrote yourself

Now execute:

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform apply
```

![intro-to-terraform-11](./resources/intro-to-terraform-11.png)

`apply` shows the same plan again, then stops and asks for confirmation. At the bottom you get the summary line: **Plan: 2 to add, 0 to change, 0 to destroy**. The only input it accepts is the literal word `yes`.

- Type: `yes`

![intro-to-terraform-12](./resources/intro-to-terraform-12.png)

If that worked, your credentials are correct and Terraform has real access to your subscription.

*(In the Azure Portal — Resource groups)*
- Open **rg-emilesherrott-devops** and refresh
- Your storage account is there

![intro-to-terraform-13](./resources/intro-to-terraform-13.png)

**NOTE FOR TRAINERS** <br>
If it fails, it's almost always one of two things: **credentials** (an `ARM_*` variable missing, or set in a different terminal) or a **non-unique storage account name**. Have students check `export | grep ARM` first, then try a more obscure storage account name. <br>
**END OF NOTE**

**HANDS ON (20 min)** <br>
*(Run from `~/terraform-training/01-terraform-basics`)*
1. Add the resource group and storage account blocks to `main.tf` — use **your own** name prefix, not mine.
2. Run `terraform plan` and read the output. Find three attributes marked `(known after apply)` and say why Terraform can't know them yet.
3. Run `terraform apply` and confirm with `yes`.
4. *(In the Azure Portal)* Verify both resources exist.
5. Run `terraform apply` a **second** time, with no changes. What does it say, and why?
**END OF NOTE**

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

terraform apply     # run it a second time
```

**Step 2 answer:** `id`, `primary_access_key` and `primary_blob_endpoint` are all generated *by Azure* at creation time. Terraform is planning ahead of that happening, so it honestly reports that it doesn't know them yet.

**Step 5 answer:** the second run reports **"No changes. Your infrastructure matches the configuration."** Desired State and Known State agree, and refreshing shows the Actual State agrees too — so there's nothing to do. That's the declarative model working: running `apply` repeatedly is safe.

<br>
<br>

### 12:15–13:00 — State, the Console, and Outputs
*(Exercise + Challenge)*

Now the important bit. Let's dig into what just happened.

#### Changing something that can't be changed

*(Run from `~/terraform-training/01-terraform-basics`)* — change the storage account name in `main.tf`:

**main.tf**
```tf
resource "azurerm_storage_account" "my_storage_account" {
  # UPDATED CODE
  name                     = "stemilesherrottdevops02"
  resource_group_name      = azurerm_resource_group.my_resource_group.name
  location                 = azurerm_resource_group.my_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform apply
```

![intro-to-terraform-15](./resources/intro-to-terraform-15.png)

Read the output carefully: **destroy and then create replacement**, and the line **`must be replaced`**. The summary is **Plan: 1 to add, 0 to change, 1 to destroy**.

Terraform knows a storage account's name **cannot be changed after creation** — that's an Azure constraint, encoded into the provider. So the only way to reach your Desired State is to delete the old one and make a new one.

- Type: `yes`

**ASK** <br>
Why does it matter that Terraform knows *which* attributes force a replacement rather than an in-place update? <br>
**ANSWER** <br>
Because a replacement is destructive. On a storage account holding real data, or a database, "destroy and recreate" is potentially catastrophic — and the plan output is your warning before it happens. This is a very strong argument for always running `plan` and actually *reading* it, especially in production. Terraform having this knowledge baked into the provider is a big part of why it got adopted so widely.

#### Enabling versioning — a property, not a resource

*(In the Azure Portal — your storage account)* — look at **Data protection** under **Data management**. **Blob versioning** is disabled.

Let's turn it on from code. In Azure, blob versioning is a *property* of the storage account:

**main.tf**
```tf
resource "azurerm_storage_account" "my_storage_account" {
  name                     = "stemilesherrottdevops02"
  resource_group_name      = azurerm_resource_group.my_resource_group.name
  location                 = azurerm_resource_group.my_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "LRS"

  # NEW CODE
  blob_properties {
    versioning_enabled = true
  }
}
```

Notice `blob_properties { }` has **no `=`**. That's a **nested block**, not an attribute. In HCL:
- `name = "value"` — an **attribute**: a single named value, uses `=`
- `blob_properties { ... }` — a **block**: a group of related settings, no `=`, uses braces

So the general shape of a resource is:

```tf
resource "<provider>_<resource-type>" "<chosen_resource_name>" {
    name = "<name-provided-to-azure>"
    blob_properties {
        versioning_enabled = true
    }
}
```

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform apply
```

This time it's an **update in place** — `~` rather than `-/+`. Versioning is a property that *can* be changed on a live account, so nothing gets destroyed.

- Type: `yes`

*(In the Azure Portal — your storage account → Data protection)* — Blob versioning now shows **Enabled**.

Here's the declarative point again: **you never wrote "enable versioning".** You declared that versioning should be enabled, and Terraform worked out that the way to get there was an update call.

**NOTE FOR TRAINERS** <br>
Occasionally `terraform.tfstate` lags behind a configuration-only change like this. If the Known State looks stale, `terraform refresh` pulls the latest Actual State from the Azure API and updates it. Worth mentioning so students don't panic if their state file looks out of date. <br>
**END OF NOTE**

#### The terraform console

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform console
```

This is an interactive REPL for querying your configuration and state — like `node` in a terminal, but for Terraform expressions.

Inside the console:
```
azurerm_storage_account.my_storage_account
```

![intro-to-terraform-16](./resources/intro-to-terraform-16.png)

That prints every attribute of the storage account. Remember the syntax — it's a **reference**, same as the one you used to link the two resources:

```
<provider>_<resource-type>.<chosen_resource_name>
```

Critically, you use the **internal name** (`my_storage_account`), *not* the Azure name (`stemilesherrottdevops02`). Terraform doesn't think in Azure names.

Dig deeper:
```
azurerm_storage_account.my_storage_account.blob_properties
```
![intro-to-terraform-17](./resources/intro-to-terraform-17.png)

Notice it comes back wrapped in `[ ]` — it's a **list**. Nested blocks can appear multiple times, so the provider models them as a list even when there's only one. So you index into it, exactly like a JavaScript array:

```
azurerm_storage_account.my_storage_account.blob_properties[0]
```
![intro-to-terraform-18](./resources/intro-to-terraform-18.png)

**ASK** <br>
If we wanted just the value of `versioning_enabled`, how would we get it? <br>
**ANSWER** <br>
`azurerm_storage_account.my_storage_account.blob_properties[0].versioning_enabled` — index into the list, then read the attribute off the object. Same chaining as JavaScript.

Exit the console with `Ctrl + C` (or type `exit`).

#### Outputs

The console is great for exploring, but if there's a value you want *every time*, define an **output**.

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

![intro-to-terraform-19](./resources/intro-to-terraform-19.png)

`-refresh=false` tells Terraform **don't go and ask Azure what's really there** — just compare Desired State against Known State. We know they match, and we only added an output, so skipping the refresh makes it fast. Terraform even says it can apply this plan to save the new output values without changing any real infrastructure.

**Be careful with `-refresh=false` in production** — you're deliberately choosing not to check reality, and reality is where your applications live.

You're not limited to one output:

**main.tf**
```tf
output "my_storage_account_versioning" {
  value = azurerm_storage_account.my_storage_account.blob_properties[0].versioning_enabled
}

# NEW CODE
output "my_storage_account_complete_details" {
  value = azurerm_storage_account.my_storage_account
}
```

**ASK** <br>
Outputs are clearly handy for poking around. What are they actually *for* in a real project? <br>
**ANSWER** <br>
Three things. **Communicating information** — passing values like a blob endpoint or a DNS name to other parts of your infrastructure, or to another Terraform project. **Debugging** — surfacing key facts so you can confirm things provisioned correctly. **Integration with external systems** — this is the big one: a configuration management tool like Ansible, or a Jenkins pipeline stage, can read `terraform output` and use those values. That's how "Terraform provisions it, then something else configures it" actually gets wired together.

#### Adding an Azure AD user

Let's add a resource from a **different provider**, to prove a project isn't limited to one.

**main.tf** — add `azuread` to `required_providers`, add the provider block, and add the user:

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

# NEW CODE
resource "azuread_user" "my_azuread_user" {
  user_principal_name = "my_iam_user_abc@emilesherrottdevops.onmicrosoft.com"
  display_name        = "my_iam_user_abc"
  mail_nickname       = "my_iam_user_abc"
  password            = "ChangeMe123!ChangeMe"
}
```

Note this is the **`azuread`** provider, separate from `azurerm`. Azure splits identity (Azure AD / Entra ID) from resource management, and Terraform mirrors that with two providers. An Azure AD user needs a `user_principal_name` (their sign-in email, on a domain your tenant owns), a `display_name`, a `mail_nickname`, and a `password`.

Because you added a new provider, you must re-initialise:

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform init
```

**NOTE FOR TRAINERS** <br>
Azure AD enforces password complexity by default (upper and lowercase, numbers, symbols, minimum 8 characters). If `apply` rejects the password, tweak it until it passes — and use the moment to say that in a real project you'd generate this with the `random_password` resource rather than hard-coding it in a file destined for Git. <br>
**END OF NOTE**

#### Saving a plan to a file

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform plan -out aduser.tfplan
```

![intro-to-terraform-20](./resources/intro-to-terraform-20.png)

`-out <file>` saves the plan. The conventional extension is `.tfplan`. The output tells you it would create an `azuread_user`, and that the only thing it can be certain of is the name you supplied. It also tells you how to run it: `terraform apply "aduser.tfplan"`.

Try opening `aduser.tfplan` in your editor — it's **binary**, not readable text.

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform apply "aduser.tfplan"
```

![intro-to-terraform-21](./resources/intro-to-terraform-21.png)

Notice: **no `yes` prompt**. It just runs.

**ASK** <br>
Applying a saved plan skips the confirmation. Is that a feature or a hazard? <br>
**ANSWER** <br>
Both, and it depends entirely on context. It's a **feature** in a pipeline — this is precisely how you'd do it in Jenkins: run `plan` in one stage, have a human review the output, then `apply` that *exact* saved plan, guaranteeing nothing changed between review and execution. It's a **hazard** if you saved a plan an hour ago and the world has moved on, because it won't re-check. The safety of a saved plan comes entirely from it being fresh and reviewed.

Now let's see the user's details. Add an output:

**main.tf**
```tf
# NEW CODE
output "my_azuread_user_complete_details" {
  value = azuread_user.my_azuread_user
}
```

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform apply -refresh=false
# type: yes
```

![intro-to-terraform-22](./resources/intro-to-terraform-22.png)

Look at the `id` — that's the user's **Object ID**, a GUID uniquely identifying them in your tenant.

**ASK** <br>
Is that the same idea as an AWS **arn**? <br>
**ANSWER** <br>
Sort of, in that it identifies the resource — but Azure's Object ID is an **opaque GUID**. It doesn't encode the subscription, service or resource type into the string the way an arn does (`arn:aws:iam::725625542800:user/name`). To know what a GUID refers to, you generally have to ask Azure, or already know from context.

#### Updating in place, with a target

*(Run from `~/terraform-training/01-terraform-basics`)* — change `abc` to `def` across all three name fields:

**main.tf**
```tf
# UPDATED CODE
resource "azuread_user" "my_azuread_user" {
  user_principal_name = "my_iam_user_def@emilesherrottdevops.onmicrosoft.com"
  display_name        = "my_iam_user_def"
  mail_nickname       = "my_iam_user_def"
  password            = "ChangeMe123!ChangeMe"
}
```

Terraform knows an Azure AD user's identifying details generally *can* be updated in place — unlike the storage account name. Let's use a shortcut, since we know only one resource is affected:

- Syntax: `terraform apply -target=<resource_type>.<internal_name>`

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform apply -target=azuread_user.my_azuread_user
```

![intro-to-terraform-23](./resources/intro-to-terraform-23.png)

- Type: `yes`

`-target` restricts the operation to one resource (and its dependencies), which saves refreshing everything else. Useful when a project has hundreds of resources — but **use it sparingly**: you're deliberately ignoring part of your configuration, so your state can end up partially applied.

#### The state files, in depth

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
ls -la
```

![intro-to-terraform-24](./resources/intro-to-terraform-24.png)

Two files:
- **`terraform.tfstate`** — the current Known State
- **`terraform.tfstate.backup`** — the *previous* Known State

Open `terraform.tfstate`. Two top-level keys matter:

**`outputs`** — the values of every `output` you defined.

**`resources`** — an array describing everything Terraform manages: type, internal name, current attribute values, and metadata.

Now compare: in `terraform.tfstate` the AD user is `def`; in `terraform.tfstate.backup` it's still `abc`. Every successful `apply` moves the old state into `.backup` and writes the new state to `terraform.tfstate`. It's a safety net for a corrupted apply.

**You should never edit these files by hand.** So naturally, let's break one on purpose.

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
mv terraform.tfstate terraform.tfstate.001
mv terraform.tfstate.backup terraform.tfstate.backup.001
ls
```

There is now no file Terraform recognises as holding Known State.

**ASK** <br>
What do we think happens if we run a plan now? <br>
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

Minimal changes again. Sanity restored.

**Why this happened — the mapping.** Look at the state file:

**terraform.tfstate**
```tf
"resources": [
  {
    "mode": "managed",
    "type": "azuread_user",
    # HERE
    "name": "my_azuread_user",
    "provider": "provider[\"registry.terraform.io/hashicorp/azuread\"]",
    "instances": [
      {
        "schema_version": 0,
        "attributes": {
          "account_enabled": true,
          # HERE - the Object ID, Azure AD's identifier
          "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
          "user_principal_name": "my_iam_user_def@emilesherrottdevops.onmicrosoft.com",
          "display_name": "my_iam_user_def",
          "mail_nickname": "my_iam_user_def",
          "department": null,
          "country": null,
          "force_password_change": false
        },
        "sensitive_attributes": [],
        "private": "bnVsbA=="
      }
    ]
  },
[ . . . ]
```

Terraform finds the resource by its **`name`** (your internal name), then reads the **`id`** from `instances` to know which real object in Azure it maps to. That's the join. Without it, `my_azuread_user` means nothing — there's no way to connect your config block to that GUID, or even to know Terraform created it.

This is also why the Azure AD users you clicked into existence in earlier sessions are invisible to Terraform: they were never in the state file, so Terraform doesn't manage them.

**HANDS ON (20 min)** <br>
*(Run from `~/terraform-training/01-terraform-basics`)*
1. Change your storage account name and `apply`. Confirm the plan says **must be replaced** and note the add/destroy counts.
2. Add the `blob_properties` block with `versioning_enabled = true`, apply, and verify in the Portal under **Data protection**.
3. Open `terraform console` and retrieve the value of `versioning_enabled` in a single expression.
4. Add outputs for the versioning flag and the full storage account details, then `terraform apply -refresh=false`.
5. Rename both state files, run `terraform plan`, and explain what you see. Then rename them back.
**END OF NOTE**

**Solution**

**main.tf** (steps 1, 2 and 4):
```tf
resource "azurerm_storage_account" "my_storage_account" {
  name                     = "stjbloggsdevops02"          # step 1: changed -> forces replacement
  resource_group_name      = azurerm_resource_group.my_resource_group.name
  location                 = azurerm_resource_group.my_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "LRS"

  # step 2
  blob_properties {
    versioning_enabled = true
  }
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
Then `Ctrl + C` to exit.

```bash
# Step 4
terraform apply -refresh=false

# Step 5
mv terraform.tfstate terraform.tfstate.001
mv terraform.tfstate.backup terraform.tfstate.backup.001
terraform plan          # -> proposes creating EVERYTHING from scratch
mv terraform.tfstate.001 terraform.tfstate
mv terraform.tfstate.backup.001 terraform.tfstate.backup
terraform plan          # -> "No changes" (or minimal changes)
```

**Step 5 explanation:** with no state file, Terraform has no mapping between your internal names and the real Azure object IDs. It assumes nothing exists and plans to create it all. The resources are still in Azure — Terraform has simply forgotten it owns them.

---

**Challenge**

*Direct* students, **in pairs**, to extend their configuration so that:

* A **second** storage account is created in the same resource group, with a different name
* The second storage account uses `account_replication_type = "GRS"` instead of `LRS`
* Both storage accounts reference the resource group rather than hard-coding its name
* An **output** called `all_blob_endpoints` returns a **list** containing the `primary_blob_endpoint` of *both* storage accounts
* **OPTIONAL** — add a second output that returns only the **first** endpoint from that list

*Provide* this example output shape as an aid:

```
all_blob_endpoints = [
  "https://stjbloggsdevops02.blob.core.windows.net/",
  "https://stjbloggsdevops03.blob.core.windows.net/",
]
```

*Grant* students ~10 minutes.

Hints to offer if they're stuck: a list in HCL uses square brackets, just like a JavaScript array — `[ thing_one, thing_two ]`. And indexing works the same way too.

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

# OPTIONAL: index into that list
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

Points to draw out when revealing:
* The two resources have **different internal names** (`my_storage_account` and `my_backup_storage_account`) but are the same *type*. The internal name is what distinguishes them.
* Building a list in an output with `[ ]` is exactly the JavaScript array literal syntax — HCL borrows heavily from things they already know.
* Both blocks reference the resource group, so Terraform knows to create it first and can safely create the two storage accounts **in parallel** — they don't depend on each other.

<br>
<br>

*(Take a 1 hour lunch break here.)*

<br>
<br>
### 14:00–14:45 — Refactoring, `.gitignore`, and Variables
*(Talk + exercise)*

Welcome back. This morning everything went into one `main.tf`. That's fine for five resources and unmanageable for five hundred. Let's tidy up, protect our secrets, and then make the configuration dynamic.

#### Splitting the configuration

*(Run from `~/terraform-training/01-terraform-basics`)*
- Run: `touch outputs.tf`

Move all three `output` blocks out of `main.tf` and into it:

**outputs.tf**
```tf
# NEW CODE
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

![intro-to-terraform-25](./resources/intro-to-terraform-25.png)

**No changes.** Which is exactly right — you moved text between files, you didn't change what you're asking for.

**The rule is simple: Terraform reads *every* `.tf` file in the current directory and concatenates them.** File names are purely for humans. `main.tf`, `outputs.tf`, `banana.tf` — Terraform doesn't care, as long as the extension is `.tf`.

Two things follow from that, and both catch beginners:
- It reads files in the **current directory only**. It does *not* recurse into subfolders. That's why each of our numbered folders is a completely separate, independent project with its own state
- Because everything is concatenated, order doesn't matter — you can reference a resource defined in another file, or defined further down the same file. Terraform builds the dependency graph itself

Let's do the same for resources:

*(Run from `~/terraform-training/01-terraform-basics`)*
- Run: `touch resources.tf`

**resources.tf**
```tf
# NEW CODE
resource "azurerm_resource_group" "my_resource_group" {
  name     = "rg-emilesherrott-devops"
  location = "uksouth"
}

resource "azurerm_storage_account" "my_storage_account" {
  name                     = "stemilesherrottdevops02"
  resource_group_name      = azurerm_resource_group.my_resource_group.name
  location                 = azurerm_resource_group.my_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "LRS"

  blob_properties {
    versioning_enabled = true
  }
}

resource "azuread_user" "my_azuread_user" {
  user_principal_name = "my_iam_user_def@emilesherrottdevops.onmicrosoft.com"
  display_name        = "my_iam_user_def"
  mail_nickname       = "my_iam_user_def"
  password            = "ChangeMe123!ChangeMe"
}
```

*REMOVE THE COPIED CONFIG FROM `main.tf`*

`main.tf` now contains only the `terraform` and `provider` blocks. The conventional layout you'll see in real projects is `main.tf` (providers and backend), `variables.tf`, `resources.tf` and `outputs.tf`.

#### Protecting state with .gitignore

**ASK** <br>
We've established the state file is important and the team needs it. So we should commit `terraform.tfstate` to GitHub… shouldn't we? <br>
**ANSWER** <br>
**No.** State files store attribute values **unencrypted** — including secrets. Our AD user's password is sitting in that file in plain text right now. Committing it to a repository, especially a public one, hands those secrets to anyone who can read it.

The proper solution is a **remote backend** — storing state in an Azure Storage Account blob container, so the whole team pulls the same authoritative state and it's never in Git. We'll implement that in a later session. Today, the takeaway is simply: **state must not be publicly available.**

*(Run from `~/terraform-training`)* — note this is the **parent** folder, so it covers every project
- Run: `touch .gitignore`

**.gitignore**
```gitignore
*.tfstate
*.tfstate.backup
**/.terraform/
```

- `*.tfstate` — any state file
- `*.tfstate.backup` — and its backup
- `**/.terraform/` — the provider download folder in *any* subdirectory. `**` means "at any depth". You'd exclude this anyway: it's hundreds of megabytes of binaries that `terraform init` can re-download

#### Variables

So far everything has been a hard-coded **constant**. Let's make it dynamic.

*(Run from `~/terraform-training/01-terraform-basics`)* — add to `main.tf`:

**main.tf**
```tf
# NEW CONFIG
variable "iam_user_name_prefix" {
  default = "my_iam_user"
}
```

And use it in the resource:

```tf
resource "azuread_user" "my_azuread_user" {
  # UPDATED CONFIG
  user_principal_name = "${var.iam_user_name_prefix}_def@emilesherrottdevops.onmicrosoft.com"
  display_name        = "${var.iam_user_name_prefix}_def"
  mail_nickname       = "${var.iam_user_name_prefix}_def"
  password            = "ChangeMe123!ChangeMe"
}
```

Two pieces of new syntax:

**Declaring** — `variable "iam_user_name_prefix" { default = "my_iam_user" }`. The block declares the variable exists and gives it a fallback value.

**Using** — `${var.iam_user_name_prefix}`. You must write the `var.` prefix; that's how Terraform knows you mean a variable rather than a resource. And `${ ... }` is **string interpolation**.

**ASK** <br>
Where have we seen `${ }` syntax before? <br>
**ANSWER** <br>
JavaScript template literals — and in the Jenkins session too. Same concept: drop the value of an expression into the middle of a string. (Bash used `${VAR}` as well.) HCL borrows the idea wholesale.

One subtlety worth stating: you only need `${ }` when the value is *inside a larger string*. If the whole value is just the variable, write it bare:

```tf
display_name = var.iam_user_name_prefix          # correct
display_name = "${var.iam_user_name_prefix}"     # works, but redundant
```

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform apply -refresh=false
```

![intro-to-terraform-35](./resources/intro-to-terraform-35.png)

No changes — the interpolated result is identical to the constant it replaced.

Check it in the console:

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform console
```
```
var.iam_user_name_prefix
```
![intro-to-terraform-36](./resources/intro-to-terraform-36.png)

Exit with `Ctrl + C`.

#### Typing variables

```tf
variable "iam_user_name_prefix" {
  # NEW CONFIG
  type    = string
  default = "my_iam_user"
}
```

`type` constrains what the variable is allowed to hold. The available types:

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

Watch what happens with a mismatch:

```tf
variable "iam_user_name_prefix" {
  # UPDATED CONFIG
  type    = number
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
terraform validate
```
![intro-to-terraform-38](./resources/intro-to-terraform-38.png)

Valid! Interesting. A variable without a default isn't an error — it's just *required*. Push further:

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform apply -refresh=false
```
![intro-to-terraform-39](./resources/intro-to-terraform-39.png)

Terraform **prompts you interactively** for the value.

- Type: `my_iam_user`

![intro-to-terraform-40](./resources/intro-to-terraform-40.png)

No changes needed.

**ASK** <br>
That interactive prompt is quite friendly. Where would it be a serious problem? <br>
**ANSWER** <br>
In a **pipeline**. A Jenkins build has no human to type an answer, so it would hang until it timed out. It's the same trap as the `read -p` prompt in the bash session: convenient by hand, fatal in automation. In CI you always supply variables non-interactively — which is exactly what we're about to cover.

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

The environment variable overrides the default. **The naming rule is `TF_VAR_` followed by the exact variable name.**

Remove it again:
- Mac/Linux: `unset TF_VAR_iam_user_name_prefix`
- Windows: `setx TF_VAR_iam_user_name_prefix ""`

**NOTE FOR TRAINERS** <br>
Flag this hard: if a student ever sees a `plan` proposing changes they can't explain, a forgotten `TF_VAR_` environment variable in that terminal is a very common culprit. It's invisible in the code. <br>
**END OF NOTE**

**3. A `terraform.tfvars` file:**

*(Run from `~/terraform-training/01-terraform-basics`)*
- Run: `touch terraform.tfvars`

**terraform.tfvars**
```tf
# NEW CODE
iam_user_name_prefix = "VALUE_FROM_TERRAFORM_TFVARS"
```

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform plan -refresh=false
```
![intro-to-terraform-42](./resources/intro-to-terraform-42.png)

Note there's no `variable` keyword in a `.tfvars` file — just `name = value`. The variable is *declared* in `.tf`; it's *assigned* here.

**4. On the command line:**

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform plan -refresh=false -var="iam_user_name_prefix=VALUE_FROM_CLI"
```
![intro-to-terraform-43](./resources/intro-to-terraform-43.png)

The CLI wins over everything.

**Precedence, from lowest to highest:**
1. `default` in the variable block
2. `TF_VAR_` environment variables
3. `terraform.tfvars`
4. `-var` on the command line

There's also `-var-file`, for pointing at a specific file:
- Syntax: `terraform apply -var-file="<file-name>.tfvars"`

That's how teams manage environments: `dev.tfvars`, `test.tfvars`, `prod.tfvars` — one configuration, three sets of values.

#### Why variables actually matter

The reason we're spending time on this: **the same Terraform configuration gets used across multiple environments.** In DevOps you'll provision Development, Test and Production resources — and you do *not* want three copies of the code that can drift apart.

A very common pattern is an `environment` variable woven into resource names:

```tf
# NEW CONFIG
variable "environment" {
  default = "dev"
}

variable "iam_user_name_prefix" {
  type    = string
  default = "my_iam_user"
}

resource "azuread_user" "my_azuread_user" {
  # UPDATED CONFIG
  user_principal_name = "${var.environment}_${var.iam_user_name_prefix}_def@emilesherrottdevops.onmicrosoft.com"
  display_name        = "${var.environment}_${var.iam_user_name_prefix}_def"
  mail_nickname       = "${var.environment}_${var.iam_user_name_prefix}_def"
  password            = "ChangeMe123!ChangeMe"
}
```

Here it just prefixes names, making it obvious which environment a resource belongs to. More practically, you'd also use it to vary *scale* — one small VM in test, six large ones in production — and **location**, standing up the same stack in a different region by changing one value.

*DELETE THE `environment` VARIABLE AND REVERT THE NAMES*

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform validate
```

#### A few more commands worth knowing

**`terraform fmt`** — auto-formats every `.tf` file in the current directory to the standard style (2-space indentation, aligned `=`). Run it before committing; it removes an entire category of pointless code-review comments.

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform fmt
```

**`terraform show`** — prints the Known State in a readable form, much friendlier than opening the raw JSON.

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
terraform show
```
![intro-to-terraform-30](./resources/intro-to-terraform-30.png)

**`terraform validate`** — checks syntax and internal consistency without contacting Azure. Fast, and pipeline-friendly.

**HANDS ON (15 min)** <br>
*(Run from `~/terraform-training/01-terraform-basics`)*
1. Split your configuration into `main.tf`, `resources.tf` and `outputs.tf`. Confirm with `terraform plan -refresh=false` that nothing changed.
2. Create the `.gitignore` in `~/terraform-training`.
3. Introduce a variable `iam_user_name_prefix` with a `type` and a `default`, and use it in the AD user's name fields.
4. Override it three ways in turn — a `TF_VAR_` environment variable, a `terraform.tfvars` file, and `-var` on the CLI — running `terraform plan -refresh=false` each time to watch the precedence.
5. Deliberately break something (misspell `var.` or a resource type) and confirm `terraform validate` catches it. Then run `terraform fmt`.
**END OF NOTE**

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
  user_principal_name = "${var.iam_user_name_prefix}_def@jbloggsdevops.onmicrosoft.com"
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

*(Run from `~/terraform-training`)*
```bash
cat > .gitignore << 'EOF'
*.tfstate
*.tfstate.backup
**/.terraform/
EOF
```

*(Run from `~/terraform-training/01-terraform-basics`)*
```bash
# Step 1
terraform plan -refresh=false          # -> No changes

# Step 4 — watch precedence, lowest to highest
terraform plan -refresh=false          # uses the default

export TF_VAR_iam_user_name_prefix=FROM_ENV
terraform plan -refresh=false          # env var beats the default

echo 'iam_user_name_prefix = "FROM_TFVARS"' > terraform.tfvars
terraform plan -refresh=false          # tfvars beats the env var

terraform plan -refresh=false -var="iam_user_name_prefix=FROM_CLI"   # CLI beats everything

# Tidy up
unset TF_VAR_iam_user_name_prefix
rm terraform.tfvars

# Step 5
terraform validate
terraform fmt
```

<br>
<br>

### 14:45–15:00 — Creating Many Resources with `count`
*(Talk + demo)*

Everything so far has been one block, one resource. Let's create many.

Start a fresh project so you don't disturb what you've built:

*(Run from `~/terraform-training/01-terraform-basics`)*
- Run: `cd ..`

*(Run from `~/terraform-training`)*
- Run: `mkdir 02-count` → **cd inside** with `cd 02-count`

*(Run from `~/terraform-training/02-count`)*
- Run: `touch main.tf`

**02-count/main.tf**
```tf
# NEW CODE
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
  user_principal_name = "my_iam_user_${count.index}@emilesherrottdevops.onmicrosoft.com"
  display_name        = "my_iam_user_${count.index}"
  mail_nickname       = "my_iam_user_${count.index}"
  password            = "ChangeMe123!ChangeMe"
}
```

Note the internal name is now **plural** — `my_azuread_users`. It represents many things, so the name should say so.

`count = 2` is a **meta-argument** — an argument Terraform itself understands, available on *any* resource type regardless of provider. It says "make this many copies of this block".

`count.index` is then available inside the block, holding the current iteration number, **starting at 0**. So with `count = 2` you get `my_iam_user_0@...` and `my_iam_user_1@...`. Without something varying per copy, you'd be trying to create two identical users — and `user_principal_name` must be unique.

**ASK** <br>
What do we need to run first in this new folder, and why? <br>
**ANSWER** <br>
`terraform init`. Each project directory is independent and needs its own provider plugins downloaded into its own `.terraform/` folder.

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

**This is the declarative model in one screenshot.** You didn't say "add one more". You said "there should be 3". Terraform checked state, found 2, and worked out that one creation gets you there.

- Type: `yes`

Have a look in the console:

*(Run from `~/terraform-training/02-count`)*
```bash
terraform console
```
```
azuread_user.my_azuread_users
```
![intro-to-terraform-27](./resources/intro-to-terraform-27.png)

The reference now returns a **list** — because `count` produced many. So you index into it, and chain attributes off:

```
azuread_user.my_azuread_users[0].id
```

**ASK** <br>
What's that returning? <br>
**ANSWER** <br>
The **Object ID** of the first indexed Azure AD user. Unlike an AWS arn, it won't break down into readable segments — it's just a GUID uniquely identifying that object in your tenant. For something human-readable, use `user_principal_name` instead.

```
azuread_user.my_azuread_users[0].user_principal_name
```

Exit with `Ctrl + C`.

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>
### 15:15–16:45 — Capstone: Lists, Sets, `for_each` and Maps
*(Exercise — 1 hour 30 minutes)*

This is the main event. You're going to discover a genuine, well-known problem with `count`, fix it with `for_each`, and then build up to configurations rich enough to describe real infrastructure.

Work individually or in pairs. Work through Parts 1–4 in order — each one motivates the next. Part 5 is stretch.

---

#### Part 1 (≈20 min) — Driving resources from a list

Let's move away from `my_iam_user_0` and use real names.

*(Run from `~/terraform-training/02-count`)*
- Run: `terraform destroy` → type `yes` (tidy up before moving on)
- Run: `cd ..`

*(Run from `~/terraform-training`)*
- Run: `mkdir 03-lists-and-sets` → **cd inside**

*(Run from `~/terraform-training/03-lists-and-sets`)*
- Run: `touch main.tf`

**03-lists-and-sets/main.tf**
```tf
# NEW CODE
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
  user_principal_name = "${var.names[count.index]}@emilesherrottdevops.onmicrosoft.com"
  display_name        = var.names[count.index]
  mail_nickname       = var.names[count.index]
  password            = "ChangeMe123!ChangeMe"
}
```

New syntax:

- `default = ["emile", "romeo", "sarah"]` — a **list**, using square brackets. Same as a JavaScript array, indexed from 0
- `length(var.names)` — a **built-in function**. In JavaScript you'd write `names.length`; HCL uses function-call syntax instead: `length(thing)`. It evaluates to 3, so `count.index` runs 0, 1, 2
- `var.names[count.index]` — index into the list using the current iteration number

Notice `display_name = var.names[count.index]` has **no** `${ }` — the whole value *is* the expression, so no interpolation is needed. But `user_principal_name` does need it, because the value is embedded in a larger string alongside the `@domain` part.

*(Run from `~/terraform-training/03-lists-and-sets`)*
```bash
terraform init
terraform apply
```

*WHILST IT'S RUNNING, SAY:*

Worth explaining why we keep making new projects. It's **Terraform best practice** — a single application commonly has several separate Terraform projects, because different resources have different **lifecycles**.

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
![intro-to-terraform-45](./resources/intro-to-terraform-45.png)
![intro-to-terraform-46](./resources/intro-to-terraform-46.png)

While you're here, meet the **collection functions**. Collections are lists, sets, maps and tuples.

```
length(var.names)              # 3
reverse(var.names)             # reverses the order
distinct(var.names)            # removes duplicates
toset(var.names)               # converts to a set
concat(var.names, ["tom", "astha"])   # joins two lists
contains(var.names, "simon")   # true / false
sort(var.names)                # alphabetical
```

![intro-to-terraform-47](./resources/intro-to-terraform-47.png)
![intro-to-terraform-48](./resources/intro-to-terraform-48.png)
![intro-to-terraform-49](./resources/intro-to-terraform-49.png)

Note `distinct` returns its result wrapped as `tolist([...])` — Terraform is being explicit that it produced a list, since the operation could apply to several collection types.

And `range` for generating sequences:
```
range(10)          # 0 to 9
range(1, 12)       # 1 up to but not including 12
range(1, 12, 3)    # third argument is the step: 1, 4, 7, 10
```
![intro-to-terraform-54](./resources/intro-to-terraform-54.png)
![intro-to-terraform-55](./resources/intro-to-terraform-55.png)
![intro-to-terraform-56](./resources/intro-to-terraform-56.png)

Exit with `Ctrl + C`. Get into the habit of checking the [function documentation](https://developer.hashicorp.com/terraform/language/functions) — there are dozens more.

---

#### Part 2 (≈20 min) — The problem with lists

Here's the exercise that makes the whole afternoon click. Add a name to the **front** of the list:

**main.tf**
```tf
variable "names" {
  # UPDATED CONFIG
  default = ["simon", "emile", "romeo", "sarah"]
}
```

Before running anything — **predict what will happen.** We added one name to a list of three. What should Terraform do?

*(Run from `~/terraform-training/03-lists-and-sets`)*
```bash
terraform apply
```
![intro-to-terraform-57](./resources/intro-to-terraform-57.png)

**1 to add… but 3 to change.**

**ASK** <br>
We added exactly one name. Why is Terraform changing three existing users? <br>
**ANSWER** <br>
Because `count` tracks resources by **position**, not by identity. Everything shifted: `simon` now occupies index 0 where `emile` was, `emile` moved to 1, `romeo` to 2 — so Terraform sees each of those slots as having *changed value*, and index 3 as brand new. It has no concept that "emile" is the same person who was there before; it only knows "the resource at index 0 used to be emile and should now be simon".

Look in the state file to see exactly why:

**terraform.tfstate**
```tf
{
  "resources": [
    {
      "mode": "managed",
      "type": "azuread_user",
      "name": "my_azuread_users",
      "instances": [
        {
          # HERE
          "index_key": 0,
          "attributes": {
            "user_principal_name": "emile@emilesherrottdevops.onmicrosoft.com",
            "display_name": "emile"
          }
        },
        {
          # HERE
          "index_key": 1,
          "attributes": {
            "user_principal_name": "romeo@emilesherrottdevops.onmicrosoft.com",
            "display_name": "romeo"
          }
        },
        {
          # HERE
          "index_key": 2,
          "attributes": {
            "user_principal_name": "sarah@emilesherrottdevops.onmicrosoft.com",
            "display_name": "sarah"
          }
        }
      ]
    }
  ]
}
```

The `index_key` is a **number**. That number is the only thing joining your configuration to the real Azure object. Reorder the list and every mapping is wrong.

**ASK** <br>
On Azure AD users this is annoying. Where would this behaviour be genuinely dangerous? <br>
**ANSWER** <br>
Anywhere the resource holds state or takes time to build — databases, VMs with data on them, storage. Inserting one item at the top of a list could trigger the destruction and recreation of *every* resource after it. This is the single most common way people get burned by `count`, and it's exactly why `for_each` exists.

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
  # UPDATED CODE — commented out
  #   count               = length(var.names)
  #   user_principal_name = "${var.names[count.index]}@emilesherrottdevops.onmicrosoft.com"

  # NEW CONFIG
  for_each            = toset(var.names)
  user_principal_name = "${each.value}@emilesherrottdevops.onmicrosoft.com"
  display_name        = each.value
  mail_nickname       = each.value
  password            = "ChangeMe123!ChangeMe"
}
```

The changes:

- `for_each` replaces `count`. It's the other iteration meta-argument, and it works over a **set or a map** rather than a number
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
- **Hashing** — each element is run through a hash function, turning it into a numeric fingerprint
- **Storage** — that fingerprint is used as an address in a table, and the element is stored there
- **Lookup** — to check whether something exists, hash it and jump straight to that address

This gives extremely fast lookups, and the price is that ordering isn't retained.
**END SIDE TANGENT**

- Type: `yes`

Now look at the state file again:

**terraform.tfstate**
```tf
"instances": [
  {
    # HERE
    "index_key": "emile",
    "attributes": {
      "user_principal_name": "emile@emilesherrottdevops.onmicrosoft.com",
      "display_name": "emile"
    }
  },
  {
    # HERE
    "index_key": "romeo",
    "attributes": {
      "user_principal_name": "romeo@emilesherrottdevops.onmicrosoft.com",
      "display_name": "romeo"
    }
  },
  {
    # HERE
    "index_key": "simon",
    "attributes": {
      "user_principal_name": "simon@emilesherrottdevops.onmicrosoft.com",
      "display_name": "simon"
    }
  }
]
```

**`index_key` is now a string — the value itself.** That's the whole fix. Each resource is tracked by *what it is*, not *where it sat*.

Prove it. Add a name at the front:

**main.tf**
```tf
variable "names" {
  # UPDATED CONFIG
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
  # UPDATED CONFIG
  default = ["emile", "sarah"]
}
```

*(Run from `~/terraform-training/03-lists-and-sets`)*
```bash
terraform apply
```
![intro-to-terraform-60](./resources/intro-to-terraform-60.png)

It identifies precisely which users to remove, and leaves the rest alone.

- Type: `yes`

**The rule to take away: prefer `for_each` over `count` whenever the things you're creating have a meaningful identity.** Reserve `count` for genuinely interchangeable resources, or for conditionally creating something (`count = var.enabled ? 1 : 0`).

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

Copy across the config from `03-lists-and-sets`, then convert the variable from a list to a **map**:

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

# UPDATED CONFIG — square brackets [ ] become curly braces { }
variable "users" {
  default = {
    emile : "England",
    sarah : "France"
  }
}

resource "azuread_user" "my_azuread_users" {
  for_each            = var.users
  user_principal_name = "${each.key}@emilesherrottdevops.onmicrosoft.com"
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

- **`[ ]` became `{ }`** — that's the syntactic difference between a list and a map
- A map is **key/value pairs**, like a JavaScript object. Here the key is the username and the value is their country
- `for_each = var.users` — **no `toset()` needed**. Map keys are already unique by definition
- `each.key` is now the username, `each.value` is the country. With a set they were the same; with a map they're properly distinct
- The variable is renamed from `names` to `users`, because it now holds more than names. Name things for what they contain

Notice the keys aren't quoted — HCL allows bare identifiers as map keys. The values are strings, so they are.

Explore it:

*(Run from `~/terraform-training/04-maps`)*
```bash
terraform console
```

```
var.users
```
![intro-to-terraform-61](./resources/intro-to-terraform-61.png)

**ASK** <br>
What will `var.users.sarah` return? <br>
**ANSWER** <br>
`"France"`

```
var.users.sarah            # dot notation
var.users["sarah"]         # bracket notation — identical result
keys(var.users)            # just the keys
values(var.users)          # just the values
lookup(var.users, "emile") # find a value by key
```
![intro-to-terraform-62](./resources/intro-to-terraform-62.png)
![intro-to-terraform-63](./resources/intro-to-terraform-63.png)
![intro-to-terraform-64](./resources/intro-to-terraform-64.png)
![intro-to-terraform-65](./resources/intro-to-terraform-65.png)
![intro-to-terraform-66](./resources/intro-to-terraform-66.png)

Both `.sarah` and `["sarah"]` work — exactly like JavaScript objects.

Exit with `Ctrl + C`, then apply:

*(Run from `~/terraform-training/04-maps`)*
```bash
terraform apply
```
![intro-to-terraform-67](./resources/intro-to-terraform-67.png)

- Type: `yes`

*(In the Azure Portal — Microsoft Entra ID → Users)* — both users exist, with countries set.

Check the state: `index_key` is the username, same as with a set. Adding and removing users behaves correctly.

**Now nest a map inside the map**, so each user can carry multiple attributes:

**main.tf**
```tf
variable "users" {
  default = {
    # UPDATED CONFIG
    emile : { country : "England" },
    sarah : { country : "France" }
  }
}
```

The key is the same, but the value is no longer a string — it's **another map**.

**ASK** <br>
The resource block currently says `country = each.value`. How do we reach the country now? <br>
**ANSWER** <br>
`each.value.country` — `each.value` is now an object, so you chain onto it, exactly as you would in JavaScript.

```tf
resource "azuread_user" "my_azuread_users" {
  for_each            = var.users
  user_principal_name = "${each.key}@emilesherrottdevops.onmicrosoft.com"
  display_name        = each.key
  mail_nickname       = each.key
  password            = "ChangeMe123!ChangeMe"
  # UPDATED CODE
  country             = each.value.country
}
```

*(Run from `~/terraform-training/04-maps`)*
```bash
terraform apply
```
![intro-to-terraform-68](./resources/intro-to-terraform-68.png)

**No changes** — the resulting values are identical, even though the route to them is different. Terraform compares *results*, not the expressions that produced them.

Now the payoff — add another attribute:

**main.tf**
```tf
variable "users" {
  default = {
    # UPDATED CONFIG
    emile : { country : "England", department : "Training" },
    sarah : { country : "France", department : "Training" }
  }
}

resource "azuread_user" "my_azuread_users" {
  for_each            = var.users
  user_principal_name = "${each.key}@emilesherrottdevops.onmicrosoft.com"
  display_name        = each.key
  mail_nickname       = each.key
  password            = "ChangeMe123!ChangeMe"
  country             = each.value.country
  # UPDATED CONFIG
  department          = each.value.department
}
```

Both `country` and `department` are **native, typed attributes** on `azuread_user` — first-class fields on the resource, not entries stuffed into a generic tags map.

*(Run from `~/terraform-training/04-maps`)*
```bash
terraform apply
```
![intro-to-terraform-69](./resources/intro-to-terraform-69.png)

An in-place update adding the attribute.

- Type: `yes`

---

#### Part 5 — Stretch goals

1. **A third user with different values.** Add someone with a different country and department, apply, and confirm only one resource is created.
2. **Optional attributes.** Give one user a `job_title` and the others none. (Hint: `lookup(each.value, "job_title", null)` returns a default when the key is absent.)
3. **An output over the map.** Produce an output listing every created user's `user_principal_name`. (Hint: `values(azuread_user.my_azuread_users)[*].user_principal_name` — `[*]` is the **splat operator**, which pulls one attribute from every element.)
4. **Drift detection.** In the Portal, manually change one user's department. Then run `terraform plan` and see Terraform detect that Actual State no longer matches Desired State. This is the ClickOps problem, caught automatically.

**Solution**

**Stretch 1 and 2** — a third user, with an optional attribute:
```tf
variable "users" {
  default = {
    emile : { country : "England", department : "Training", job_title : "Trainer" },
    sarah : { country : "France", department : "Training" },
    romeo : { country : "Italy", department : "Engineering" }
  }
}

resource "azuread_user" "my_azuread_users" {
  for_each            = var.users
  user_principal_name = "${each.key}@jbloggsdevops.onmicrosoft.com"
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

**Stretch 3** — the output:
```tf
output "all_user_principal_names" {
  # values() turns the map of resources into a list,
  # then [*] pulls one attribute from every element
  value = values(azuread_user.my_azuread_users)[*].user_principal_name
}
```

*(Run from `~/terraform-training/04-maps`)*
```bash
terraform apply     # -> 1 to add (romeo), 1 to change (emile gains a job_title)
# type: yes
terraform output all_user_principal_names
```

**Stretch 4** — drift detection:

*(In the Azure Portal — Microsoft Entra ID → Users → sarah)* — change **Department** to `Marketing` and save.

*(Run from `~/terraform-training/04-maps`)*
```bash
terraform plan
```

The plan reports **1 to change**, showing `department: "Marketing" -> "Training"` — Terraform refreshed the Actual State, found it no longer matches the Desired State in your code, and proposes putting it back. Run `terraform apply` and the manual change is reverted.

**This is the single strongest argument for Infrastructure as Code.** Your `.tf` files aren't just a record of what you built — they're the *authority*. Anything anyone does by hand gets detected and corrected. That's the ClickOps drift problem from the Azure Fundamentals session, solved.

<br>
<br>

### 16:45–17:00 — Wrap-up, Destroy, & Q&A

#### Clean up first

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

`terraform destroy` works exactly like `apply` in reverse: it refreshes state, identifies every resource Terraform manages, and deletes them **in reverse dependency order** — so the storage account goes before the resource group that contains it.

**ASK** <br>
Why is the fact that destroying is *this easy* actually a headline feature, not just a convenience? <br>
**ANSWER** <br>
Because it changes what you can afford to do. If tearing down is one command, you can spin up a full test environment for an afternoon and destroy it before you leave. Before cloud computing, an organisation had to buy office space, order server components and physically install them. Now the whole lifecycle — **create, use, destroy** — is a habit, and it's what keeps cloud spend under control.

#### Terraform FAQ — a quick review

Worth walking through HashiCorp's own [state documentation](https://developer.hashicorp.com/terraform/language/state/purpose), which sets out why state exists:

**1. Mapping to the real world** — connecting your `my_azuread_user` label to a real Azure Object ID. Early Terraform prototypes tried to do this with tags instead; it didn't work well.

**2. Metadata** — tracking dependencies between resources so things get created and destroyed in the right order. If an application needs a VM and a database, the database must exist before the app connects to it.

**3. Performance** — state acts as a cache. Querying every resource on every run is slow; that's what `-refresh=false` lets you skip.

**Why so many separate projects?** Because it's best practice. One application commonly has several Terraform projects, split by lifecycle: users change a few times a year, VMs might change daily, storage accounts persist for years. Separate projects mean separate state, separate blast radius and faster plans.

#### Where today sits

You started with the ClickOps complaint from the Azure session and ended with the answer. You can now describe infrastructure declaratively, you understand what the state file is for and what happens when it's missing, and you can create many resources from one block — knowing why `for_each` beats `count`.

**ASK** <br>
Think back to the Jenkins session. We built a pipeline that checked out code, ran tests, then built and pushed a Docker image — with credentials from the vault and a `post` block. What would we change to make that pipeline run **Terraform** instead? <br>
**ANSWER** <br>
Structurally, almost nothing. You'd store the four `ARM_*` values in Jenkins' credentials vault instead of a Docker Hub token. The `sh` steps would run `terraform init`, then `terraform plan -out=tfplan` in one stage, then — after a human approval gate — `terraform apply tfplan`. And that's precisely why we saved a plan to a file this morning: **the reviewed plan is the thing that gets applied**, so nothing can change between review and execution.

Where this goes next:
- **Today** — infrastructure as declarative, version-controlled code
- **Next** — remote backends (state in an Azure Storage Account so a team can share it safely), more complex resources like Virtual Machines and Load Balancers, and modules for reusable infrastructure
- **Kubernetes** — running and scaling the containers your pipeline builds, on infrastructure Terraform provisions
- **Integration** — a Git push flowing through test, build, `terraform plan`, review, `terraform apply`, and deploy

**Q&A** — take remaining questions.

<br>
<br>

### Exercise (take-home / reinforcement)

Before the next session, individually or in pairs:

1. From an **empty folder**, get to a working resource group and storage account from memory — no notes. Rebuilding from scratch is the fastest way to make it stick.
2. Add a `location` variable with a default of `uksouth`, use it on your resource group, then override it to `ukwest` on the command line and read the plan. Don't apply it.
3. Convert a `count`-based configuration to `for_each`, and write a short paragraph explaining to a colleague why you did.
4. Add a `blob container` (`azurerm_storage_container`) inside your storage account, referencing it properly so Terraform builds the dependency graph.
5. Use `terraform state list` and `terraform state show <address>` to inspect state without opening the JSON. Work out what those commands do.
6. **(Stretch)** Read about **remote backends** and write down, in your own words, what problem they solve that a local `terraform.tfstate` doesn't.
7. **(Stretch)** Research the `random_password` resource and rework your AD user so the password isn't hard-coded.

**Solution** *(for the guided ones — 2, 4, 5)*

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

*(Run from your project folder)*
```bash
terraform plan -var="location=ukwest"
```

The plan shows the resource group **must be replaced** — location is another attribute that can't be changed in place. A very good reason to read plans carefully.

**Take-home 4** — a blob container inside the storage account:
```tf
resource "azurerm_storage_container" "my_container" {
  name                  = "uploads"
  # The reference creates the dependency, so Terraform builds
  # the storage account FIRST, then the container inside it
  storage_account_name  = azurerm_storage_account.my_storage_account.name
  container_access_type = "private"
}
```

**Take-home 5** — inspecting state without opening the JSON:

*(Run from your project folder)*
```bash
terraform state list
# lists every resource address Terraform is managing, e.g.:
#   azurerm_resource_group.my_resource_group
#   azurerm_storage_account.my_storage_account

terraform state show azurerm_storage_account.my_storage_account
# prints every attribute of that one resource, formatted readably
```

`state list` gives you the addresses; `state show` gives you the detail for one. Together they're far safer and faster than opening `terraform.tfstate` in an editor — and they remove any temptation to edit it.

<br>
<br>

### Conclusion

- **Inform** students this marks the end of the Terraform & Infrastructure as Code session
- **Reinforce** the three states — Desired, Known, Actual — and that the state file is what maps their code to real cloud resources
- **Remind** everyone to check `az resource list -o table` before they close their laptop; anything still running costs money
- **Preview** the next session: **remote backends**, so state lives in an Azure Storage Account and a whole team can work safely against the same infrastructure, plus more complex resources like Virtual Machines and Load Balancers
- **Direct** students to the take-home exercises and to the [Terraform Registry](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs) — the `azurerm` provider docs are the reference they'll use daily

---

[Back](./README.md)

---
