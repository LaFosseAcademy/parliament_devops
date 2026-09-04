# Session 5 — Terraform: Resource Provisioning & Remote State (Part 2 of 2) — Trainer Script

Taking trainees from "I can create a storage account and a user in Terraform" to "I can provision a networked, load-balanced fleet of virtual machines, configure them on creation, and store my state safely in a shared remote backend". This is written as a script — read it out, adapt where it feels natural, but the intent is that you could deliver this cold from the page.

---

## 📦 STARTER CODE — put this in the repo before training

Everything here goes into **`/infrastructure-as-code-terraform/understanding-resource-provisioning-on-azure/starter-code`** before the session.

**Same principle as Part 1: ship the reference, withhold the learning.** But today has one long, fiddly block — the network security group and its rules — where transcription genuinely costs twenty minutes and teaches nothing. That goes to Slack when we reach it, not into the starter repo, so students still build it in sequence rather than finding it pre-written.

<br>

**`README.md`**
```markdown
# Terraform Part 2 — Starter Code

## What's here

- **commands.md** — Terraform, SSH and Azure CLI commands for today.

- **session5-notes.md** — you fill this in. Carries forward your
  values from Session 1 and Part 1.

- **.gitignore** — copy this to the root of your terraform folder
  before your first apply, exactly as in Part 1.

- **completed-code/** — finished .tf files for each folder.
  Catch-up only.

## Before we start

    export | grep ARM     # all four must be set
    az account show       # must be signed in
    terraform version     # must respond
    az resource list -o table   # should be EMPTY from last session
```

<br>

**`.gitignore`** *(same as Part 1 — students copy it in again for the new folder)*
```gitignore
# Terraform state — contains secrets IN PLAIN TEXT. Never commit.
*.tfstate
*.tfstate.*

# Downloaded providers — hundreds of MB, re-downloadable
**/.terraform/

# Variable files often hold environment-specific or sensitive values
*.tfvars
!example.tfvars

# SSH private keys — NEVER commit these
*.pem
*.key

crash.log
```

<br>

**`commands.md`**
```markdown
# Command Reference — Session 5

## Terraform (revision from Part 1)

    terraform init          Download providers for THIS folder
    terraform validate      Check syntax without calling Azure
    terraform plan          What WOULD change
    terraform apply         Make it happen
    terraform destroy       Delete everything this project manages
    terraform console       Interactive REPL
    terraform graph         Print the dependency graph (DOT format)

## New today

    terraform init -reconfigure     Re-init after changing the backend
    terraform output -raw NAME      Print one output with no quotes
    terraform state list            Every resource being managed

## SSH

    chmod 400 key.pem                       Lock down a private key (REQUIRED)
    ssh -i key.pem azureuser@<public-ip>    Connect to a VM
    ssh-keygen -y -f key.pem > key.pub      Derive the public key from a private one

## Azure CLI

    az account show --query id -o tsv       Subscription ID
    az resource list -o table               Everything you own
    az group list -o table                  Your resource groups
    curl ifconfig.me                        Your own public IP address

## Where things live on a Linux VM (from Session 2)

    /var/www/html      Web server content — Apache serves from here
    /etc               Configuration files
    /var/log           Logs
```

<br>

**`session5-notes.md`** *(students fill this in)*
```markdown
# Session 5 Notes

## Carried forward

| What | Value |
|---|---|
| Subscription ID | |
| Tenant domain | |
| My storage prefix | e.g. `stjbloggs` |
| My public IP (`curl ifconfig.me`) | needed for the SSH rule |

## SSH key

| What | Value |
|---|---|
| Key name | e.g. `default-vm-ssh` |
| Private key path | `~/azure/azure_ssh_keys/default-vm-ssh.pem` |
| Public key path | `~/azure/azure_ssh_keys/default-vm-ssh.pub` |
| Permissions checked? | `ls -l` should show `-r--------` |

## Remote backend (built this afternoon)

| What | Value |
|---|---|
| Backend resource group | |
| Backend storage account | |
| Container name | `tfstate` |
| My state key | e.g. `dev/session5/users/terraform.tfstate` |

## What a VM needs around it — in my own words

| Resource | Why it's needed |
|---|---|
| Resource group | |
| Virtual network | |
| Subnet | |
| Public IP | |
| Network interface (NIC) | |
| Network security group | |
```

<br>

**`completed-code/`** — finished `.tf` files for `05-virtual-machines`, `06-vms-with-lb` and `07-backend-state`. Catch-up only.

<br>

---

## 💬 SLACK SNIPPETS — queue these before you start

**Post reactively.** Today has the longest blocks of the course — the NSG rules and the load balancer's five linked resources are pure transcription, and a single mistyped `priority` costs a student their afternoon.

| # | When | What to post |
|---|---|---|
| 1 | 09:05 | The pre-flight check |
| 2 | 10:20 | VNet + subnets block |
| 3 | 10:35 | **NSG + both rules** — long, post it |
| 4 | 11:25 | SSH key permissions + `ssh-keygen` |
| 5 | 11:35 | Public IP + NIC + NSG association |
| 6 | 11:45 | The virtual machine block |
| 7 | 12:20 | `connection` + `provisioner` blocks |
| 8 | 15:20 | **The whole load balancer set** — five resources, post it |
| 9 | 15:50 | The `backend "azurerm"` block |
| 10 | 16:45 | The destroy sequence |

Each is marked **💬 SLACK** in the body below.

---

## Organisation

### Audience

Trainees who completed **Terraform Part 1**. They can write resource blocks, reference one resource from another, run `init`/`plan`/`apply`/`destroy`, use variables, and explain the three states. They know `count` versus `for_each` and why `for_each` usually wins.

They are experienced developers: **Node, Express, REST, MVC, SQL, Git, GitHub, Docker, frontend, unit and integration testing**. Two things they already know matter enormously today:

- **Docker networking.** They've written `docker compose` files with an app and a database, and they understand that the database is reachable *by the app* but not from outside unless a port is published. **That's a VNet.** Lean on it hard
- **Immutable containers.** They already know you don't SSH into a running container to patch it — you build a new image and replace it. Today applies that to servers, and the instinct is already there

They have **not** provisioned anything with networking attached, used a `provisioner`, used a `data` source, or moved state off their laptop.

**NOTE FOR TRAINERS** <br>
The thing that makes today feel harder than Part 1 isn't difficulty, it's **volume**. A VM needs six resources around it before it can exist, and students who were comfortable yesterday can feel swamped by 11:30. <br>
Counter it by naming the pattern early and repeatedly: **every one of those six resources is either a container, an address, or a rule.** And point out that Terraform's dependency graph means they don't have to hold the order in their heads — that's the tool's job. Let the graph do the worrying. <br>
**END OF NOTE**

### How this document is laid out

Terraform is directory-scoped, and today we work across several folders plus the Azure Portal. Every instruction block is labelled:

- *(Run from `~/terraform-training/05-virtual-machines`)* — a **terminal** command, in that exact folder
- *(In the Azure Portal — Virtual machines)* — a **browser** action, from that screen

The folders we build today, continuing Part 1's numbering:

| Folder | What it's for |
|---|---|
| `~/terraform-training/05-virtual-machines` | One VM, VNet, subnets, NSG, public IP, NIC, provisioner |
| `~/terraform-training/06-vms-with-lb` | Three VMs across zones, behind a load balancer |
| `~/terraform-training/07-backend-state` | Remote state — a `users` project and a `backend-state` project |
| `~/azure/azure_ssh_keys` | SSH private keys. **Never in the repo** |

**Every activity has a `**Solution**` block** immediately afterwards.

### Duration & schedule

Full day, **09:00–17:00**, with a one-hour lunch and two 15-minute breaks.

**Hands-on time today: ~3 hours 10 minutes** across eight activities, every one with a solution in this document.

| Time | Section | Activity time |
|---|---|---|
| 09:00–09:15 | Welcome & recap | |
| 09:15–10:00 | What a VM actually needs: touring the Portal wizard | |
| **10:00–10:15** | **Break** | |
| 10:15–11:15 | Building the network: VNet, subnets, NSG and rules | **20 min** |
| 11:15–12:15 | SSH keys, public IP, NIC, and your first VM | **25 min** |
| 12:15–13:00 | Provisioners: configuring the VM on creation | **15 min + challenge** |
| **13:00–14:00** | **Lunch** | |
| 14:00–14:30 | Immutable servers, `data` sources and `terraform graph` | **10 min** |
| 14:30–15:00 | Why local state doesn't scale | **15 min research** |
| **15:00–15:15** | **Break** | |
| 15:15–16:45 | Capstone: load-balanced VMs & a remote backend | **90 min** |
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
  - **/infrastructure-as-code-terraform/understanding-resource-provisioning-on-azure/starter-code**
- **Make sure**, before the session starts, every student has:
  - Their `~/terraform-training` folder and working `ARM_*` environment variables from Part 1
  - `terraform version` and `az account show` both responding
  - A **globally-unique storage prefix** they've been using
  - **Nothing left running** — have them run `az resource list -o table` first thing

**NOTE FOR TRAINERS — the three things that will eat your day** <br>
**(1) Environment variables.** The `ARM_*` exports don't survive a new terminal unless they went into `.zshrc`. First thing this morning: `export | grep ARM` for everyone. <br>
**(2) SSH key file permissions.** `chmod 400` is mandatory — SSH silently refuses keys other users could read, and the resulting error ("Permission denied (publickey)") looks like a Terraform problem rather than a file-permissions one. <br>
**(3) Provisioner timeouts.** `remote-exec` needs port 22 reachable **and** the VM fully booted. If an apply hangs for five minutes then fails, it's nearly always a missing SSH NSG rule or a missing public IP. Have them check those two before anything else. <br>
**END OF NOTE**

## Learning objectives

- **Explain** what a virtual machine needs around it before it can exist
- **Build** a VNet, subnets and a Network Security Group with explicit rules in Terraform
- **Understand** how Azure NSG rules work — priority, direction, access, and default outbound
- **Provision** a Linux VM with an SSH key and configure it on creation with a `remote-exec` provisioner
- **Explain** why immutable servers are the recommended IaC pattern
- **Use** a `data` source to look up information Terraform doesn't manage
- **Visualise** a dependency graph with `terraform graph`
- **Scale** a single VM into a load-balanced fleet using `for_each`
- **Migrate** state to a remote backend in Azure Blob Storage, and explain locking and key structure

<br>

## Sequence

### 09:00–09:15 — Welcome & Recap: Where We Got To

Morning. Quick recap, because today builds directly on Part 1.

So yesterday you learned the Terraform **mechanics**: resource blocks, references, the three states, `plan` and `apply`, variables, and iterating with `count` and `for_each`. But everything you created was, honestly, quite small — a resource group, a storage account, some Azure AD users. Things with no dependencies and nothing to connect to.

Today you provision **real infrastructure**: virtual machines that sit inside a network, behind security rules, reachable from the internet, and eventually behind a load balancer. And you'll hit the thing that makes this feel like real engineering — **a VM in Azure cannot exist on its own.** It needs a network, a subnet, a network interface, an IP address and security rules, all created in the right order.

That's where yesterday's dependency graph, (where resources can be built in the right order) stops being a nice idea and starts doing serious work for you.

Then this afternoon we'll try to fix the last big weakness in what you've built: **your state file is on your laptop.** Fine alone, useless in a team, and — critically — the one thing blocking you from running Terraform in a pipeline.

**Standing rule again: anything we create, we destroy before we leave.** Today's resources genuinely cost money. VMs and load balancers are not free-tier-forever the way a couple of AD users were.

**Everyone check three things now:**

*(Run from `~/terraform-training`)*
```bash
export | grep ARM            # your four Azure credentials
az account show              # signed in?
az resource list -o table    # anything left from last session?
```

If `export | grep ARM` comes back empty, re-export the four variables now — everything today depends on them.


```bash
export ARM_CLIENT_ID=<appId>
export ARM_CLIENT_SECRET='<password>'
export ARM_SUBSCRIPTION_ID=<your-subscription-id>
export ARM_TENANT_ID=<tenant>
```

**💬 SLACK — snippet 1**, post at 09:05:
```bash
cd ~/terraform-training

export | grep ARM            # all four must appear
az account show              # must be signed in
terraform version            # must respond
az resource list -o table    # should be empty
curl ifconfig.me             # your public IP — note it down, you'll want it later
```

<br>
<br>

### 09:15–10:00 — What a VM Actually Needs: A Tour of the Portal Wizard

Before writing any Terraform, let's walk the Portal wizard for creating a VM. Not to create one — to **inventory everything it asks for**, so we know what we'll have to build in code.

*(In the Azure Portal — search **Virtual machines**)*

When a company has a server in a data centre, it's a **physical server** — where applications and databases get deployed. A **virtual machine** is a slice of a physical machine in someone else's data centre, which you rent and reach over the internet. They're not really "in the cloud" — they're in physical buildings all over the world; you just don't have to think about the building.

We've been using `uksouth`. Today let's switch to **Sweden Central**. For some reasons on a free account it looks like we're blocked from using the vast majority of rejions and this is the closest.

Aside from that though..

**ASK** <br>
By having multiple regions, what do we improve? <br>
**ANSWER** <br>
**Availability** — a failure in one region doesn't take everything down. And **low latency** — users get served from somewhere physically close, so responses are faster and more consistent.

Keep a scratch note of the configuration we'll translate into Terraform.

*(Run from `~/terraform-training`)*
```bash
touch config.md
```

**config.md**
```md
Location - swedencentral
```

Now walk the wizard.

- Click *Create -> Virtual Machine*

#### Basics: project and instance details

Straightforward — a **name**, and a **resource group**. Exactly like the Azure AD users yesterday.

#### Image (Marketplace Image)

The **software** the machine boots with: Ubuntu Server, Windows Server, Red Hat, Debian. We'll choose **Ubuntu Server 24.04 LTS** — common and well-supported. (**LTS** = Long Term Support: security patches for years rather than months.)


Terms worth noting, because they show up in our Terraform:

- `SLIDE ACROSS`

**`urn`** — the unique identifier for a Marketplace image, it stands for **Uniform Resource Name: The format is `publisher:offer:sku:version`. Four colon-separated parts.

SKU stands for **Stock Keeping Unit**, it's an identifer. 

**ASK** <br>
`publisher:offer:sku:version` — four parts identifying exactly one image. How could we compare this to something we've seen in Docker. <br>
**ANSWER** <br>
A **Docker image reference** — `registry/namespace/image:tag`. Same job: unambiguously name one specific resource so anyone can pull the identical thing.

**Gen2 VM** — boots using **UEFI** rather than the older Generation 1 / BIOS path.

**NOTE FOR TRAINERS — optional detour if the room is engaged** <br>
**BIOS** — the **Basic Input/Output System**. Firmware on a motherboard chip doing the **power-on-self-test** (checking processor, memory and storage are alive) then **bootstrapping** — finding the bootloader, which loads the OS into memory. <br>
**UEFI** — the **Unified Extensible Firmware Interface**, the modern replacement. Graphical rather than text, **Secure Boot** (only trusted software loads during boot), faster boot. <br>
Both are firmware that initialises hardware and loads the OS. Azure's **Generation 2** VMs use UEFI; Generation 1 uses BIOS-style boot. <br>
**END OF NOTE**

**Architecture: x64** — 64-bit x86, the widest software and hardware support.

This is where Azure annoys me. In AWS we'd use a GUI to find an image we want. AWS calls it an AMI, Amazon Machine Imagine and it'll give us the ID we need right there and then. 

Azure is difficult and we have to manually try and find it. 

No need to run this yourself but:

```bash
az vm image list \
  --location swedencentral \
  --publisher Canonical \
  --offer ubuntu-24_04-lts \
  --sku server \
  --architecture x64 \
  --all \
  --output table
```

- Canonical is the company behind Ubuntu 
- the 'offer' is which specifc distribution we want
- the 'sku' is just the image varient, we want Ubuntu Server

It outputs lots of images we could use and the Uniform Resoruce Names. So I can grap one and add it to the notes. 

**config.md**
```md
Location - swedencentral
Marketplace Image (urn) - Canonical:ubuntu-24_04-lts:server:latest
```

#### Instance type (VM size)

Software chosen; now the **hardware**. vCPUs, memory, storage, network performance.

What matters for us: eligibility under the **Azure free account** — a **B1s** burstable VM, free for 12 months. ("Burstable" means it accumulates CPU credits while idle and spends them on short bursts — perfect for a demo web server, useless for sustained compute.)

If I click on *See all* we can see the different families of Virtual Machines available to us. 

I'm going to look into the **B family** as its the cheaper options.

There's information on the amount of virtual CPU's, RAM, Disk Data, Max InputOutput PerSecond and maybe most importantly, price. 

The cost is per month but I'm going to choose: B2als_v2
- 2 vCPUs, 4GB RAM

**config.md**
```md
Location - swedencentral
Marketplace Image (urn) - Canonical:ubuntu-24_04-lts:server:latest
VM size - B2als_v2
```



#### Networking — the part that matters most

- *Go to 'Networking' tab*

In a physical data centre, resources are protected by a firewall on the device — inspecting packets and allowing or blocking them per your security policy.

In the cloud we get the same thing with a **Virtual Network (VNet)** — your own private network. Inside it you create smaller **subnets** to hold resources.

- `SLIDE ACROSS`

Read the diagram outside-in:
- Inside a **region** (North Europe) sits a **VNet**
- The VNet spans **availability zones** — physically separated locations within that region, each with independent power and networking
- Inside those sit **subnets**, where resources actually live

**ASK** <br>
You've all written a `docker compose` file with an app container and a Postgres container. Which of those is reachable from the internet, and why? <br>
**ANSWER** <br>
Only whichever one you published a port for. **Postgres is reachable by the app**, over Docker's internal network, but not from outside unless you explicitly mapped a port. **A VNet is exactly that, at cloud scale** — a private network where things reach each other by default, and exposure to the outside is something you deliberately configure. 

**Private vs public subnets.** A VM in a **private** subnet can only be reached from inside the VNet — where you'd put a **database**, or servers handling sensitive data. A VM with a **public IP attached** can be reached from outside — where you'd put an **HTTP server**.



So we'll create **1** VNet and **3** subnets, one per availability zone.


**config.md**
```md
Location - swedencentral
Marketplace Image (urn) - Canonical:ubuntu-24_04-lts:server:latest
VM size - B2als_v2
VNet - vnet-emilesherrott-devops
```

#### Firewall (Network Security Group)

- `SLIDE ACROSS`

**Network Security Groups (NSGs)** under **NIC** which is a Network Interface Card control traffic to resources beyond just choosing a subnet. They're their own resource, and a subnet — or a specific network interface — gets associated with the rules you define.

**ASK**<br>
Which two ports do HTTP and HTTPS requests use? <br>
**ANSWER**
80 and 443

The scenario that makes it concrete:
- You want your VM in a **public** subnet so the internet can reach it
- But you want to block anything that isn't HTTP or HTTPS, on ports 80 and 443

So even though traffic reaches the subnet, the Network Security Group blocks what shouldn't reach the Virtual Machine. **An NSG is a set of firewall rules.**

**ASK** <br>
Does anyone remember what **SSH** traffic is? <br>
**ANSWER** <br>
Secure Shell — remotely log into and control a machine over an encrypted connection. Conventionally **port 22**. It's how we'll get onto the VM to configure it.

For a web server we'll allow **SSH (22)** and **HTTP (80)**.


#### Disks and Advanced

Default Managed Disk is fine; leave Advanced alone.

Advanced let's you do something called *"Bootstrapping"*, not the rubbish bootstrap you learnt on week 1 but bootstrapping is something you do as you start. I.e. *"You strap your boots on before going out"*. Technically if could be something like, installing Docker, Git, updating packages etc..

We're not going to do that. 

**The point of all that.** Look at everything we had to think about for **one** VM: regions, availability zones, virtual networks, subnets, network security groups.


That was maybe fifteen decisions, and we haven't created anything. What happens when you need this VM in three regions, for three environments? <br>

By hand it's fifteen decisions times nine, each an opportunity to click the wrong thing — and **no record of what you chose or why**. That's the ClickOps problem at full scale. In Terraform it's one configuration and a changed variable. This is exactly the case for automating it.

We won't launch from the browser. Let's build it in VS Code.

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>

### 10:15–11:15 — Building the Network: VNet, Subnets, NSG and Rules
*(Activity: 20 min)*

#### Setting up the project

*(Run from `~/terraform-training`)*
- Run: `mkdir 05-virtual-machines`
- Run: `mv config.md 05-virtual-machines/` — move our notes in
- Run: `cd 05-virtual-machines`

*(Run from `~/terraform-training/05-virtual-machines`)*
- Run: `touch main.tf`
- Then: `code main.tf`

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

resource "azurerm_resource_group" "vm_resource_group" {
  name     = "rg-vm-emilesherrott-devops"
  # UPDATED — new region today
  location = "swedencentral"
}
```

- Remember all resources in Azure are associated with a Resource Group which is why we started with that. 

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform init
```

#### Virtual Network and Subnets

Because Azure gives us no default VNet, our first job is building one.

**main.tf**
```tf
[ . . . ]

# NEW CODE
resource "azurerm_virtual_network" "vm_vnet" {
  name                = "vnet-emilesherrott-devops"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
}

resource "azurerm_subnet" "public_subnets" {
  for_each             = { "1" = "10.0.1.0/24", "2" = "10.0.2.0/24", "3" = "10.0.3.0/24" }
  name                 = "subnet-public-${each.key}"
  resource_group_name  = azurerm_resource_group.vm_resource_group.name
  virtual_network_name = azurerm_virtual_network.vm_vnet.name
  address_prefixes     = [each.value]
}
```

Two new things to unpack properly.

**`address_space = ["10.0.0.0/16"]`** — the range of private IP addresses this VNet can hand out. That `/16` is **CIDR notation**, which you met in Session 1. Quick refresher:

- An IPv4 address is four numbers, each 0–255, because each is stored in **8 bits** of memory
- The number after the slash says **how many bits are fixed** — the network part
- `/16` fixes the first two numbers (`10.0`), leaving two numbers free (each can be between 0 and 255) → about 65,000 addresses
- `/24` fixes the first three (`10.0.1`), leaving one free → 256 addresses

**Bigger number = smaller network.** So our VNet is `10.0.0.0/16`, and each subnet carves a `/24` slice inside it.

**The `for_each` over a map** — yesterday's material doing real work:

```tf
for_each = { "1" = "10.0.1.0/24", "2" = "10.0.2.0/24", "3" = "10.0.3.0/24" }
```

`each.key` is `"1"`, `"2"`, `"3"` (used in the names) and `each.value` is the CIDR range. Three subnets from one block, each tracked in state by its key rather than a positional index.

**ASK** <br>
Why write this as a `for_each` over a map rather than three separate `azurerm_subnet` blocks? <br>
**ANSWER** <br>
Less repetition, and one place to change the pattern — but the real reason is that **later resources can iterate over this same collection.** When we build three VMs this afternoon we'll write `for_each = azurerm_subnet.public_subnets` and get one VM per subnet automatically, with matching keys. Three separate blocks couldn't be iterated over. You've made a collection, not just three things.

#### Network Security Group

We're building an **HTTP server** — conventionally accessed on **port 80** via **TCP**.

TCP is like two people on the phone: **connection** (you dial, they answer), **conversation** (taking turns, data flowing back and forth), **closing the call**. HTTP is the *structure of what's said* — how browsers and servers phrase requests and responses. TCP is the connection; HTTP is the conversation over it.

We also want **SSH** on **port 22** to connect in and configure the machine.

And because this is a public web server, it must be reachable from anywhere — which means a **CIDR block** specifying which source IPs are allowed.

- `SLIDE ACROSS`

**`"0.0.0.0/0"`** — the **default route**, meaning "any address anywhere":
- `0.0.0.0` — any address
- `/0` — zero bits fixed, so *every* IP is included

Azure also accepts the wildcard `"*"` for the same thing.

Now the Network Security Group:

**main.tf**
```tf
[ . . . ]

# NEW CODE
resource "azurerm_network_security_group" "http_server_nsg" {
  name                = "http-server-nsg"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
}
```

Note this creates an **empty** NSG — a container for rules. In Azure each rule is its own resource, which is actually the cleaner pattern: rules can be added, changed and reviewed independently without touching the group.

**main.tf**
```tf
[ . . . ]

# NEW CONFIG
resource "azurerm_network_security_rule" "http_ingress" {
  name                        = "AllowHTTP"
  priority                    = 100
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "80"
  source_address_prefix       = "0.0.0.0/0"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.vm_resource_group.name
  network_security_group_name = azurerm_network_security_group.http_server_nsg.name
}
```

Azure NSG rules need a few things worth explaining one at a time:

- `SLIDE ACROSS`

| Attribute | Meaning |
|---|---|
| `priority` | **Lower number is evaluated first.** Range 100–4096. Once a rule matches, evaluation **stops** |
| `direction` | `Inbound` or `Outbound` |
| `access` | `Allow` or `Deny` — Azure NSGs can explicitly **deny**, not just allow |
| `protocol` | `Tcp`, `Udp`, `Icmp` or `*` |
| `source_port_range` | Which port the traffic *came from*. Almost always `*` |
| `destination_port_range` | Which port it's *arriving at* — the one you care about |
| `source_address_prefix` | Which IPs are allowed. `0.0.0.0/0` means anywhere |

**ASK** <br>
Why is `source_port_range` almost always `*` while `destination_port_range` is specific? <br>
**ANSWER** <br>
Because the **destination** port identifies the service — 80 is "the web server", 22 is "SSH". The **source** port however is an arbitrary high-numbered port the client's operating system picked for that connection, different every time. You can't predict it and have no reason to care. 


"Lower priority number wins, and evaluation stops at the first match." We've seen this before with **Express middleware and route matching** — first matching route handles the request, and order in the file determines precedence. Think about your debug assignment and the **/top snack endpoint**. 

Now SSH:

**main.tf**
```tf
[ . . . ]

# NEW CONFIG
resource "azurerm_network_security_rule" "ssh_ingress" {
  name                        = "AllowSSH"
  priority                    = 110
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "22"
  source_address_prefix       = "0.0.0.0/0"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.vm_resource_group.name
  network_security_group_name = azurerm_network_security_group.http_server_nsg.name
}
```

Note `priority = 110`. **Priorities must be unique within an NSG.** Leaving gaps (100, 110, 120) is a good habit.


**ASK** <br>
We've opened SSH to `0.0.0.0/0` — the entire internet: `source_address_prefix`. Is that wise? <br>
**ANSWER** <br>
**No**, not for anything real. Every SSH server exposed to the open internet gets continuously probed by automated bots trying default credentials — within minutes of it existing. In production you'd restrict `source_address_prefix` to your office or VPN range, or avoid public SSH entirely using Azure Bastion. We're doing it because a classroom on different networks is otherwise painful — but **it's a shortcut.**

Now apply:

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform validate
terraform apply
```


- Type: `yes`

*(In the Azure Portal — search **Network security groups**)* — click into `http-server-nsg`. Your inbound rules are there.

*Scroll down and note the default outbound rules*

#### Tags, and ForceNew

Let's add a tag as well. 

We could be in the situation where we have 100s of Virtual Machines, some for testing, development or production. 

Adding tags helps us target them a little more accurately. 

**main.tf**
```tf
resource "azurerm_network_security_group" "http_server_nsg" {
  [ . . . ]
  # NEW CONFIG
  tags = {
    name = "http-server-nsg"
  }
}
```

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform apply
# type: yes
```

Get into the habit of tagging — `environment`, `owner`, `who's paying for it`. Once you have hundreds of resources, untagged infrastructure becomes unmanageable and unattributable on the bill.

Now go and read the docs:

*GOOGLE: terraform azurerm network security group* <br>
https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/network_security_group

Scroll to **Argument Reference**. Notice some arguments are marked **`Changing this forces a new resource to be created`**.



**ASK** <br>
`name` is marked ForceNew. What happens if you rename an NSG that a running VM depends on? <br>
**ANSWER** <br>
Terraform will **destroy and recreate** it — and because the VM's association depends on it, that cascades. Same "must be replaced" behaviour as the storage account name in Part 1. The provider documentation tells you in advance which attributes are safe to change and which are destructive. Checking before editing something in production takes thirty seconds and occasionally saves your afternoon.

- `SLIDE ACROSS`

**HANDS ON (20 min)** <br>
*(Run from `~/terraform-training/05-virtual-machines`)*
1. Create the project, `main.tf` with the provider and a resource group in `northeurope`, and run `terraform init`
2. Add the VNet with address space `10.0.0.0/16`
3. Add the three subnets using `for_each` over a map
4. Add the NSG, plus inbound rules for HTTP (80) and SSH (22)
5. `terraform validate`, then `terraform apply`
6. *(In the Azure Portal)* Find your NSG, confirm both inbound rules, and **note the default outbound rules**
7. Add a `tags` block to the NSG and apply again
8. Open the `azurerm_network_security_rule` docs and find **two** attributes marked ForceNew
**END OF NOTE**

Equally feel free to make notes on what we've covered so far or also browse the Azure Portal GUI for configuration options. 

**💬 SLACK — snippet 2**, the VNet and subnets:
```tf
resource "azurerm_virtual_network" "vm_vnet" {
  name                = "vnet-CHANGEME-devops"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
}

resource "azurerm_subnet" "public_subnets" {
  for_each             = { "1" = "10.0.1.0/24", "2" = "10.0.2.0/24", "3" = "10.0.3.0/24" }
  name                 = "subnet-public-${each.key}"
  resource_group_name  = azurerm_resource_group.vm_resource_group.name
  virtual_network_name = azurerm_virtual_network.vm_vnet.name
  address_prefixes     = [each.value]
}
```

**💬 SLACK — snippet 3**, the NSG and both rules — **long, definitely post this**:
```tf
resource "azurerm_network_security_group" "http_server_nsg" {
  name                = "http-server-nsg"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name

  tags = {
    name = "http-server-nsg"
  }
}

resource "azurerm_network_security_rule" "http_ingress" {
  name                        = "AllowHTTP"
  priority                    = 100
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "80"
  source_address_prefix       = "0.0.0.0/0"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.vm_resource_group.name
  network_security_group_name = azurerm_network_security_group.http_server_nsg.name
}

resource "azurerm_network_security_rule" "ssh_ingress" {
  name                        = "AllowSSH"
  priority                    = 110
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "22"
  source_address_prefix       = "0.0.0.0/0"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.vm_resource_group.name
  network_security_group_name = azurerm_network_security_group.http_server_nsg.name
}
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
  }
}

provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "vm_resource_group" {
  name     = "rg-vm-jbloggs-devops"
  location = "northeurope"
}

resource "azurerm_virtual_network" "vm_vnet" {
  name                = "vnet-jbloggs-devops"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
}

resource "azurerm_subnet" "public_subnets" {
  for_each             = { "1" = "10.0.1.0/24", "2" = "10.0.2.0/24", "3" = "10.0.3.0/24" }
  name                 = "subnet-public-${each.key}"
  resource_group_name  = azurerm_resource_group.vm_resource_group.name
  virtual_network_name = azurerm_virtual_network.vm_vnet.name
  address_prefixes     = [each.value]
}

resource "azurerm_network_security_group" "http_server_nsg" {
  name                = "http-server-nsg"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name

  tags = {
    name = "http-server-nsg"
  }
}

resource "azurerm_network_security_rule" "http_ingress" {
  name                        = "AllowHTTP"
  priority                    = 100
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "80"
  source_address_prefix       = "0.0.0.0/0"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.vm_resource_group.name
  network_security_group_name = azurerm_network_security_group.http_server_nsg.name
}

resource "azurerm_network_security_rule" "ssh_ingress" {
  name                        = "AllowSSH"
  priority                    = 110
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "22"
  source_address_prefix       = "0.0.0.0/0"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.vm_resource_group.name
  network_security_group_name = azurerm_network_security_group.http_server_nsg.name
}
```

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform init
terraform validate
terraform apply
# type: yes
```

**Watch the creation order in the output**: resource group → VNet → subnets and NSG → NSG rules. **You never specified that order.** The references between resources built the dependency graph, exactly as in Part 1.

**Step 8 answer** — on `azurerm_network_security_rule`, ForceNew attributes include `name`, `resource_group_name` and `network_security_group_name`. Which makes sense: those three together *identify* the rule, so changing any of them means it's a different rule, not a modified one.

<br>
<br>

### 11:15–12:15 — SSH Keys, Public IP, NIC, and Your First VM
*(Activity: 25 min)*

#### Creating an SSH key

We've allowed SSH through the firewall — but that only opens the door. To connect we need an **SSH key pair**, and the public half must be attached to the VM **when it's first created**. So the key comes first.

**How key pairs work.** You generate two mathematically linked files. The **public key** can be shared freely and gets installed on the server. The **private key** stays on your machine and proves you're you. The server encrypts a challenge with the public key; only the matching private key can answer it. **You never send your private key anywhere.**

*(In the Azure Portal — search **SSH keys**)*


Creating one asks for:
- **name**
- **key pair type**, at the bottom you'll see:
  - **RSA** — the long-established public-key algorithm, the standard for many cryptographic operations
  - **Ed25519** — newer, faster, and in some respects stronger
- **key pair source**: **Generate new key pair** (Azure creates both halves), or **Use existing public key** (you've already run `ssh-keygen` locally)

Create one with the **resource group** `rg-vm-emilesherrott-devops`, **region** `Sweden Central`, **name** `default-vm-ssh`, **type** `RSA`, **source** `Generate new key pair`.


Hit **Create**. Your browser should give you the option to download `default-vm-ssh.pem` — the **private** key.


**This file is how anyone gets full access to your VM.** Like your Terraform state, it must never go near GitHub. That's why today's `.gitignore` includes `*.pem`.

Now set its permissions and store it sensibly:

I like to create a folder at the route of my user called `azure` and another directory called `azure-ssh-keys`. I'm going to move mine there.

*(Run from `~/Downloads` — wherever it landed)*
```bash
chmod 400 default-vm-ssh.pem
ls -l default-vm-ssh.pem
```

Recall `chmod` from Session 2. The three digits are three groups of people:
- First digit `4` — **you, the owner**: read only
- Second digit `0` — **your group**: nothing
- Third digit `0` — **everyone else**: nothing

`ls -l` should now show `-r--------`.

**ASK** <br>
SSH insists on `400` rather than something more relaxed like `644`, why do you think? <br>
**ANSWER** <br>
`644` would let any other user on the machine **read** your private key — completely defeating the point of it being private. SSH checks this and flatly refuses to use a loosely-permissioned key. It's a rare and welcome case of a tool protecting you from yourself, and it's why the error message is so confusing when it happens: SSH says "Permission denied (publickey)", which sounds like the *server* rejected you, when actually your *own client* refused to offer the key.

Store it consistently:

*(Run from `~/azure/azure_ssh_keys`)*
```bash
mkdir -p ~/azure/azure_ssh_keys
mv ~/Downloads/default-vm-ssh.pem ~/azure/azure_ssh_keys/
```

You'll also need the **public** half for Terraform, so we can provide it to the Virtual Machines when they're created. Azure only gave you the private key, so derive the public one:

*(Run from `~/azure/azure_ssh_keys`)*
```bash
ssh-keygen -y -f default-vm-ssh.pem > default-vm-ssh.pub
ls -la
```

`ssh-keygen -y` reads a private key (`-f`) and prints its matching public key, which we redirect into a `.pub` file with `>`.

This step is easily forgotten and blocks everything downstream — note it in `05-session-notes.md`.

#### Public IP and Network Interface

A VM in Azure can't just be created and attached to a subnet. It needs a **Network Interface (NIC)** — the virtual equivalent of a network card — and if we want it reachable from the internet, a **Public IP** resource attached to that NIC.

**main.tf**
```tf
[ . . . ]

# NEW CONFIG
resource "azurerm_public_ip" "http_server_pip" {
  name                = "pip-http-server"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

resource "azurerm_network_interface" "http_server_nic" {
  name                = "nic-http-server"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.public_subnets["1"].id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.http_server_pip.id
  }
}

resource "azurerm_network_interface_security_group_association" "http_server_nic_nsg" {
  network_interface_id      = azurerm_network_interface.http_server_nic.id
  network_security_group_id = azurerm_network_security_group.http_server_nsg.id
}
```

- `SLIDE ACROSS`

Three resources, each doing one job.

**`azurerm_public_ip`** — a routable internet address.
- `allocation_method = "Static"` — the address is reserved and doesn't change if the VM restarts. `Dynamic` gives you a new one each time, which is useless once anything points at it
- `sku = "Standard"` — the modern tier. Standard public IPs are **secure by default**: closed to inbound traffic unless an NSG explicitly allows it. That's why our NSG rules matter

**`azurerm_network_interface`** — the virtual network card connecting the VM to a subnet.
- `ip_configuration { }` — a nested block (no `=`, remember)
- `subnet_id = azurerm_subnet.public_subnets["1"].id` — **note the `["1"]`**. Because `public_subnets` was created with `for_each` over a map, referencing it requires a **key**. With `count` you'd write `[0]`; with `for_each` you write `["1"]`

**`azurerm_network_interface_security_group_association`** — a **join resource**, whose only job is linking two other resources. It has no properties of its own beyond the two IDs.

So: 
- `azurerm_public_ip` gives our server an internet address. 
- `azurerm_network_interface` connecects the server to the network


The Network Security Group is a seperate resource, rather than an attribute on the Network Interface because the relationship is **many-to-many** and has its own lifecycle. One NSG can be associated with many NICs and many subnets; making it a separate resource means you can attach and detach without modifying either side. If you've ever modelled a many-to-many relationship in SQL, you built a **join table** for exactly the same reason.

#### The Virtual Machine

**main.tf**
```tf
[ . . . ]

# NEW CONFIG
resource "azurerm_linux_virtual_machine" "http_server" {
  name                  = "http-server"
  resource_group_name   = azurerm_resource_group.vm_resource_group.name
  location              = azurerm_resource_group.vm_resource_group.location
  size                  = "B2als_v2"
  admin_username        = "azureuser"
  network_interface_ids = [azurerm_network_interface.http_server_nic.id]

  admin_ssh_key {
    username   = "azureuser"
    public_key = file(var.azure_ssh_public_key)
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "ubuntu-24_04-lts"
    sku       = "server"
    version   = "latest"
  }
}

variable "azure_ssh_public_key" {
  default = "~/azure/azure_ssh_keys/default-vm-ssh.pub"
}
```

Working through it:

**`size = "Standard_B1s"`** — the hardware, straight from `config.md`.

**`admin_username = "azureuser"`** — **we choose this ourselves.** Worth flagging: in AWS the login (`ec2-user`, `ubuntu`) is baked into the AMI and you have to know it. In Azure you pick it, so there's nothing to look up.

**`network_interface_ids = [ ... ]`** — note the **square brackets**: a VM can have multiple NICs, so this is a list even with one.

**`admin_ssh_key { }`** — installs the public key for that user. `file(...)` is a **built-in function** reading a file from disk and returning its contents as a string. Terraform reads your `.pub` at apply time and sends the contents to Azure.

**ASK** <br>
`file()` reads from your local disk at apply time. What does that imply for running this in a pipeline later? <br>
**ANSWER** <br>
That **the file has to exist on whatever machine runs Terraform.** Your laptop has it; a Jenkins agent starting from a fresh container does not. So a pipeline would need the key injected as a credential, or — better — you'd avoid `file()` for secrets entirely and pass the public key as a variable. It's a small thing that quietly breaks the transition from "works on my machine" to "works in CI", which is exactly the class of problem Session 8 is about.

**`os_disk { }`** — the managed disk. `caching = "ReadWrite"` is a performance setting; `storage_account_type = "Standard_LRS"` is HDD-backed locally-redundant storage. Cheap, fine for training.

**`source_image_reference { }`** — the four parts of the `urn` from `config.md`, split into named attributes. `"latest"` is an alias Azure resolves at apply time.

Now apply:

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform validate
terraform apply
# type: yes
```

![understanding-resource-provisioning-28](./resources/understanding-resource-provisioning-28.png)

**HANDS ON (25 min)** <br>

Part A *(10 min)* — the SSH key.
1. *(In the Azure Portal)* Create an SSH key called `default-vm-ssh`, download the `.pem`
2. *(Run from `~/Downloads`)* `chmod 400` it, confirm with `ls -l`, and move it to `~/azure/azure_ssh_keys/`
3. *(Run from `~/azure/azure_ssh_keys`)* Derive the `.pub` with `ssh-keygen -y`

Part B *(15 min)* — the VM and its plumbing.
4. *(Run from `~/terraform-training/05-virtual-machines`)* Add the public IP, NIC and NSG association
5. Add the `azurerm_linux_virtual_machine` and the `azure_ssh_public_key` variable
6. `terraform apply`, and confirm the VM appears in the Portal
7. Get the VM's public IP **from the terminal, not the Portal**, then SSH into it. Run `hostname` and `exit`
**END OF NOTE**

**💬 SLACK — snippet 4**, the SSH key handling:
```bash
# In whichever folder the .pem downloaded to:
chmod 400 default-vm-ssh.pem
ls -l default-vm-ssh.pem          # must show -r--------

mkdir -p ~/azure/azure_ssh_keys
mv default-vm-ssh.pem ~/azure/azure_ssh_keys/
cd ~/azure/azure_ssh_keys

# Derive the PUBLIC key from the private one
ssh-keygen -y -f default-vm-ssh.pem > default-vm-ssh.pub
ls -la
```

**💬 SLACK — snippet 5**, public IP + NIC + association:
```tf
resource "azurerm_public_ip" "http_server_pip" {
  name                = "pip-http-server"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

resource "azurerm_network_interface" "http_server_nic" {
  name                = "nic-http-server"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.public_subnets["1"].id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.http_server_pip.id
  }
}

resource "azurerm_network_interface_security_group_association" "http_server_nic_nsg" {
  network_interface_id      = azurerm_network_interface.http_server_nic.id
  network_security_group_id = azurerm_network_security_group.http_server_nsg.id
}
```

**💬 SLACK — snippet 6**, the VM:
```tf
resource "azurerm_linux_virtual_machine" "http_server" {
  name                  = "http-server"
  resource_group_name   = azurerm_resource_group.vm_resource_group.name
  location              = azurerm_resource_group.vm_resource_group.location
  size                  = "Standard_B1s"
  admin_username        = "azureuser"
  network_interface_ids = [azurerm_network_interface.http_server_nic.id]

  admin_ssh_key {
    username   = "azureuser"
    public_key = file(var.azure_ssh_public_key)
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts-gen2"
    version   = "latest"
  }
}

variable "azure_ssh_public_key" {
  default = "~/azure/azure_ssh_keys/default-vm-ssh.pub"
}
```

**Solution**

Parts A and B as shown above. For **step 7**, get the IP from the CLI rather than clicking:

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform console
```
Inside the console:
```
azurerm_public_ip.http_server_pip.ip_address
```
`Ctrl + C` to exit, then:
```bash
ssh -i ~/azure/azure_ssh_keys/default-vm-ssh.pem azureuser@<the-ip>
# type 'yes' to accept the host fingerprint the first time
hostname
exit
```

**If SSH refuses, check in this order:**

| Symptom | Cause |
|---|---|
| `Permission denied (publickey)` | `.pem` permissions. `ls -l` must show `-r--------` |
| `Connection timed out` | The SSH NSG rule didn't apply, or there's no public IP on the NIC |
| `Could not resolve hostname` | You pasted the private IP (`10.0.x.x`) instead of the public one |
| `Host key verification failed` | You've reused an IP from a destroyed VM. `ssh-keygen -R <ip>` clears it |

That last one will happen to somebody today, and the error is alarming — it's a security warning about a possible man-in-the-middle attack. It's just that Azure recycled an IP address.

<br>
<br>

### 12:15–13:00 — Provisioners: Configuring the VM on Creation
*(Activity: 15 min + challenge)*

We have a running, reachable machine — with nothing on it. Let's turn it into an actual web server.

We said configuration management is Ansible's job, and that's still true — but Terraform has basic tooling for it, and seeing it makes clear where the boundary sits.

Two pieces are needed:
- a **`connection`** block — *how* to reach the machine
- a **`provisioner`** block — *what to run* once we're on it

#### The connection block

**main.tf**
```tf
resource "azurerm_linux_virtual_machine" "http_server" {
  [ . . . ]

  # NEW CONFIG
  connection {
    type        = "ssh"
    host        = azurerm_public_ip.http_server_pip.ip_address
    user        = "azureuser"
    private_key = file(var.azure_ssh_private_key)
  }
}

# NEW CONFIG
variable "azure_ssh_private_key" {
  default = "~/azure/azure_ssh_keys/default-vm-ssh.pem"
}
```

- **`type = "ssh"`** — the connection protocol
- **`host`** — note we reference the **separate `azurerm_public_ip` resource's `ip_address`**, because unlike an AWS `aws_instance`, an Azure VM doesn't carry its own public IP as an attribute. The IP is its own resource
- **`user = "azureuser"`** — the `admin_username` we chose
- **`private_key = file(...)`** — the **private** half this time, because we're authenticating *as* the client

#### The provisioner block

**main.tf**
```tf
resource "azurerm_linux_virtual_machine" "http_server" {
  [ . . . ]
  connection { [ . . . ] }

  # NEW CONFIG
  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update -y",
      "sudo apt-get install apache2 -y",
      "sudo systemctl start apache2",
      "echo Welcome - Virtual Server is at ${azurerm_public_ip.http_server_pip.ip_address} | sudo tee /var/www/html/index.html"
    ]
  }
}
```

`remote-exec` runs commands **on the remote machine** over the connection we defined. `inline` takes a list, run in order.

Walk through each command — all Session 2 material:

**`sudo apt-get update -y`** — refresh the package index. `sudo` runs as root; `apt-get` is the Debian/Ubuntu package manager; **`-y` auto-answers "yes"**. Essential in automation — there's no human to press y, exactly the trap from Session 2.

**`sudo apt-get install apache2 -y`** — install the Apache HTTP server.

**`sudo systemctl start apache2`** — start the service.

**NOTE FOR TRAINERS** <br>
On Ubuntu the `apache2` package starts the service automatically on install, so this line is usually a no-op. **Keep it anyway** — being explicit makes the script portable (Amazon Linux does *not* auto-start `httpd`) and keeps the "install, then start" structure legible. <br>
**END OF NOTE**

**`echo ... | sudo tee /var/www/html/index.html`** — write our page. `echo` outputs the text, `|` pipes it into the next command, `sudo tee <file>` reads standard input and writes it to a file.

**ASK** <br>
Why `| sudo tee file` rather than the simpler `sudo echo ... > file`? <br>
**ANSWER** <br>
Because of **when the redirection happens**. In `sudo echo x > file`, the shell sets up the `>` redirection **first, as your normal user** — before `sudo` runs anything — so it fails with permission denied on a root-owned directory. Piping into `sudo tee` means the **writing program itself** runs as root. This trips up a lot of people and is genuinely useful to know.

`/var/www/html` is Apache's default web root — the `/var` folder from Session 2, holding variable data that changes as the system runs.

Note the `${azurerm_public_ip.http_server_pip.ip_address}` interpolation inside the command — Terraform substitutes the real IP before sending it.

Now apply:

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform validate
terraform apply
```

![understanding-resource-provisioning-30](./resources/understanding-resource-provisioning-30.png)

**Nothing happens.** No changes planned.

**ASK** <br>
We just added a whole provisioner and Terraform says there's nothing to do. Why? <br>
**ANSWER** <br>
Because **provisioners only run at creation time.** They aren't part of a resource's state — Terraform can't inspect a running VM and ask "have these commands been run?". So adding or changing a provisioner doesn't register as a change. **The only way to run it is to create the resource fresh.**

So:

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform destroy
# type: yes
terraform apply
# type: yes
```

Watch the **destroy order**:

![understanding-resource-provisioning-31](./resources/understanding-resource-provisioning-31.png)

1. **azurerm_linux_virtual_machine** — first
2. **azurerm_network_interface_security_group_association** — the NIC/NSG link
3. **azurerm_network_interface** and **azurerm_public_ip**
4. **azurerm_network_security_rule** and **azurerm_network_security_group** — last

**Exactly the reverse of creation order.** Terraform won't delete an NSG while a VM still depends on it — the dependency graph enforces safety in both directions.

Then on creation: VNet, subnets, NSG and rules → public IP and NIC → VM. Then it connects via SSH and runs your commands.

*(In your browser)* — paste the public IP into the address bar.

![understanding-resource-provisioning-33.png](./resources/understanding-resource-provisioning-33.png)

**What just happened, end to end:**
1. Your browser routes to that IP, opens a **TCP** connection, and sends an **HTTP GET** — by default to **port 80**
2. The NSG rule allows port 80 inbound, so it reaches the VM
3. Apache, installed by `remote-exec`, is listening and serves `/var/www/html/index.html`
4. Your message comes back

**You've automated the deployment of a working server in Azure.**

**HANDS ON (15 min)** <br>
*(Run from `~/terraform-training/05-virtual-machines`)*
1. Add the `connection` block and the `azure_ssh_private_key` variable
2. Add the `remote-exec` provisioner with all four commands
3. `terraform apply` — observe that **nothing changes**, and explain why
4. `terraform destroy` then `terraform apply`. **Watch both orders** and note them in `session5-notes.md`
5. Visit the public IP in a browser and confirm your message
**END OF NOTE**

**💬 SLACK — snippet 7**:
```tf
  connection {
    type        = "ssh"
    host        = azurerm_public_ip.http_server_pip.ip_address
    user        = "azureuser"
    private_key = file(var.azure_ssh_private_key)
  }

  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update -y",
      "sudo apt-get install apache2 -y",
      "sudo systemctl start apache2",
      "echo Welcome - Virtual Server is at ${azurerm_public_ip.http_server_pip.ip_address} | sudo tee /var/www/html/index.html"
    ]
  }
}

variable "azure_ssh_private_key" {
  default = "~/azure/azure_ssh_keys/default-vm-ssh.pem"
}
```

**Solution**

```tf
resource "azurerm_linux_virtual_machine" "http_server" {
  name                  = "http-server"
  resource_group_name   = azurerm_resource_group.vm_resource_group.name
  location              = azurerm_resource_group.vm_resource_group.location
  size                  = "Standard_B1s"
  admin_username        = "azureuser"
  network_interface_ids = [azurerm_network_interface.http_server_nic.id]

  admin_ssh_key {
    username   = "azureuser"
    public_key = file(var.azure_ssh_public_key)
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts-gen2"
    version   = "latest"
  }

  connection {
    type        = "ssh"
    host        = azurerm_public_ip.http_server_pip.ip_address
    user        = "azureuser"
    private_key = file(var.azure_ssh_private_key)
  }

  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update -y",
      "sudo apt-get install apache2 -y",
      "sudo systemctl start apache2",
      "echo Welcome - Virtual Server is at ${azurerm_public_ip.http_server_pip.ip_address} | sudo tee /var/www/html/index.html"
    ]
  }
}

variable "azure_ssh_private_key" {
  default = "~/azure/azure_ssh_keys/default-vm-ssh.pem"
}

variable "azure_ssh_public_key" {
  default = "~/azure/azure_ssh_keys/default-vm-ssh.pub"
}
```

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform validate
terraform apply          # step 3: "No changes" — provisioners only run at creation
terraform destroy        # type: yes
terraform apply          # type: yes — NOW the provisioner runs
```

**Step 3 answer:** provisioners aren't tracked in state and only execute when a resource is created. Terraform has no way to know whether the commands have run on an existing machine, so changing them produces no diff. **This is a genuine limitation, not a bug** — and it's the strongest practical argument for the immutable-server pattern we cover after lunch.

---

**Challenge**

*Direct* students, **in pairs**, to extend the provisioner so the served page:

* Displays the **VM's name** as well as its IP address
* Includes the **date and time** it was provisioned
* Is valid **HTML** — an `<h1>` and a `<p>`, not one line of plain text
* **OPTIONAL** — also writes `/var/www/html/health.html` containing just the word `OK` *(we'll want exactly this for the load balancer's health probe this afternoon)*

*Provide* this example of what should appear at `http://<your-ip>`:

```
Welcome to http-server

This server is at 20.31.44.12 and was provisioned on Tue 21 Oct 09:42:15 UTC 2025
```

*Grant* ~10 minutes.

Hints if stuck: `inline` is just a list of shell commands, so add as many as you like. A command running *on the VM* can use the VM's own shell — `$(date)` will be evaluated **there**, not by Terraform. And beware: **`${...}` is Terraform's interpolation, `$(...)` is the shell's** — they don't collide.

**SOLUTION**

```tf
provisioner "remote-exec" {
  inline = [
    "sudo apt-get update -y",
    "sudo apt-get install apache2 -y",
    "sudo systemctl start apache2",

    # Terraform substitutes ${...} before sending the command.
    # The VM's own shell evaluates $(date) when it runs.
    "echo '<h1>Welcome to http-server</h1>' | sudo tee /var/www/html/index.html",
    "echo '<p>This server is at ${azurerm_public_ip.http_server_pip.ip_address} and was provisioned on '$(date)'</p>' | sudo tee -a /var/www/html/index.html",

    # OPTIONAL: a health endpoint for the load balancer probe
    "echo OK | sudo tee /var/www/html/health.html"
  ]
}
```

Three points to draw out:

- **`tee -a` on the second line.** `tee` overwrites by default; `-a` **appends**. Without it, the paragraph wipes out the heading — the exact `>` versus `>>` lesson from Session 2, in a different disguise
- **Two different `$` syntaxes side by side.** `${azurerm_public_ip...}` is resolved by **Terraform, on your laptop, before the command is ever sent**. `$(date)` is resolved by **bash on the VM, at the moment it runs**. Getting this backwards is a very common error, and it's the same distinction as quoted-versus-unquoted heredocs from Session 2
- **Quoting.** The single quotes around the HTML stop bash interpreting the tags; we drop out of them around `$(date)` so the shell *does* evaluate it

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform destroy       # provisioners only run on create — so recreate
terraform apply
# type: yes
```

Then visit `http://<your-ip>` and `http://<your-ip>/health.html`.

<br>
<br>

*(Take a 1 hour lunch break here.)*

<br>
<br>
### 14:00–14:30 — Immutable Servers, `data` Sources and `terraform graph`
*(Activity: 10 min)*

#### Why immutability is the point, not a limitation

Before lunch you couldn't change the HTML with `terraform apply` — you had to destroy and recreate. That felt like a limitation. **It's actually the recommended pattern**, and worth understanding why.

*REFER TO RESOURCE 5 - SLIDEE* <br>
![understanding-resource-provisioning-34](./resources/understanding-resource-provisioning-34.png)

**The mutable approach.** Suppose servers could be modified in place. Every change means running a script against a live machine. Over months you accumulate a history of scripts, applied in some order, to some machines.

**ASK** <br>
Six months in, you need a brand new server identical to the existing ones. What do you have to do? <br>
**ANSWER** <br>
Re-run **every script, in exactly the original order** — and hope none were applied by hand, out of order, or only to some machines. In practice nobody can reconstruct that reliably, which is how **"snowflake servers"** happen: machines nobody dares touch because nobody knows how they got that way. It's environment drift from Session 1, at the level of individual servers.

*REFER TO RESOURCE 6 - SLIDEE* <br>
![understanding-resource-provisioning-35](./resources/understanding-resource-provisioning-35.png)

**The immutable approach.** A server is never modified. Want a change? Provision a **new** server from the updated configuration and destroy the old one. Every machine is built once, from a single declarative description, and is therefore identical to every other machine built from it.

**ASK** <br>
You already do this daily with something else. What? <br>
**ANSWER** <br>
**Docker containers.** You never SSH into a running container to patch it — you change the Dockerfile, build a new image, and replace the container. **You already have the instinct**; today just applies it one layer down, to the servers themselves. And notice the parallel goes further: a Dockerfile is a declarative description of a machine's contents, built once, producing identical results. Terraform's `remote-exec` is the *imperative* version of the same idea, which is precisely why HashiCorp calls provisioners a last resort.

**ASK** <br>
What does this pattern do for your ability to roll back a bad change? <br>
**ANSWER** <br>
Makes it trivial and **reliable**. The previous configuration is in Git; revert the commit and apply, and you get exactly the previous state — **not "the previous state plus whatever hand-fixes accumulated"**. Same as reverting a commit and redeploying, rather than trying to un-patch a live server.

#### Data sources

We hard-coded `version = "latest"` in the image reference. That works — Azure resolves the alias at apply time. But **you can't see which version you got**, which matters for auditing and reproducibility.

A **`data` source** looks up information Terraform **doesn't manage**. A `resource` block says "make this exist". A `data` block says "go and find out about something that already exists".

*(Run from `~/terraform-training/05-virtual-machines`)*

**main.tf**
```tf
[ . . . ]

# NEW CONFIG
data "azurerm_platform_image" "ubuntu_latest" {
  location  = azurerm_resource_group.vm_resource_group.location
  publisher = "Canonical"
  offer     = "0001-com-ubuntu-server-jammy"
  sku       = "22_04-lts-gen2"
}
```

- `data` instead of `resource` — read-only lookup
- `location` — image catalogues vary slightly by region
- `publisher / offer / sku` — the same identifiers, **without** the version. That's what we're asking it to find

References get a `data.` prefix: `data.azurerm_platform_image.ubuntu_latest.version`.

**ASK** <br>
`resource` creates and owns; `data` only reads. What's the practical difference when you run `terraform destroy`? <br>
**ANSWER** <br>
**Data sources are never destroyed**, because Terraform doesn't own them — it just looked them up. That's the whole distinction: `resource` means "I am responsible for this thing's existence"; `data` means "this exists independently and I need to know about it". Getting that boundary right is how you safely reference infrastructure another team owns without accidentally managing — or deleting — it.

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform apply
terraform console
```
```
data.azurerm_platform_image.ubuntu_latest
```
![understanding-resource-provisioning-46](./resources/understanding-resource-provisioning-46.png)

There's a resolved **`version`** — a real build number, not the string `"latest"`. Use it:

**main.tf**
```tf
source_image_reference {
  publisher = "Canonical"
  offer     = "0001-com-ubuntu-server-jammy"
  sku       = "22_04-lts-gen2"
  # NEW CONFIG
  version   = data.azurerm_platform_image.ubuntu_latest.version
}
```

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform apply
```

May or may not show a change, depending on whether a newer build shipped since you last applied.

**NOTE FOR TRAINERS** <br>
Good moment to compare with the AWS material. There, students used `data "aws_default_vpc"` to *adopt* an already-existing default VPC and `data "aws_subnets"` to find its subnets — necessary because AWS provisions those automatically in every region. <br>
Here we needed neither, because we **created our own** VNet and subnets. `azurerm_subnet.public_subnets["1"].id` was already dynamic; there was never a hardcoded value to remove. **Azure having no "default network" to adopt means less need for this flavour of data source.** Worth a minute — it's a real platform difference, not an omission. <br>
**END OF NOTE**

#### terraform graph

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform graph
```
![understanding-resource-provisioning-51](./resources/understanding-resource-provisioning-51.png)

That output is a **digraph** in the DOT language — a description of every resource and what depends on what.

*(In your browser)* — search "graphviz online", open [Graphviz Online](https://dreampuf.github.io/GraphvizOnline/), paste it in.

Arrows show dependency. `azurerm_linux_virtual_machine.http_server` depends on the data source, the NIC, and through that the subnet, VNet, public IP and NSG.

![understanding-resource-provisioning-52](./resources/understanding-resource-provisioning-52.png)

**This graph is exactly what Terraform builds internally** to decide creation and destruction order, and what can safely run in parallel.

**ASK** <br>
You've all seen a dependency graph before. Where? <br>
**ANSWER** <br>
**`npm ls`**, or the dependency graph on a GitHub repo. Same structure, same purpose: work out what depends on what so you can install (or create) in a valid order, and detect cycles. It's genuinely useful for reading an unfamiliar Terraform codebase, in the same way `npm ls` helps you understand an unfamiliar project.

#### Refactoring into logical files

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
touch variables.tf data-providers.tf network-security-group.tf
```

**variables.tf**
```tf
variable "azure_ssh_private_key" {
  default = "~/azure/azure_ssh_keys/default-vm-ssh.pem"
}

variable "azure_ssh_public_key" {
  default = "~/azure/azure_ssh_keys/default-vm-ssh.pub"
}
```

**data-providers.tf**
```tf
data "azurerm_platform_image" "ubuntu_latest" {
  location  = azurerm_resource_group.vm_resource_group.location
  publisher = "Canonical"
  offer     = "0001-com-ubuntu-server-jammy"
  sku       = "22_04-lts-gen2"
}
```

**network-security-group.tf** — move the NSG and both rules across.

*REMOVE THE MOVED CONFIG FROM `main.tf`*

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform apply     # -> No changes. Just reorganised.
```

Remember from Part 1: Terraform reads **every `.tf` file in the directory** and concatenates them. Filenames are for humans.

**HANDS ON (10 min)** <br>
*(Run from `~/terraform-training/05-virtual-machines`)*
1. Add the `data "azurerm_platform_image"` block, apply, and inspect it in `terraform console`
2. Use its `version` in your `source_image_reference` and apply again
3. Run `terraform graph`, paste it into Graphviz Online, and **find the longest dependency chain** in your infrastructure
4. Split the config into `main.tf`, `variables.tf`, `data-providers.tf` and `network-security-group.tf`. Confirm `terraform apply` reports **no changes**
5. `terraform destroy` — we're done with this project
**END OF NOTE**

**Solution**

Steps 1, 2 and 4 as shown above.

**Step 3 answer** — the longest chain is:

```
resource group -> vnet -> subnet -> nic -> vm
                     \-> nsg -> nsg rule -> (nic association) -> vm
```

Five deep. Point out that **Terraform worked all of that out from your references alone** — you never wrote a line of ordering logic. And it does the same in reverse for destroy.

**Step 5:**
```bash
terraform destroy
# type: yes
```

<br>
<br>

### 14:30–15:00 — Why Local State Doesn't Scale
*(Research activity: 15 min)*

One weakness remains in everything you've built: **your `terraform.tfstate` is sitting on your laptop.**

**ASK** <br>
Two engineers are working on the same infrastructure. Each has their own copy of the repo. Where's the state file, and what goes wrong? <br>
**ANSWER** <br>
Each has their **own local state file**, so each has a different idea of what exists. Engineer A creates a VM; A's state knows about it, B's doesn't. B runs `apply` and Terraform — seeing no record — tries to create it again. You get duplicates, or errors, or worse: **B's apply destroys something A created**, because B's state says it shouldn't exist.

**ASK** <br>
So we commit the state file to Git and share it that way? <br>
**ANSWER** <br>
**No** — two reasons. **Security**: state contains attribute values unencrypted, including secrets. You proved that yourself in Part 1 with `grep -i password terraform.tfstate`. **Reliability**: even ignoring security, you cannot guarantee everyone always adds, commits and pushes state at exactly the right moment. Someone forgets to push, someone else pulls a stale version, and you're back to conflicting views of reality — now with **merge conflicts in a machine-generated JSON file** nobody can resolve by hand.

**ASK** <br>
"A shared file that multiple people read and write, that must never be stale, that can't tolerate concurrent writes." What are you actually describing? <br>
**ANSWER** <br>
**A database.** And once you frame it that way, the solution is obvious: it needs to live in **one authoritative place**, with **locking** so two writers can't collide, **access control**, and ideally **versioning** so you can roll back a corrupted write. You wouldn't email a SQLite file around a team; the same reasoning applies here.

**The answer is a remote backend** — state stored centrally in cloud storage, which every engineer and every pipeline reads from and writes to.

#### Locking

Two engineers run `terraform apply` at the same moment. Both read state, both compute a plan from it, both start making changes. Neither knows about the other, and state ends up corrupted.

The fix is **locking**:
1. **Lock** the state before applying, so nobody else can start
2. Make the changes and update state
3. **Unlock** it

**NOTE FOR TRAINERS** <br>
Another good comparison with the AWS material. There, students had to stand up a whole extra resource — a **DynamoDB table** — purely to hold locks alongside the S3 bucket. <br>
Azure's `azurerm` backend needs no equivalent: locking is handled automatically with a **blob lease** on the state file itself, a native feature of Azure Blob Storage. **One fewer resource to create, pay for and maintain.** A genuine simplification worth calling out. <br>
**END OF NOTE**

An Azure Storage Account gives us everything:
- **Locking** — automatic, via blob lease
- **Encryption at rest** — on by default, no extra resource
- **Versioning** — turn on blob versioning and a corrupted state can be rolled back
- **Access control** — RBAC, so only the right identities can read it

**HANDS ON — research (15 min)** <br>
Before we build it, spend fifteen minutes reading and writing. **In pairs**, produce short written answers to:

1. **Backend options.** Open the [Terraform backends documentation](https://developer.hashicorp.com/terraform/language/settings/backends/configuration). List **three** backend types other than `azurerm`, and note one situation where you'd choose each
2. **Locking.** Find how the `azurerm` backend implements locking. Write one sentence explaining what a **blob lease** is
3. **The pipeline problem.** In your own words, in 2–3 sentences: *why can a Jenkins pipeline not use a local state file?* Be specific about what actually happens on the second build
4. **What if it's lost?** Research what `terraform import` does. If you lost your state file entirely but the resources still existed in Azure, how would you recover?
5. **(Stretch)** Read about **state locking failures**. What does `terraform force-unlock` do, and why is it dangerous?
**END OF NOTE**

**Solution**

**1 — Backend options.** From the docs:

| Backend | When you'd choose it |
|---|---|
| `azurerm` | Azure Blob Storage. You're on Azure — locking and encryption come free |
| `s3` | AWS. Very common, but needs a **DynamoDB table** for locking |
| `gcs` | Google Cloud Storage |
| `remote` / `cloud` | Terraform Cloud/Enterprise — adds a UI, run history, policy enforcement and team permissions |
| `http` | A generic REST backend. GitLab provides one built in |
| `local` | The default. Fine for solo experimentation, useless for teams |

**2 — Blob lease.** A **lease** is an exclusive lock Azure Blob Storage grants on a single blob for a period of time. While one client holds the lease, no other client can write to that blob. Terraform acquires a lease on the state blob before making changes and releases it afterwards — so a second `terraform apply` fails immediately with a clear message rather than corrupting anything.

**3 — The pipeline problem.** A Jenkins build runs in a **fresh, disposable workspace**. On the first build there is no state file, so Terraform believes nothing exists and plans to create everything — duplicating resources or failing on name collisions. Then the workspace is thrown away, so whatever state it wrote is lost, and the **second build repeats exactly the same mistake**. With a remote backend the pipeline reads the same authoritative state every time and takes a lock while it works.

**4 — `terraform import`.** It takes an existing real resource and **adds it to state**, mapping it to a resource block in your configuration:

```bash
terraform import azurerm_resource_group.my_rg /subscriptions/<sub-id>/resourceGroups/rg-jbloggs-devops
```

You must already have a matching `resource` block written; import only creates the state entry, not the configuration. Recovering a lost state file means importing **every resource one at a time**, which for a real environment is hours of tedious work. That's the honest answer to "what happens if you lose state", and it's why we're about to put it somewhere safe.

**5 — `force-unlock`.** It removes a lock by ID when one has been left behind — typically because a `terraform apply` was killed mid-run, or a pipeline job was cancelled while holding it. It's dangerous because **you might be removing a lock that another process is legitimately still using**, which is precisely the corruption the lock existed to prevent. The rule is: only use it when you're certain nothing else is running, and ideally after checking with whoever's name is on the lock.

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>
### 15:15–16:45 — Capstone: Load-Balanced VMs & a Remote Backend
*(Activity: 90 min)*

Two connected pieces. **Part A** scales your single VM into a load-balanced fleet of three, one per availability zone. **Part B** moves state off your laptop into Azure Blob Storage.

Work individually or in pairs. **Get Part A running and verified before starting Part B.**

**NOTE FOR TRAINERS** <br>
Watch the clock and the cost here. Part A is the more satisfying half — seeing traffic alternate between three servers is genuinely memorable — but **Part B is the more important one**, because it's the blocker for Session 8. <br>
If the room is running slow at 16:00, cut Part A's stretch work and make sure everyone completes Part B. A student who leaves without a working remote backend will be stuck in the integration session. <br>
**END OF NOTE**

---

#### Part A1 (≈15 min) — Set up the project and multiply the network resources

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
cd ..
```

*(Run from `~/terraform-training`)*
```bash
mkdir 06-vms-with-lb
cd 05-virtual-machines
```

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
cp data-providers.tf main.tf network-security-group.tf variables.tf ../06-vms-with-lb/
cd ../06-vms-with-lb
```

Rename the VM resource to a plural, because it'll represent many:

**06-vms-with-lb/main.tf**
```tf
# UPDATED — plural name to reflect many resources
resource "azurerm_linux_virtual_machine" "http_servers" {
  [ . . . ]
}
```

Each VM needs **its own NIC** and **its own public IP**, so those iterate too — over the subnets map we already have:

**main.tf**
```tf
# UPDATED CONFIG
resource "azurerm_public_ip" "http_server_pips" {
  for_each            = azurerm_subnet.public_subnets
  name                = "pip-http-server-${each.key}"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

resource "azurerm_network_interface" "http_server_nics" {
  for_each            = azurerm_subnet.public_subnets
  name                = "nic-http-server-${each.key}"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = each.value.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.http_server_pips[each.key].id
  }
}

resource "azurerm_network_interface_security_group_association" "http_server_nics_nsg" {
  for_each                  = azurerm_network_interface.http_server_nics
  network_interface_id      = each.value.id
  network_security_group_id = azurerm_network_security_group.http_server_nsg.id
}
```

**This is the pattern that makes `for_each` powerful**, so slow down here.

**`for_each = azurerm_subnet.public_subnets`** — we're iterating over **another resource** that was itself created with `for_each`. Because that resource is a map keyed by `"1"`, `"2"`, `"3"`, iterating it gives us the same keys:
- `each.key` is `"1"`, `"2"`, `"3"` — used in names
- `each.value` is the whole **subnet object**, so `each.value.id` is that subnet's ID

**`azurerm_public_ip.http_server_pips[each.key].id`** — reach into the *matching* public IP by key. Because every collection shares the same keys, everything lines up: **NIC `"2"` gets public IP `"2"` in subnet `"2"`.**

**ASK** <br>
Why is keying everything consistently like this so much safer than three sets of numbered resources? <br>
**ANSWER** <br>
Because **the key is the identity**. Remove subnet `"2"` from the map and Terraform removes exactly that subnet, its NIC, its public IP and its VM — leaving `"1"` and `"3"` untouched. With positional indexes everything after the removal would shift and get rebuilt. It's Part 1's list-versus-set lesson — the React `key` prop problem — now applied to real infrastructure where "rebuilt" means **real downtime**.

Add an output:

*(Run from `~/terraform-training/06-vms-with-lb`)*
```bash
touch outputs.tf
```

**outputs.tf**
```tf
# NEW CONFIG
output "http_server_public_ips" {
  value = { for k, pip in azurerm_public_ip.http_server_pips : k => pip.ip_address }
}
```

That's a **`for` expression** — HCL's comprehension syntax. Read it as: "for each key `k` and value `pip` in the collection, produce an entry mapping `k` to `pip.ip_address`". The `{ }` means we're building a map; `[ ]` would build a list.

**ASK** <br>
`{ for k, v in collection : k => v.something }`. What's the JavaScript? <br>
**ANSWER** <br>
`Object.fromEntries(Object.entries(obj).map(([k, v]) => [k, v.something]))` — or more readably, a `.map()` that rebuilds an object. HCL's version is arguably tidier. The square-bracket form `[for x in list : x.name]` is just `.map(x => x.name)`.

---

#### Part A2 (≈15 min) — Three VMs

**main.tf**
```tf
resource "azurerm_linux_virtual_machine" "http_servers" {
  # NEW CONFIG
  for_each              = azurerm_subnet.public_subnets
  name                  = "http-server-${each.key}"
  resource_group_name   = azurerm_resource_group.vm_resource_group.name
  location              = azurerm_resource_group.vm_resource_group.location
  size                  = "Standard_B1s"
  admin_username        = "azureuser"
  network_interface_ids = [azurerm_network_interface.http_server_nics[each.key].id]

  admin_ssh_key {
    username   = "azureuser"
    public_key = file(var.azure_ssh_public_key)
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts-gen2"
    version   = data.azurerm_platform_image.ubuntu_latest.version
  }

  tags = {
    name = "http-server-${each.key}"
  }

  connection {
    type        = "ssh"
    host        = azurerm_public_ip.http_server_pips[each.key].ip_address
    user        = "azureuser"
    private_key = file(var.azure_ssh_private_key)
  }

  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update -y",
      "sudo apt-get install apache2 -y",
      "sudo systemctl start apache2",
      "echo Welcome - Virtual Server ${each.key} is at ${azurerm_public_ip.http_server_pips[each.key].ip_address} | sudo tee /var/www/html/index.html"
    ]
  }
}
```

Note the message now includes `${each.key}` — **each server identifies itself.** That's how we'll prove the load balancer is actually distributing traffic.

*(Run from `~/terraform-training/06-vms-with-lb`)*
```bash
terraform init
terraform apply
# type: yes
```

Three HTTP servers. There's a lot of output — three VMs, each being SSH'd into and configured. Terraform does this **in parallel** where the graph allows, which is why the output interleaves.

**NOTE FOR TRAINERS** <br>
Watch the cost. A single B1s is free-tier for 12 months, but **three at once burns that allowance three times as fast**, and the Standard-SKU public IPs and load balancer are **chargeable from the first minute**. Keep runtime short and make sure everyone destroys. Good live example of the cost-consciousness point from Session 1. <br>
**END OF NOTE**

---

#### Part A3 (≈20 min) — The Load Balancer

A load balancer needs **its own NSG** — we don't want SSH open on the public entry point to our whole application.

**network-security-group.tf**
```tf
# HTTP_SERVER NSG ABOVE
[ . . . ]

# NEW CONFIG — LOAD BALANCER
resource "azurerm_network_security_group" "lb_nsg" {
  name                = "lb-nsg"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
}

resource "azurerm_network_security_rule" "lb_http_ingress" {
  name                        = "AllowHTTP"
  priority                    = 100
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "80"
  source_address_prefix       = "0.0.0.0/0"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.vm_resource_group.name
  network_security_group_name = azurerm_network_security_group.lb_nsg.name
}
```

**ASK** <br>
Why does the load balancer's NSG have an HTTP rule but **no SSH rule**? <br>
**ANSWER** <br>
**Least privilege.** The load balancer's job is to accept web traffic and pass it on — nobody should ever SSH into it. Every port you open is attack surface, so you open only what's needed. Same principle as scoping a Service Principal to one resource group rather than a whole subscription in Session 1, and the same principle as `Contributor` rather than `Owner` in Part 1.

**Unlike AWS's all-in-one Classic Load Balancer, Azure spreads this across several linked resources**: a frontend Public IP, the Load Balancer, a Backend Address Pool, a Health Probe, and a Load Balancing Rule.

The options, before we build:

| Option | For |
|---|---|
| **Basic SKU** | The oldest tier, comparable to a Classic ELB. Being retired |
| **Standard SKU** | Zone-redundant, higher scale, secure by default. **What we'll use** |
| **Application Gateway** | Azure's ALB equivalent. HTTP(S) content-based routing, path-based rules, WebSockets |
| **Azure Front Door** | Global routing and caching, worldwide content delivery |

**main.tf**
```tf
[ . . . ]

# NEW CONFIG
resource "azurerm_public_ip" "lb_pip" {
  name                = "pip-lb"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

resource "azurerm_lb" "lb" {
  name                = "lb"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
  sku                 = "Standard"

  # The public-facing side, tied to our public IP
  frontend_ip_configuration {
    name                 = "lb-frontend"
    public_ip_address_id = azurerm_public_ip.lb_pip.id
  }
}

resource "azurerm_lb_backend_address_pool" "lb_backend_pool" {
  loadbalancer_id = azurerm_lb.lb.id
  name            = "http-servers-pool"
}

resource "azurerm_network_interface_backend_address_pool_association" "http_server_nics_pool" {
  for_each                = azurerm_network_interface.http_server_nics
  network_interface_id    = each.value.id
  ip_configuration_name   = "internal"
  backend_address_pool_id = azurerm_lb_backend_address_pool.lb_backend_pool.id
}

resource "azurerm_lb_probe" "http_probe" {
  loadbalancer_id = azurerm_lb.lb.id
  name            = "http-probe"
  port            = 80
  protocol        = "Http"
  request_path    = "/"
}

resource "azurerm_lb_rule" "http_rule" {
  loadbalancer_id                = azurerm_lb.lb.id
  name                           = "http-rule"
  protocol                       = "Tcp"
  frontend_port                  = 80
  backend_port                   = 80
  frontend_ip_configuration_name = "lb-frontend"
  backend_address_pool_ids       = [azurerm_lb_backend_address_pool.lb_backend_pool.id]
  probe_id                       = azurerm_lb_probe.http_probe.id
}
```

Reading the pieces:

**`frontend_ip_configuration`** — the **front door**, the single address users hit. Note its `name` (`"lb-frontend"`); the rule refers back to it by that name.

**`azurerm_lb_backend_address_pool`** — where traffic goes. It starts **empty**.

**`azurerm_network_interface_backend_address_pool_association`** — another **join resource**, iterated, adding each NIC to the pool. `ip_configuration_name = "internal"` matches the name we gave inside the NIC — a NIC can have several IP configurations, so we say which one joins.

**`azurerm_lb_probe`** — the health check.

**ASK** <br>
Why does a load balancer need a health probe — why not just send traffic to all three servers? <br>
**ANSWER** <br>
Because a server can be **running** while being **broken** — Apache crashed, disk full, app wedged. The probe repeatedly requests `/`; if a server stops responding correctly it's **taken out of rotation automatically**, and put back when it recovers. Without it, the load balancer cheerfully sends a third of your users to a dead machine. **This is what turns three servers into genuine resilience rather than just three servers.**

**`azurerm_lb_rule`** — read it as a sentence: *traffic arriving on frontend port 80, at the frontend named `lb-frontend`, goes to backend port 80 on members of this pool, provided the probe says they're healthy.*

Note `frontend_port` and `backend_port` are separate — they don't have to match. A common real pattern is **443 in, 80 out**, terminating TLS at the load balancer.

**outputs.tf**
```tf
output "http_server_public_ips" {
  value = { for k, pip in azurerm_public_ip.http_server_pips : k => pip.ip_address }
}

# NEW CONFIG
output "lb_public_ip" {
  value = azurerm_public_ip.lb_pip.ip_address
}
```

*(Run from `~/terraform-training/06-vms-with-lb`)*
```bash
terraform apply
# type: yes
```

*(In your browser)* — visit the `lb_public_ip` value.

![understanding-resource-provisioning-58](./resources/understanding-resource-provisioning-58.png)

**NOTE FOR STUDENTS** <br>
Don't panic if the first request fails — the load balancer and its health probe take **a few minutes** to settle, and until the probe confirms a server is healthy it won't route to it. <br>
**END OF NOTE**

Keep refreshing. You'll see it alternate between servers 1, 2 and 3 — the message you wrote with `${each.key}` telling you which one you hit.

Then have a look at the graph:

*(Run from `~/terraform-training/06-vms-with-lb`)*
```bash
terraform graph
```

Paste into Graphviz Online — considerably more interesting than this morning's.

**Destroy before moving on** — this is the expensive part of the day:

*(Run from `~/terraform-training/06-vms-with-lb`)*
```bash
terraform destroy
# type: yes
```

**💬 SLACK — snippet 8**, the whole load balancer set — **definitely post this, it's five resources**. *(Contents as in the `main.tf` block above.)*

**Solution — Part A**

The complete `main.tf`:

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

resource "azurerm_resource_group" "vm_resource_group" {
  name     = "rg-vm-jbloggs-devops"
  location = "northeurope"
}

resource "azurerm_virtual_network" "vm_vnet" {
  name                = "vnet-jbloggs-devops"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
}

resource "azurerm_subnet" "public_subnets" {
  for_each             = { "1" = "10.0.1.0/24", "2" = "10.0.2.0/24", "3" = "10.0.3.0/24" }
  name                 = "subnet-public-${each.key}"
  resource_group_name  = azurerm_resource_group.vm_resource_group.name
  virtual_network_name = azurerm_virtual_network.vm_vnet.name
  address_prefixes     = [each.value]
}

# --- One public IP and NIC per subnet ---

resource "azurerm_public_ip" "http_server_pips" {
  for_each            = azurerm_subnet.public_subnets
  name                = "pip-http-server-${each.key}"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

resource "azurerm_network_interface" "http_server_nics" {
  for_each            = azurerm_subnet.public_subnets
  name                = "nic-http-server-${each.key}"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = each.value.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.http_server_pips[each.key].id
  }
}

resource "azurerm_network_interface_security_group_association" "http_server_nics_nsg" {
  for_each                  = azurerm_network_interface.http_server_nics
  network_interface_id      = each.value.id
  network_security_group_id = azurerm_network_security_group.http_server_nsg.id
}

# --- Three VMs, one per subnet ---

resource "azurerm_linux_virtual_machine" "http_servers" {
  for_each              = azurerm_subnet.public_subnets
  name                  = "http-server-${each.key}"
  resource_group_name   = azurerm_resource_group.vm_resource_group.name
  location              = azurerm_resource_group.vm_resource_group.location
  size                  = "Standard_B1s"
  admin_username        = "azureuser"
  network_interface_ids = [azurerm_network_interface.http_server_nics[each.key].id]

  admin_ssh_key {
    username   = "azureuser"
    public_key = file(var.azure_ssh_public_key)
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts-gen2"
    version   = data.azurerm_platform_image.ubuntu_latest.version
  }

  tags = {
    name = "http-server-${each.key}"
  }

  connection {
    type        = "ssh"
    host        = azurerm_public_ip.http_server_pips[each.key].ip_address
    user        = "azureuser"
    private_key = file(var.azure_ssh_private_key)
  }

  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update -y",
      "sudo apt-get install apache2 -y",
      "sudo systemctl start apache2",
      "echo Welcome - Virtual Server ${each.key} is at ${azurerm_public_ip.http_server_pips[each.key].ip_address} | sudo tee /var/www/html/index.html"
    ]
  }
}

# --- Load balancer ---

resource "azurerm_public_ip" "lb_pip" {
  name                = "pip-lb"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

resource "azurerm_lb" "lb" {
  name                = "lb"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
  sku                 = "Standard"

  frontend_ip_configuration {
    name                 = "lb-frontend"
    public_ip_address_id = azurerm_public_ip.lb_pip.id
  }
}

resource "azurerm_lb_backend_address_pool" "lb_backend_pool" {
  loadbalancer_id = azurerm_lb.lb.id
  name            = "http-servers-pool"
}

resource "azurerm_network_interface_backend_address_pool_association" "http_server_nics_pool" {
  for_each                = azurerm_network_interface.http_server_nics
  network_interface_id    = each.value.id
  ip_configuration_name   = "internal"
  backend_address_pool_id = azurerm_lb_backend_address_pool.lb_backend_pool.id
}

resource "azurerm_lb_probe" "http_probe" {
  loadbalancer_id = azurerm_lb.lb.id
  name            = "http-probe"
  port            = 80
  protocol        = "Http"
  request_path    = "/"
}

resource "azurerm_lb_rule" "http_rule" {
  loadbalancer_id                = azurerm_lb.lb.id
  name                           = "http-rule"
  protocol                       = "Tcp"
  frontend_port                  = 80
  backend_port                   = 80
  frontend_ip_configuration_name = "lb-frontend"
  backend_address_pool_ids       = [azurerm_lb_backend_address_pool.lb_backend_pool.id]
  probe_id                       = azurerm_lb_probe.http_probe.id
}
```

---

#### Part B1 (≈10 min) — Set up the backend-state project

*(Run from `~/terraform-training`)*
```bash
mkdir -p 07-backend-state/users
mkdir -p 07-backend-state/backend-state
cd 07-backend-state/users
```

*(Run from `~/terraform-training/07-backend-state/users`)*
```bash
touch main.tf outputs.tf
```

Keep the `users` project deliberately simple:

**07-backend-state/users/main.tf**
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

# REMEMBER: YOUR tenant domain, not mine
resource "azuread_user" "my_azuread_user" {
  user_principal_name = "my_iam_user_def@jbloggs.onmicrosoft.com"
  display_name        = "my_iam_user_def"
  mail_nickname       = "my_iam_user_def"
  password            = "ChangeMe123!ChangeMe"
}
```

**07-backend-state/users/outputs.tf**
```tf
output "my_azuread_user_complete_details" {
  value = azuread_user.my_azuread_user
}
```

*(Run from `~/terraform-training/07-backend-state/users`)*
```bash
terraform init
terraform apply
# type: yes
ls -la          # note terraform.tfstate is HERE, locally
```

**ASK** <br>
Why two separate sub-projects rather than one? <br>
**ANSWER** <br>
**Chicken and egg.** You can't store a project's state in a storage account that the same project is responsible for creating — the first `apply` would need the backend to exist before it had created it. So `backend-state` creates the storage; `users` consumes it. And in practice one backend storage account serves **many** projects: users, VMs, load balancers, all of them.

---

#### Part B2 (≈15 min) — Build the storage account

*(Run from `~/terraform-training/07-backend-state/backend-state`)*
```bash
touch main.tf
```

**backend-state/main.tf**
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

resource "azurerm_resource_group" "backend_resource_group" {
  name     = "rg-backend-state-jbloggs-devops"
  location = "uksouth"
}

resource "azurerm_storage_account" "organisation_backend_state" {
  name                     = "stdevappsbackendjbloggs"
  resource_group_name      = azurerm_resource_group.backend_resource_group.name
  location                 = azurerm_resource_group.backend_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "LRS"

  blob_properties {
    versioning_enabled = true
  }
}

resource "azurerm_storage_container" "tfstate_container" {
  name                  = "tfstate"
  storage_account_name  = azurerm_storage_account.organisation_backend_state.name
  container_access_type = "private"
}
```

**On naming the storage account** — you have a choice of scope:

*REFER TO RESOURCE 7 - SLIDEE* <br>

- **One account for everything** — all state, all projects, all environments
- **One per environment** — `stdevbackend`, `stprodbackend`
- **One per application per environment** — `stapplicationnamebackend`

Remember: no hyphens or underscores, globally unique. We're using `stdevappsbackend<yourname>`.

Three things this gives us:
- **`versioning_enabled = true`** — every state write keeps the previous version, so a corrupted state can be rolled back
- **Encryption at rest** — on by default. No extra resource, unlike AWS
- **`container_access_type = "private"`** — no anonymous access. **Absolutely not optional** for a file containing secrets

*(Run from `~/terraform-training/07-backend-state/backend-state`)*
```bash
terraform init
terraform validate
terraform apply
# type: yes
```

---

#### Part B3 (≈15 min) — Migrate the users project

Point `users` at that storage. Add a **`backend`** block **inside** the `terraform { }` block:

**07-backend-state/users/main.tf**
```tf
terraform {
  required_providers {
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.0"
    }
  }

  # NEW CONFIG
  backend "azurerm" {
    resource_group_name  = "rg-backend-state-jbloggs-devops"
    storage_account_name = "stdevappsbackendjbloggs"
    container_name       = "tfstate"
    key                  = "07-backend-dev-users.tfstate"
  }
}
```

- `backend "azurerm"` — the backend *type*. Others include `s3`, `gcs`, `remote`
- `resource_group_name` / `storage_account_name` / `container_name` — where the storage lives
- `key` — the **blob name** the state is stored under. **This is what separates one project's state from another's** in the same container

**NOTE FOR TRAINERS** <br>
Values in a `backend` block **cannot be interpolated** — no `var.` references, no expressions. A genuine limitation that surprises people, and it exists because the backend must be resolved **before** Terraform has evaluated any variables. Teams handle it with **partial configuration**: leave values out and supply them at init time with `terraform init -backend-config=dev.hcl`. Worth mentioning as the answer to "but how do we vary this per environment?" <br>
**END OF NOTE**

*(Run from `~/terraform-training/07-backend-state/users`)*
```bash
terraform init
```

![understanding-resource-provisioning-65](./resources/understanding-resource-provisioning-65.png)

Terraform detects the backend change and asks whether to **copy the existing local state** to the new backend.

- Type: `yes`

If it errors, use `terraform init -reconfigure`.

Now delete the local copies — they're no longer authoritative:

*(Run from `~/terraform-training/07-backend-state/users`)*
```bash
rm terraform.tfstate terraform.tfstate.backup
ls -la
```

*(In the Azure Portal — **Storage accounts → stdevappsbackend... → Containers → tfstate**)*

There's a blob named exactly what you set as `key`. Open it — it's your Known State.

*(Run from `~/terraform-training/07-backend-state/users`)*
```bash
terraform plan
```

**It works with no local state file at all** — Terraform fetched it from Azure.

#### A better key structure

A flat filename works, but a **hierarchical** key using forward slashes gives Azure pseudo-folders and scales far better.

First destroy, because we're changing where state lives — otherwise you'd get a fresh empty state trying to create a user that already exists:

*(Run from `~/terraform-training/07-backend-state/users`)*
```bash
terraform destroy
# type: yes
```

**users/main.tf**
```tf
  backend "azurerm" {
    resource_group_name  = "rg-backend-state-jbloggs-devops"
    storage_account_name = "stdevappsbackendjbloggs"
    container_name       = "tfstate"
    # key = environment / application / project / state
    # NEW CONFIG
    key = "dev/07-backend-state/users/backend-state.tfstate"
  }
```

*(Run from `~/terraform-training/07-backend-state/users`)*
```bash
terraform init -reconfigure
terraform apply
# type: yes
```

*(In the Azure Portal — the `tfstate` container)* — refresh until you see the `dev/` prefix.

**ASK** <br>
Blob storage is flat — there are no real folders. So what does putting slashes in the key achieve? <br>
**ANSWER** <br>
Azure's tooling **displays** slash-separated prefixes as a folder hierarchy, so it's browsable. More importantly it gives a **consistent, predictable convention** — any engineer can work out where a given project's state lives — and you can grant **RBAC access by prefix**, letting a team read `dev/` state but not `prod/`. The structure carries meaning even though the storage is flat. It's exactly like S3 "folders", or the way you'd namespace Redis keys.

**💬 SLACK — snippet 9**, the backend block:
```tf
  # Goes INSIDE the terraform { } block.
  # Values CANNOT be variables — this must be literal.
  backend "azurerm" {
    resource_group_name  = "rg-backend-state-CHANGEME-devops"
    storage_account_name = "stdevappsbackendCHANGEME"
    container_name       = "tfstate"
    key                  = "dev/07-backend-state/users/backend-state.tfstate"
  }
```
```bash
terraform init -reconfigure     # then 'yes' to copy state up
rm terraform.tfstate terraform.tfstate.backup
terraform plan                  # works with NO local state
```

**Solution — Part B**

**backend-state/main.tf** — as written above.

**users/main.tf**
```tf
terraform {
  required_providers {
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.0"
    }
  }

  backend "azurerm" {
    resource_group_name  = "rg-backend-state-jbloggs-devops"
    storage_account_name = "stdevappsbackendjbloggs"
    container_name       = "tfstate"
    key                  = "dev/07-backend-state/users/backend-state.tfstate"
  }
}

provider "azuread" {}

resource "azuread_user" "my_azuread_user" {
  user_principal_name = "my_iam_user_def@jbloggs.onmicrosoft.com"
  display_name        = "my_iam_user_def"
  mail_nickname       = "my_iam_user_def"
  password            = "ChangeMe123!ChangeMe"
}
```

The full sequence:

*(Run from `~/terraform-training/07-backend-state/backend-state`)*
```bash
terraform init
terraform apply         # type: yes — creates RG, storage account, container
```

*(Run from `~/terraform-training/07-backend-state/users`)*
```bash
terraform init          # type: yes to copy state to the backend
rm terraform.tfstate terraform.tfstate.backup
terraform plan          # works with no local state
terraform destroy       # type: yes, BEFORE changing the key
# ...update key to the hierarchical form...
terraform init -reconfigure
terraform apply         # type: yes
```

**Stretch, if anyone finishes early:**
- Open two terminals in the `users` folder. Start `terraform apply` in one and, while it's waiting for confirmation, run `terraform plan` in the other. **Watch the second one report the state is locked**, and see whose lock it is
- Turn on blob versioning in the Portal and look at the version history of your state blob after a few applies

<br>
<br>

### 16:45–17:00 — Wrap-up, Destroy, & Q&A
*(Activity: 10 min)*

#### Clean up — do this first

*(Run from `~/terraform-training/07-backend-state/users`)*
```bash
terraform destroy
# type: yes
```

*(Run from `~/terraform-training/07-backend-state/backend-state`)*
```bash
terraform destroy
# type: yes
```

*(Run from `~/terraform-training/06-vms-with-lb`)*
```bash
terraform destroy
# type: yes
```

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform destroy
# type: yes
```

Then verify:

*(Run from anywhere)*
```bash
az resource list -o table
az group list -o table
```

**💬 SLACK — snippet 10**:
```bash
# Destroy in THIS ORDER. users BEFORE backend-state.
cd ~/terraform-training/07-backend-state/users        && terraform destroy
cd ~/terraform-training/07-backend-state/backend-state && terraform destroy
cd ~/terraform-training/06-vms-with-lb                 && terraform destroy
cd ~/terraform-training/05-virtual-machines            && terraform destroy

# Then confirm from anywhere:
az resource list -o table
az group list -o table
```

**Destroy `users` before `backend-state`.** If you delete the storage account first, the `users` project loses its state entirely and has no idea it owns an Azure AD user — leaving an orphaned resource Terraform can no longer manage.

**ASK** <br>
That ordering problem is interesting. `users` depends on `backend-state`, but Terraform has no idea — there's no reference between them. What does that tell us? <br>
**ANSWER** <br>
That **the dependency graph only works within a single project.** Across projects, ordering is a **human responsibility**. That's the trade-off for splitting infrastructure by lifecycle: smaller blast radius and faster plans, but you have to know and document the ordering yourself. Teams handle it with naming conventions, READMEs, and pipelines that run projects in a defined order. It's the same problem as ordering database migrations across services — the tool can't see across the boundary, so a human has to.

#### What you built today

- **A network from scratch** — VNet, three subnets across availability zones, NSG with priority-ordered rules
- **A VM that couldn't exist alone** — public IP, NIC, NSG association, SSH key, all wired by the dependency graph
- **Configuration at creation** — `remote-exec` over SSH installing and starting Apache
- **Immutability** — why you replace servers rather than patch them
- **`data` sources** — reading things Terraform doesn't own
- **`terraform graph`** — seeing the graph you'd been building implicitly all along
- **Scaling with `for_each`** — one block, three servers, keyed consistently across every related resource
- **A load balancer** — frontend, backend pool, health probe, rule
- **A remote backend** — state in Azure Blob Storage, with locking via blob lease and encryption by default

**ASK** <br>
Think back to Session 3. What **single thing** that we did today makes it possible to run Terraform from a Jenkins pipeline at all? <br>
**ANSWER** <br>
**The remote backend.** A Jenkins agent is a disposable container — no local state file, and anything it writes disappears. With state in Azure Blob Storage the pipeline reads the same authoritative state everyone else uses, **takes a lock** so a concurrent run can't corrupt it, and writes back. Without it, automated Terraform is impossible. That's why it's the last thing we did today and the first thing Session 8 relies on.

Where this goes:
- **Today** — real infrastructure, provisioned and configured, with shared state
- **Session 6, Kubernetes** — running containers on infrastructure like this
- **Session 7/8, Integration** — a Git push flowing through test, build, `terraform plan`, human review, `terraform apply`, deploy

**Before you leave:**
1. `session5-notes.md` complete, including your backend details — **Session 8 needs them**
2. `az resource list -o table` is empty
3. **No `.pem` file and no state file has been committed anywhere**

**Q&A**

<br>
<br>

### Exercise (take-home / reinforcement)

Individually or in pairs. **At least one practical and one research task.**

**Practical**

1. Rebuild the `05-virtual-machines` project **from memory** — network, NSG, public IP, NIC, VM
2. Change the NSG so SSH is only allowed from **your own IP** rather than `0.0.0.0/0`. Find it with `curl ifconfig.me`. Confirm you can still connect
3. Add an `environment` variable defaulting to `dev`, and use it in every resource name and as a tag
4. Add a second load balancing rule for **HTTPS on port 443**, and a matching NSG rule. You won't have a certificate, so just confirm the **plan** is correct — don't chase a working TLS setup
5. Move the `05-virtual-machines` state into your remote backend, under a sensible hierarchical key

**Research**

6. **Provisioners versus cloud-init.** Read about **`custom_data`** on `azurerm_linux_virtual_machine`. Write half a page on why HashiCorp calls provisioners "a last resort", and what `custom_data` does better
7. **Read about `lifecycle` blocks** — `prevent_destroy`, `create_before_destroy`, `ignore_changes`. Write one sentence each, and say which you'd put on your backend storage account and why
8. **(Stretch)** Research **Azure Bastion**. What problem does it solve that our `0.0.0.0/0` SSH rule creates, and what does it cost?
9. **(Stretch)** Find out what happens to a **blob lease** if the process holding it dies. How long before it expires?

**Solutions** *(for the guided ones — 2, 3, 6, 7)*

**Take-home 2** — restricting SSH to your own IP:

*(Run from anywhere)*
```bash
curl ifconfig.me      # e.g. 81.134.22.7
```

```tf
resource "azurerm_network_security_rule" "ssh_ingress" {
  name                        = "AllowSSH"
  priority                    = 110
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "22"
  # /32 means exactly this ONE address — no range
  source_address_prefix       = "81.134.22.7/32"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.vm_resource_group.name
  network_security_group_name = azurerm_network_security_group.http_server_nsg.name
}
```

`/32` fixes all 32 bits, so the range is exactly one address. Note this **breaks if your IP changes** — which is why real setups use a VPN range or Azure Bastion.

**Take-home 3** — an environment variable threaded through:
```tf
variable "environment" {
  type        = string
  default     = "dev"
  description = "Environment these resources belong to"
}

resource "azurerm_resource_group" "vm_resource_group" {
  name     = "rg-${var.environment}-vm-jbloggs-devops"
  location = "northeurope"

  tags = {
    environment = var.environment
  }
}
```

Now `terraform apply -var="environment=test"` gives a completely separate, clearly-labelled set of resources.

**Take-home 6** — `custom_data` instead of a provisioner:
```tf
resource "azurerm_linux_virtual_machine" "http_server" {
  # ...
  custom_data = base64encode(<<-EOF
    #!/bin/bash
    apt-get update -y
    apt-get install apache2 -y
    systemctl start apache2
    echo "Configured by cloud-init" > /var/www/html/index.html
  EOF
  )
}
```

**Why it's preferred:** the provisioner runs **from your machine**, needs SSH reachable, needs the private key present, and fails the whole apply if the network hiccups. `custom_data` hands the script to **Azure**, which runs it **on the VM at first boot** via cloud-init — no SSH, no key, no inbound port 22 required, and it works identically from a Jenkins agent that has no key at all. Note `base64encode` (Azure requires the payload encoded) and `<<-EOF`, which is a **heredoc** — the `-` allows the closing marker to be indented. Exactly the syntax from Session 2.

**Take-home 7** — `lifecycle` blocks:

| Directive | Does |
|---|---|
| `prevent_destroy = true` | Terraform **refuses to destroy** this resource, erroring instead |
| `create_before_destroy = true` | On replacement, create the new one **before** deleting the old — avoids downtime |
| `ignore_changes = [tags]` | Ignore drift in named attributes, so Terraform stops trying to revert them |

You'd put **`prevent_destroy`** on your **backend storage account** — losing it means losing the state for every project that uses it, so a stray `terraform destroy` in that folder should error rather than proceed. `ignore_changes` is useful where another system legitimately modifies a resource (an autoscaler changing instance counts, or a tagging policy adding tags).

<br>
<br>

### Conclusion

- **Inform** students this marks the end of Terraform Part 2
- **Reinforce** the two headline ideas: **infrastructure has dependencies**, and Terraform's graph manages them for you in both directions; and **state belongs somewhere shared**, or a team can't work and a pipeline can't run
- **Confirm** everyone has run `terraform destroy` in all four folders and checked `az resource list -o table` — VMs, public IPs and load balancers all cost money
- **Confirm** nobody has committed a `.pem` file or a state file
- **Confirm** everyone's backend details are recorded — **Session 8 depends on them**
- **Preview** what's next: **Kubernetes**, running containers on infrastructure like this; then **integration**, where the pipeline from Session 3 runs the Terraform from Sessions 4 and 5
- **Direct** students to the take-home exercises and the [azurerm provider docs](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs), particularly the Virtual Machine and Load Balancer pages

---

[Back](./README.md)

---
