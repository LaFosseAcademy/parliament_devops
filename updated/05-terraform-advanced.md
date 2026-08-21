# Terraform: Resource Provisioning & Remote State on Azure — Trainer Script

A full day taking trainees from "I can create a storage account and a user in Terraform" to "I can provision a networked, load-balanced fleet of virtual machines, configure them on creation, and store my state safely in a shared remote backend". This is written as a script — read it out, adapt where it feels natural, but the intent is that you could deliver this cold from the page.

## Organisation

### Audience

Trainees who completed the **Terraform & Infrastructure as Code with Azure** day. They can write resource blocks, reference one resource from another, run the `init`/`plan`/`apply`/`destroy` cycle, use variables, and explain the three states. They know `count` vs `for_each` and why `for_each` usually wins.

They have **not** yet provisioned anything with networking attached, used a `provisioner`, used a `data` source, or moved state off their laptop. That's today.

They also know **Azure networking concepts** from the Azure Fundamentals session — VNets, subnets, NSGs — but have only ever clicked them into existence.

### How this document is laid out — read before delivering

Terraform is directory-scoped, and today we work across several project folders plus the Azure Portal. Every instruction block is labelled:

- *(Run from `~/terraform-training/05-virtual-machines`)* — a **terminal** command, in that exact folder
- *(In the Azure Portal — Virtual machines)* — a **browser** action, starting from that screen

Navigation is written as breadcrumbs, e.g. **Storage accounts → stdevappsbackend... → Containers → tfstate**.

The folders we build today, continuing yesterday's numbering:

| Folder | What it's for |
|---|---|
| `~/terraform-training/05-virtual-machines` | One VM, VNet, subnets, NSG, public IP, NIC, provisioner |
| `~/terraform-training/06-vms-with-lb` | Three VMs across zones, behind a Load Balancer |
| `~/terraform-training/07-backend-state` | Remote state — a `users` project and a `backend-state` project |
| `~/azure/azure_ssh_keys` | Where SSH private keys are stored (never in the repo) |

**Every activity has a `**Solution**` block** immediately afterwards.

### Duration & schedule

Full day, **09:00–17:00**, with a one-hour lunch and two 15-minute breaks. Roughly **4 hours of instruction and 3 hours of hands-on**, weighted towards a large end-of-day capstone.

| Time | Section | Shape |
|---|---|---|
| 09:00–09:15 | Welcome & recap: where we got to | Talk |
| 09:15–10:00 | What a VM actually needs: a tour of the Portal wizard | Talk + Portal walkthrough |
| **10:00–10:15** | **Break** | |
| 10:15–11:15 | Building the network: VNet, subnets, NSG and rules | **Exercise** |
| 11:15–12:15 | SSH keys, public IP, NIC, and your first VM | **Exercise** |
| 12:15–13:00 | Provisioners: configuring the VM on creation | **Exercise + Challenge** |
| **13:00–14:00** | **Lunch** | |
| 14:00–14:30 | Immutable servers, `data` sources and `terraform graph` | Talk + demo |
| 14:30–15:00 | Why local state doesn't scale | Talk + discussion |
| **15:00–15:15** | **Break** | |
| 15:15–16:45 | Capstone: load-balanced VMs & a remote backend | **Exercise (1 hr 30 min)** |
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
  - **infrastructure-as-code-terraform/understanding-resource-provisioning-on-azure/starter-code**
- **Make sure**, *before the session starts*, every student has:
  - Their `~/terraform-training` folder and working `ARM_*` environment variables from yesterday
  - **`terraform --version`** and **`az account show`** both responding
  - A **globally-unique prefix** they've been using for storage account names
  - Nothing left running from yesterday — have them run `az resource list -o table` first thing

**NOTE FOR TRAINERS — the three things that will eat your day** <br>
**(1) Environment variables.** The `ARM_*` exports don't survive a new terminal unless they went into `.zshrc`. First thing this morning, have everyone run `export | grep ARM`. <br>
**(2) SSH key file permissions.** `chmod 400` on the private key is mandatory — SSH silently refuses keys other users could read, and the resulting Terraform error ("Permission denied (publickey)") looks like a Terraform problem rather than a file-permissions one. <br>
**(3) Provisioner timeouts.** `remote-exec` needs port 22 reachable *and* the VM fully booted. If a student's apply hangs for five minutes and then fails, it's nearly always a missing SSH NSG rule or a missing public IP. Have them check those two things before anything else. <br>
**END OF NOTE**

## Learning objectives

- **Explain** what a virtual machine needs around it before it can exist — network, subnet, NIC, public IP, security rules
- **Build** a VNet, subnets and a Network Security Group with explicit rules in Terraform
- **Understand** how Azure NSG rules differ from AWS security groups — priority, direction, access, and default outbound
- **Provision** a Linux VM with an SSH key and configure it on creation using a `remote-exec` provisioner
- **Explain** why immutable servers are the recommended Infrastructure as Code pattern
- **Use** a `data` source to look up information Terraform doesn't manage
- **Visualise** a dependency graph with `terraform graph`
- **Scale** a single VM into a load-balanced fleet using `for_each`
- **Migrate** state to a remote backend in Azure Blob Storage, and explain locking and key structure

<br>

## Sequence

### 09:00–09:15 — Welcome & Recap: Where We Got To

Morning. Quick recap, because today builds directly on yesterday.

Yesterday you learned the Terraform *mechanics*: resource blocks, references, the three states, `plan` and `apply`, variables, and iterating with `count` and `for_each`. But everything you created was, honestly, quite small — a resource group, a storage account, some Azure AD users. Things with no dependencies and nothing to connect to.

Today you provision **real infrastructure**: virtual machines that sit inside a network, behind security rules, reachable from the internet, and eventually behind a load balancer. And you'll hit the thing that makes this feel like real engineering — **a VM in Azure can't exist on its own.** It needs a network, a subnet, a network interface, an IP address and security rules, and all of them have to be created in the right order.

That's where yesterday's dependency graph stops being a nice idea and starts doing serious work for you.

Then this afternoon we fix the last big weakness in what you've built: **your state file is sitting on your laptop.** That's fine alone and useless in a team. We'll move it into Azure Blob Storage.

Standing rule again: **anything we create, we destroy before we leave.** Today's resources genuinely cost money — VMs and load balancers are not free-tier-forever the way a couple of AD users were.

**First, everyone check three things:**

*(Run from `~/terraform-training`)*
```bash
export | grep ARM        # your four Azure credentials should be here
az account show          # confirm you're signed in
az resource list -o table  # confirm nothing's left running from yesterday
```

If `export | grep ARM` comes back empty, re-export the four variables now — everything today depends on them.

<br>
<br>

### 09:15–10:00 — What a VM Actually Needs: A Tour of the Portal Wizard

Before writing any Terraform, let's walk the Portal wizard for creating a VM. Not to create one — to *inventory everything it asks us for*, so we know what we'll have to build in code.

*(In the Azure Portal — search **Virtual machines**)*

**What is a virtual server?** When a company has a server in a data centre, it's a **physical server** — that's where applications and databases get deployed. A **virtual machine** is a slice of a physical machine in someone else's data centre, which you rent and reach over the internet. They're not really "in the cloud" — they're in physical buildings all over the world; you just don't have to think about the building.

In Azure they're **Virtual Machines**, or **VMs**.

![understanding-resource-provisioning-1](./resources/understanding-resource-provisioning-1.png)

We've been using `uksouth`. Today let's switch to **North Europe** (Dublin), just to prove regions are a one-line change.

**ASK** <br>
By having multiple regions, what do we improve? <br>
**ANSWER** <br>
**Availability** — a failure in one region doesn't take everything down. And **low latency** — users get served from somewhere physically close to them, so responses are faster and more consistent.

Let's keep a scratch note of the configuration we'll need to translate into Terraform.

*(Run from `~/terraform-training`)*
```bash
touch config.md
```

**config.md**
```md
Location - northeurope
```

Now walk the wizard, step by step.

![understanding-resource-provisioning-2](./resources/understanding-resource-provisioning-2.png)

#### Basics: project and instance details

Straightforward — a **name**, and a **resource group** to organise it under. Exactly like the Azure AD users yesterday.

#### Image (Marketplace Image)

This is the **software** the machine boots with:
- Ubuntu Server
- Windows Server
- Red Hat Enterprise Linux
- Debian

We'll choose **Ubuntu Server 22.04 LTS** — a common, well-supported default. (**LTS** means Long Term Support: security patches for years, rather than months.)

![understanding-resource-provisioning-3](./resources/understanding-resource-provisioning-3.png)

Look at the detail attached to an image, because several of these terms will show up in our Terraform:

**`urn`** — the unique identifier for a Marketplace image, in the form `publisher:offer:sku:version`. Four colon-separated parts. This is how you unambiguously say "that exact image".

**Gen2 VM** — the image boots using **UEFI** rather than the older Generation 1 / BIOS path.

**NOTE FOR TRAINERS** <br>
Worth a short detour if the room is engaged: <br>
**BIOS** — the **Basic Input/Output System**, the old standard. Firmware on a motherboard chip that does the **power-on-self-test (POST)** (checking processor, memory and storage are alive) and then **bootstrapping** — finding the bootloader, which loads the OS into memory. <br>
**UEFI** — the **Unified Extensible Firmware Interface**, the modern replacement. A graphical rather than text interface, **Secure Boot** (only trusted software loads during boot, protecting against malware), and faster boot times. <br>
Both are firmware that initialises hardware and loads the OS. Azure's **Generation 2** VMs use UEFI; **Generation 1** uses BIOS-style boot. <br>
**END OF NOTE**

**Architecture: x64** — the image supports 64-bit x86, the very common CPU architecture with the widest software and hardware support.

**Accelerated Networking: Supported** — a high-performance networking path, giving **higher throughput** (more data processed per second) and **lower latency**.

**Disk type: Managed Disk** — the storage holding the OS. Azure calls this a Managed Disk; it's roughly an EBS volume in AWS terms.

Note the image identifier down:

**config.md**
```md
Location - northeurope
# NEW CONFIG EXAMPLE
Marketplace Image (urn) - Canonical:0001-com-ubuntu-server-jammy:22_04-lts-gen2:latest
```

#### Instance type (VM size)

We've chosen the software; now the **hardware**. How powerful is this machine:
- vCPUs
- Memory
- Storage
- Network performance

![understanding-resource-provisioning-5](./resources/understanding-resource-provisioning-5.png)

What matters for us is that it's eligible under the **Azure free account** — Azure gives you a **B1s** burstable VM free for 12 months. ("Burstable" means it accumulates CPU credits while idle and can spend them on short bursts of load — perfect for a demo web server, useless for sustained heavy compute.)

**config.md**
```md
Location - northeurope
Marketplace Image (urn) - Canonical:0001-com-ubuntu-server-jammy:22_04-lts-gen2:latest
# NEW CONFIG
VM size - Standard_B1s
```

#### Networking — the part that matters most

*REFER TO RESOURCE 3 - SLIDEE* <br>
![understanding-resource-provisioning-6](./resources/understanding-resource-provisioning-6.png)

In a physical data centre, resources are protected by a firewall on the device — inspecting incoming and outgoing network packets and allowing or blocking them per your security policy.

In the cloud we get the same thing by creating a **Virtual Network (VNet)** — your own private network in the cloud. Inside a VNet you create smaller **subnets** to hold resources.

Read the diagram with them, outside-in:
- Inside a **region** (here, North Europe) sits a **VNet**
- The VNet spans **availability zones** — physically separated locations within that region, each with independent power and networking
- Inside those sit **subnets**, which is where resources actually live

**Private vs public subnets.** A VM in a **private** subnet can only be reached by resources inside the VNet. That's where you'd put:
- a **database** — perhaps only the model layer of an MVC app should reach it
- **servers handling sensitive data**

A VM with a **public IP attached** can be reached from outside the VNet too. That's where you'd put:
- **HTTP servers** that need to serve the internet

**NOTE FOR TRAINERS** <br>
Flag this explicitly, because students coming from AWS material will expect otherwise: **Azure does not pre-provision a default VNet** in every region. AWS gives you a default VPC you can adopt; Azure does not. When the Portal wizard notices you have no VNet, it offers to create one *as part of the wizard* — created on demand, not sitting there in advance. This has a real consequence for our Terraform: we must build the network ourselves, and there's no `data` source to "adopt" a default. <br>
**END OF NOTE**

So we'll create our own VNet with **1** VNet and **3** subnets, one per availability zone.

![understanding-resource-provisioning-10](./resources/understanding-resource-provisioning-10.png)

*(In the Azure Portal — search **Virtual networks**)* — click into the newly created VNet and copy the **VNet resource ID**.

![understanding-resource-provisioning-11](./resources/understanding-resource-provisioning-11.png)

**config.md**
```md
Location - northeurope
Marketplace Image (urn) - Canonical:0001-com-ubuntu-server-jammy:22_04-lts-gen2:latest
VM size - Standard_B1s
# NEW CONFIG
VNet - vnet-emilesherrott-devops
```

#### Firewall (Network Security Group)

Under **Networking** there's also **NIC network security group**.

**Network Security Groups (NSGs)** are how you control traffic to resources beyond just choosing a subnet. They're their own resource, and a subnet — or a specific network interface — gets associated with the rules you define.

The scenario that makes it concrete:
- You want your VM in a **public** subnet so the internet can reach it
- But you want to block anything that isn't HTTP or HTTPS, on ports 80 and 443

So even though traffic can reach the subnet, the NSG blocks what shouldn't reach the VM. Think of an NSG as **a set of firewall rules**.

**ASK** <br>
Does anyone remember what **SSH** traffic is? <br>
**ANSWER** <br>
Secure Shell — it lets you remotely log into and control a machine over an encrypted connection. Conventionally on **port 22**. It's how we'll get onto the VM to configure it.

For a web server we'll allow **SSH (22)** and **HTTP (80)**.

![understanding-resource-provisioning-15](./resources/understanding-resource-provisioning-15.png)

#### Disks and Advanced

Default Managed Disk is fine; leave Advanced alone.

**The point of all that.** Look at everything we had to think about for *one* VM:
- regions
- availability zones
- virtual networks
- subnets
- network security groups

**ASK** <br>
That was maybe fifteen decisions, and we haven't created anything yet. What happens when you need this VM in three regions, for three environments? <br>
**ANSWER** <br>
By hand, it's fifteen decisions times nine, each an opportunity to click the wrong thing — and no record of what you chose or why. That's the ClickOps problem at full scale. In Terraform it's one configuration and a changed variable. This is exactly the case for automating it.

We won't launch from the browser. Let's go and build it in VS Code.

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>

### 10:15–11:15 — Building the Network: VNet, Subnets, NSG and Rules
*(Exercise)*

#### Setting up the project

*(Run from `~/terraform-training`)*
- Run: `mkdir 05-virtual-machines`
- Run: `mv config.md 05-virtual-machines/` — move our notes into the project
- Run: `cd 05-virtual-machines`

*(Run from `~/terraform-training/05-virtual-machines`)*
- Run: `touch main.tf`
- Then open it: `code main.tf`

**main.tf**
```tf
# NEW CONFIG
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
  # UPDATED CONFIG
  location = "northeurope"
}
```

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform init
```

#### Virtual Network and Subnets

Because Azure gives us no default VNet, our first job is building our own.

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

**`address_space = ["10.0.0.0/16"]`** — the range of private IP addresses this VNet can hand out. That `/16` is **CIDR notation**, which you met in the Azure Fundamentals session. Quick refresher:

- An IPv4 address is four numbers, each 0–255, because each is stored in 8 bits
- The number after the slash says **how many bits are fixed** — the network part
- `/16` fixes the first two numbers (`10.0`), leaving the last two free → about 65,000 addresses
- `/24` fixes the first three (`10.0.1`), leaving one free → 256 addresses

**Bigger number = smaller network.** So our VNet is `10.0.0.0/16`, and each subnet carves out a `/24` slice inside it: `10.0.1.0/24`, `10.0.2.0/24`, `10.0.3.0/24`.

**The `for_each` over a map** — this is yesterday's material doing real work:

```tf
for_each = { "1" = "10.0.1.0/24", "2" = "10.0.2.0/24", "3" = "10.0.3.0/24" }
```

`each.key` is `"1"`, `"2"`, `"3"` (used in the subnet names) and `each.value` is the CIDR range. Three subnets from one block, each tracked in state by its key rather than a positional index.

**ASK** <br>
Why write this as a `for_each` over a map rather than three separate `azurerm_subnet` blocks? <br>
**ANSWER** <br>
Less repetition and a single place to change the pattern — but more importantly, later resources can iterate over *this same collection*. When we build three VMs this afternoon, we'll write `for_each = azurerm_subnet.public_subnets` and get one VM per subnet automatically. Three separate blocks couldn't be iterated over like that.

#### Network Security Group

We're building an **HTTP server** — a web server. Conventionally accessed on **port 80** via **TCP**.

**How do TCP and HTTP fit together?**

**TCP** (Transmission Control Protocol) is like two friends on the phone:
- **Connection** — you dial, they answer
- **Conversation** — you take turns talking and listening, like data flowing back and forth
- **Closing the call** — you both say goodbye; the TCP connection closes

**HTTP** is the *structure of what's said* — how browsers and servers phrase requests and responses. So TCP is the connection; HTTP is the conversation happening over it.

We also want **SSH** on **port 22** to connect in and configure the machine.

And because this is a public web server, it must be reachable from anywhere. That means a **CIDR block** specifying which source IPs are allowed.

*REFER TO RESOURCE 4 - SLIDEE* <br>

**`"0.0.0.0/0"`** — the **default route**, meaning "any address anywhere":
- `0.0.0.0` — any address
- `/0` — zero bits fixed, so *every* IP address is included

Azure also accepts the wildcard `"*"` for the same thing.

Now the NSG itself:

**main.tf**
```tf
[ . . . ]

# NEW CODE
# "<provider>_<service-name>" "<resource-name>"
resource "azurerm_network_security_group" "http_server_nsg" {
  name                = "http-server-nsg"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
}
```

Note this creates an **empty** NSG — a container for rules. In Azure we define each rule as its own resource, which is actually the cleaner pattern: rules can be added, changed and reviewed independently without touching the group.

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

| Attribute | Meaning |
|---|---|
| `priority` | **Lower number is evaluated first.** Range 100–4096. Once a rule matches, evaluation stops |
| `direction` | `Inbound` or `Outbound` |
| `access` | `Allow` or `Deny` — Azure NSGs can explicitly **deny**, not just allow |
| `protocol` | `Tcp`, `Udp`, `Icmp` or `*` |
| `source_port_range` | Which port the traffic *came from*. Almost always `*`, because clients pick a random high port |
| `destination_port_range` | Which port it's *arriving at* — this is the one you care about |
| `source_address_prefix` | Which IPs are allowed. `0.0.0.0/0` means anywhere |

**ASK** <br>
Why is `source_port_range` almost always `*` while `destination_port_range` is specific? <br>
**ANSWER** <br>
Because the *destination* port identifies the service — 80 is "the web server", 22 is "SSH". The *source* port is just an arbitrary high-numbered port the client's operating system picked for that connection, and it's different every time. You have no way to predict it and no reason to care.

Now the SSH rule:

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

Note `priority = 110` — different from the HTTP rule's 100. **Priorities must be unique within an NSG.** Leaving gaps (100, 110, 120) is a good habit, so you can slot a rule in between later without renumbering everything.

**NOTE FOR TRAINERS** <br>
Worth calling out as a genuine platform difference: in AWS, a security group denies all outbound traffic unless you write an explicit `egress` rule. Azure NSGs ship with built-in default rules including **`AllowInternetOutBound`**, which already permits all outbound traffic. So we don't need an outbound rule here — and that's a deliberate design choice, not an oversight in our config. Students who've seen AWS material will look for it. <br>
**END OF NOTE**

**ASK** <br>
We've opened SSH (port 22) to `0.0.0.0/0` — the entire internet. Is that wise? <br>
**ANSWER** <br>
No, not for anything real. Every SSH server exposed to the open internet gets continuously probed by automated bots trying default credentials. In production you'd restrict `source_address_prefix` to your office or VPN range, or avoid public SSH entirely using Azure Bastion. We're doing it because a classroom full of people on different networks is otherwise painful — but it's a shortcut, and worth naming as one.

Now apply:

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform validate
terraform apply
```

![understanding-resource-provisioning-16](./resources/understanding-resource-provisioning-16.png)
![understanding-resource-provisioning-17](./resources/understanding-resource-provisioning-17.png)

- Type: `yes`

If you get errors, check your `location` matches the region you intend to work in.

*(In the Azure Portal — search **Network security groups**)* — click into `http-server-nsg`. Your inbound rules are there, with the ports you specified.

![understanding-resource-provisioning-19](./resources/understanding-resource-provisioning-19.png)

Scroll down and note the **default outbound rules**, already present without you defining anything.

#### Tags, and ForceNew

**main.tf**
```tf
resource "azurerm_network_security_group" "http_server_nsg" {
  name                = "http-server-nsg"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
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

Get into the habit of tagging — `environment`, `owner`, `cost-centre` — exactly as in the Azure Fundamentals session. Once you have hundreds of resources, untagged infrastructure becomes unmanageable and unattributable on the bill.

Now go and read the docs, because we've written a lot of config:

*GOOGLE: terraform azurerm network security group* <br>
https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/network_security_group

Scroll to the **Argument Reference**. Notice some arguments are marked **`Changing this forces a new resource to be created`** — often shortened to **ForceNew**.

![understanding-resource-provisioning-20](./resources/understanding-resource-provisioning-20.png)

**ASK** <br>
`name` is marked ForceNew. What does that mean will happen if you rename an NSG that a running VM depends on? <br>
**ANSWER** <br>
Terraform will **destroy and recreate** it — and because the VM's association depends on it, that cascades. This is the same "must be replaced" behaviour as the storage account name yesterday. The lesson is identical: **read the plan**. The provider documentation tells you in advance which attributes are safe to change and which are destructive, and it's worth checking before you edit something in production.

**HANDS ON (20 min)** <br>
*(Run from `~/terraform-training/05-virtual-machines`)*
1. Create the project, `main.tf` with the provider and resource group in `northeurope`, and run `terraform init`.
2. Add the VNet with address space `10.0.0.0/16`.
3. Add the three subnets using `for_each` over a map.
4. Add the NSG, plus inbound rules for HTTP (80) and SSH (22).
5. `terraform validate`, then `terraform apply`.
6. *(In the Azure Portal)* find your NSG and confirm both inbound rules, and note the default outbound rules.
7. Add a `tags` block to the NSG and apply again.
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

Note the creation order in the output: **resource group → VNet → subnets and NSG → NSG rules**. You never specified that order — the references between resources built the dependency graph.

<br>
<br>
### 11:15–12:15 — SSH Keys, Public IP, NIC, and Your First VM
*(Exercise)*

#### Creating an SSH key

We've allowed SSH traffic through the firewall — but that only opens the door. To actually connect we need an **SSH key pair**, and the public half has to be attached to the VM **when it's first created**. So the key comes first.

**How key pairs work, briefly.** You generate two mathematically linked files. The **public key** can be shared freely and gets installed on the server. The **private key** stays on your machine and proves you're you. The server encrypts a challenge with the public key; only the matching private key can answer it. You never send your private key anywhere.

*(In the Azure Portal — search **SSH keys**)*

![understanding-resource-provisioning-21](./resources/understanding-resource-provisioning-21.png)

Creating one asks for:
- **name**
- **key pair type**:
  - **RSA** — the widely-used, long-established public-key algorithm. The standard for many cryptographic operations
  - **Ed25519** — newer, popular for efficiency and security; faster, and in some respects stronger
- **key pair source**:
  - **Generate new key pair** — Azure creates both halves and lets you download the private key
  - **Use existing public key** — you've already run `ssh-keygen` locally and just want Azure to store the public half
- optional **tags**

Create one with:
- **name**: `default-vm-ssh`
- **key pair type**: `RSA`
- **source**: `Generate new key pair`

![understanding-resource-provisioning-22](./resources/understanding-resource-provisioning-22.png)

Hit **Create**. Your browser immediately downloads `default-vm-ssh.pem` — the **private** key.

![understanding-resource-provisioning-23](./resources/understanding-resource-provisioning-23.png)

**This file is how anyone gets full access to your VM.** Like your Terraform state, it must never go near GitHub.

Now set its permissions and store it somewhere sensible:

*(Run from wherever the `.pem` downloaded — usually `~/Downloads`)*
```bash
chmod 400 default-vm-ssh.pem
```

Recall `chmod` from the bash session — **ch**ange **mod**e. The three digits are three groups of people:
- First digit `4` — **you, the owner**: read only
- Second digit `0` — **your group**: nothing
- Third digit `0` — **everyone else**: nothing

**ASK** <br>
Why does SSH insist on `400` rather than something more relaxed like `644`? <br>
**ANSWER** <br>
`644` would let any other user on the machine *read* your private key — which completely defeats the point of it being private. SSH checks this and flatly refuses to use a key with loose permissions. It's a rare and welcome case of a tool protecting you from yourself.

Store it in a consistent place:

*(Run from `~/`)*
```bash
mkdir -p ~/azure/azure_ssh_keys
mv ~/Downloads/default-vm-ssh.pem ~/azure/azure_ssh_keys/
```

You'll also need the **public** half for Terraform. If Azure only gave you the private key, derive the public one from it:

*(Run from `~/azure/azure_ssh_keys`)*
```bash
ssh-keygen -y -f default-vm-ssh.pem > default-vm-ssh.pub
ls -la
```

`ssh-keygen -y` reads a private key (`-f`) and prints its matching public key, which we redirect into a `.pub` file with `>` — the redirection from the bash session.

**NOTE FOR TRAINERS** <br>
You *could* generate a key pair in Terraform with the `tls_private_key` resource, or store an existing public key with `azurerm_ssh_public_key`. The catch is that `tls_private_key` writes the **private key into your state file in plain text** — which is exactly the thing we spend this afternoon trying to avoid. Doing it out-of-band as we have is the more honest teaching path, and worth saying out loud, because students will find `tls_private_key` in the docs and wonder why we didn't use it. <br>
**END OF NOTE**

This step is easily forgotten and blocks everything downstream, so make a note of it.

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

Three resources, each doing one job:

**`azurerm_public_ip`** — a routable internet address.
- `allocation_method = "Static"` — the address is reserved and doesn't change if the VM restarts. `Dynamic` would give you a new one each time, which is useless once anything points at it
- `sku = "Standard"` — the modern tier. Standard public IPs are **secure by default**: closed to inbound traffic unless an NSG explicitly allows it. That's why our NSG rules matter

**`azurerm_network_interface`** — the virtual network card connecting the VM to a subnet.
- `ip_configuration { }` — a nested block (no `=`, remember)
- `subnet_id = azurerm_subnet.public_subnets["1"].id` — **note the `["1"]`**. Because `public_subnets` was created with `for_each` over a map, referencing it requires a key. This is exactly the difference from yesterday: with `count` you'd write `[0]`; with `for_each` you write `["1"]`, the map key
- `private_ip_address_allocation = "Dynamic"` — Azure picks an internal address from the subnet's range. Fine, because internal addressing is handled by Azure's DNS

**`azurerm_network_interface_security_group_association`** — a **join resource**, whose only job is to link two other resources together. It has no properties of its own beyond the two IDs.

**ASK** <br>
Why is the NSG association a *separate resource*, rather than just an attribute on the network interface? <br>
**ANSWER** <br>
Because the relationship is many-to-many and has its own lifecycle. One NSG can be associated with many NICs and many subnets; making it a separate resource means you can attach and detach without modifying either side. You'll meet this "join resource" pattern repeatedly in Terraform — the load balancer this afternoon uses several.

#### The Virtual Machine

**main.tf**
```tf
[ . . . ]

resource "azurerm_linux_virtual_machine" "http_server" {
  # NEW CODE
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

# NEW CONFIG
variable "azure_ssh_public_key" {
  default = "~/azure/azure_ssh_keys/default-vm-ssh.pub"
}
```

Working through what a VM needs:

**`size = "Standard_B1s"`** — the hardware, straight from our `config.md` notes.

**`admin_username = "azureuser"`** — **we choose this ourselves.** Worth flagging: in AWS the login (`ec2-user`, `ubuntu`) is baked into the AMI and you have to know it. In Azure you pick it, so there's nothing to look up.

**`network_interface_ids = [ ... ]`** — note the **square brackets**: a VM can have multiple NICs, so this attribute is a list even when there's one.

**`admin_ssh_key { }`** — installs the public key for that user. `file(...)` is a **built-in function** that reads a file from disk and returns its contents as a string. So Terraform reads your `.pub` file at apply time and sends the contents to Azure.

**`os_disk { }`** — the managed disk.
- `caching = "ReadWrite"` — a performance setting for how disk reads and writes are cached
- `storage_account_type = "Standard_LRS"` — standard (HDD-backed) locally-redundant storage. Cheap, fine for training

**`source_image_reference { }`** — the four parts of the `urn` from `config.md`, split into named attributes: `publisher`, `offer`, `sku`, `version`. `"latest"` is an alias Azure resolves at apply time.

Now apply:

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform validate
terraform apply
# type: yes
```

![understanding-resource-provisioning-28](./resources/understanding-resource-provisioning-28.png)

**HANDS ON (20 min)** <br>
1. *(In the Azure Portal)* Create an SSH key called `default-vm-ssh`, download the `.pem`, `chmod 400` it, and move it to `~/azure/azure_ssh_keys/`.
2. *(Run from `~/azure/azure_ssh_keys`)* Derive the `.pub` public key with `ssh-keygen -y`.
3. *(Run from `~/terraform-training/05-virtual-machines`)* Add the public IP, NIC and NSG association resources.
4. Add the `azurerm_linux_virtual_machine` and the `azure_ssh_public_key` variable.
5. `terraform apply`, and confirm the VM appears in the Portal.
6. *(In the Azure Portal)* Find the VM's public IP address. Then try `ssh -i ~/azure/azure_ssh_keys/default-vm-ssh.pem azureuser@<the-ip>` — you should get a login prompt on the machine you just created in code.
**END OF NOTE**

**Solution**

*(Run from `~/Downloads`, after downloading from the Portal)*
```bash
chmod 400 default-vm-ssh.pem
mkdir -p ~/azure/azure_ssh_keys
mv default-vm-ssh.pem ~/azure/azure_ssh_keys/
```

*(Run from `~/azure/azure_ssh_keys`)*
```bash
ssh-keygen -y -f default-vm-ssh.pem > default-vm-ssh.pub
ls -la          # both .pem and .pub, .pem showing -r--------
```

**main.tf** (added to the network config from the last section):
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

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform validate
terraform apply
# type: yes

# Step 6 — get the IP straight from the CLI rather than clicking
terraform console
```
Inside the console:
```
azurerm_public_ip.http_server_pip.ip_address
```
Then `Ctrl + C`, and:
```bash
ssh -i ~/azure/azure_ssh_keys/default-vm-ssh.pem azureuser@<the-ip>
# type 'yes' to accept the host fingerprint the first time
# then 'exit' to come back
```

If SSH refuses, check in this order: the `.pem` permissions (`ls -la` should show `-r--------`), that your SSH NSG rule applied, and that the public IP actually exists on the NIC.

<br>
<br>

### 12:15–13:00 — Provisioners: Configuring the VM on Creation
*(Exercise + Challenge)*

We have a running, reachable machine — with nothing on it. Let's turn it into an actual web server.

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform console
```
```
azurerm_linux_virtual_machine.http_server
```

Lots of information about the instance. Now let's configure it.

**We said configuration management is Ansible's job, and that's still true** — but Terraform has basic tooling for it, and seeing it makes clear where the boundary sits.

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
- **`host`** — what we're connecting to. Note we reference the **separate `azurerm_public_ip` resource's `ip_address`**, because unlike an AWS `aws_instance`, an Azure VM doesn't carry its own public IP as an attribute — the IP is its own resource
- **`user = "azureuser"`** — the `admin_username` we chose ourselves
- **`private_key = file(...)`** — reads the `.pem` off disk. The **private** half this time, because we're authenticating *as* the client

#### The provisioner block

**main.tf**
```tf
resource "azurerm_linux_virtual_machine" "http_server" {
  [ . . . ]
  connection {
    [ . . . ]
  }

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

`remote-exec` runs commands **on the remote machine** over the connection we defined. `inline` takes a list of commands, run in order.

Walk through each command — this is all bash-session material:

**`sudo apt-get update -y`** — refresh the package index.
- `sudo` runs as root
- `apt-get` is the package manager for Debian/Ubuntu, which is what our image is
- `-y` auto-answers "yes" to prompts. **Essential in automation** — there's no human to press y, exactly like the interactive-prompt problem from the bash session

**`sudo apt-get install apache2 -y`** — install the Apache HTTP server. It responds to requests with static files or routes them onward.

**`sudo systemctl start apache2`** — start the service.

**NOTE FOR TRAINERS** <br>
On Ubuntu the `apache2` package actually starts the service automatically on install, so this line is usually a no-op. Keep it anyway — being explicit makes the script portable (Amazon Linux does *not* auto-start `httpd`) and keeps the "install, then start" structure legible. It's the same argument made in the Linux and Automation session. <br>
**END OF NOTE**

**`echo ... | sudo tee /var/www/html/index.html`** — write our page.
- `echo` outputs the text that follows
- `|` pipes that output into the next command as input
- `sudo tee <file>` reads standard input and writes it to a file

**ASK** <br>
Why `| sudo tee file` rather than the simpler `sudo echo ... > file`? <br>
**ANSWER** <br>
Because of **when the redirection happens**. In `sudo echo x > file`, the shell sets up the `>` redirection *first*, as your normal user — before `sudo` runs anything — so it fails with permission denied on a root-owned directory. Piping into `sudo tee` means the *writing program itself* runs as root. This trips up a lot of people, and it's a genuinely useful thing to know.

`/var/www/html` is Apache's default web root on both Ubuntu and Amazon Linux, so the path doesn't change between them. Installing `apache2` creates a default `index.html` there; we're overwriting it.

Note the `${azurerm_public_ip.http_server_pip.ip_address}` interpolation inside the command — Terraform substitutes the real IP into the message before sending it.

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
Because **provisioners only run at creation time.** They aren't part of a resource's state — Terraform can't inspect a running VM and ask "have these commands been run?". So adding or changing a provisioner doesn't register as a change. The only way to run it is to create the resource fresh.

So:

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform destroy
# type: yes
terraform apply
# type: yes
```

Watch the **destroy order** in the output:

![understanding-resource-provisioning-31](./resources/understanding-resource-provisioning-31.png)

1. **azurerm_linux_virtual_machine** — first
2. **azurerm_network_interface_security_group_association** — the NIC/NSG link
3. **azurerm_network_interface** and **azurerm_public_ip**
4. **azurerm_network_security_rule** and **azurerm_network_security_group** — last

**Exactly the reverse of creation order.** Terraform won't delete an NSG while a VM still depends on it — the dependency graph enforces safety in both directions.

Then on creation:

![understanding-resource-provisioning-32](./resources/understanding-resource-provisioning-32.png)

VNet, subnets, NSG and rules → public IP and NIC → VM. Then it connects via SSH and runs your commands.

*(In your browser)* — paste the public IP into the address bar.

![understanding-resource-provisioning-33.png](./resources/understanding-resource-provisioning-33.png)

**What just happened, end to end:**
1. Your browser routes to that IP, opens a **TCP** connection, and sends an **HTTP GET** request — by default to **port 80**
2. The NSG rule allows port 80 inbound, so it reaches the VM
3. Apache, installed by `remote-exec`, is listening and serves `/var/www/html/index.html`
4. Your message comes back

You've automated the deployment of a working server in Azure.

**HANDS ON (15 min)** <br>
*(Run from `~/terraform-training/05-virtual-machines`)*
1. Add the `connection` block and the `azure_ssh_private_key` variable.
2. Add the `remote-exec` provisioner with all four commands.
3. `terraform apply` — observe that nothing changes, and explain why.
4. `terraform destroy` then `terraform apply`. Watch the destroy order and the create order.
5. Visit the public IP in a browser and confirm your message.
**END OF NOTE**

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
terraform apply          # type: yes — now the provisioner runs
```

**Step 3 answer:** provisioners aren't tracked in state and only execute when a resource is created. Terraform has no way to know whether the commands have run on an existing machine, so changing them produces no diff.

---

**Challenge**

*Direct* students, **in pairs**, to extend the provisioner so that the served page:

* Displays the **VM's name** as well as its IP address
* Includes the **date and time** the server was provisioned
* Is valid **HTML** — an `<h1>` heading and a `<p>` paragraph, rather than one line of plain text
* **OPTIONAL** — also writes a second file, `/var/www/html/health.html`, containing just the word `OK` (we'll want exactly this for the load balancer's health probe this afternoon)

*Provide* this example of what should appear at `http://<your-ip>` as an aid:

```
Welcome to http-server

This server is at 20.31.44.12 and was provisioned on Tue 21 Oct 09:42:15 UTC 2025
```

*Grant* students ~10 minutes.

Hints to offer if they're stuck: `inline` is just a list of shell commands, so you can add as many as you like. Remember that a command running *on the VM* can use the VM's own shell features — `$(date)` will be evaluated there, not by Terraform. And beware: **`${...}` is Terraform's interpolation and `$(...)` is the shell's**, so they don't collide.

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

Points to draw out when revealing:

* **`tee -a` on the second line.** `tee` overwrites by default; `-a` **appends**. Without it, the paragraph would wipe out the heading — the exact `>` vs `>>` lesson from the bash session, in a different disguise.
* **Two different `$` syntaxes side by side.** `${azurerm_public_ip...}` is resolved by **Terraform**, on your laptop, before the command is ever sent. `$(date)` is resolved by **bash on the VM**, at the moment it runs. Getting this backwards is a very common error.
* **Quoting.** The single quotes around the HTML stop bash interpreting the tags, and we drop out of them around `$(date)` so the shell *does* evaluate it.

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
*(Talk + demo)*

#### Why immutability is the point, not a limitation

Before lunch you couldn't change the HTML message with `terraform apply` — you had to destroy and recreate. That felt like a limitation. It's actually the recommended pattern, and worth understanding why.

*REFER TO RESOURCE 5 - SLIDEE* <br>
![understanding-resource-provisioning-34](./resources/understanding-resource-provisioning-34.png)

**The mutable approach.** Suppose servers could be modified in place. Every change means running a script against a live machine. Over months you accumulate a history of scripts, applied in some order, to some machines.

**ASK** <br>
Six months in, you need a brand new server identical to the existing ones. What do you have to do? <br>
**ANSWER** <br>
Re-run **every script, in exactly the original order** — and hope none of them were applied by hand, out of order, or only to some machines. In practice nobody can reconstruct that reliably, which is precisely how "snowflake servers" happen: machines nobody dares touch because nobody knows how they got that way. It's environment drift from the Azure Fundamentals session, at the level of individual servers.

*REFER TO RESOURCE 6 - SLIDEE* <br>
![understanding-resource-provisioning-35](./resources/understanding-resource-provisioning-35.png)

**The immutable approach.** A server is never modified. Want a change? Provision a *new* server from the updated configuration and destroy the old one. Every machine is built once, from a single declarative description, and is therefore identical to every other machine built from it.

The changes we mean are real ones — a security patch, a software upgrade, a config change.

**ASK** <br>
What does this pattern do for your ability to roll back a bad change? <br>
**ANSWER** <br>
It makes rollback trivial and *reliable*. The previous configuration is in Git; you revert the commit and apply, and you get back exactly the previous state — not "the previous state plus whatever hand-fixes accumulated". Compare it to a Docker image: you don't SSH into a running container to patch it, you build a new image and replace the container. Same instinct, one layer down.

#### Data sources

We hard-coded `version = "latest"` in the image reference. That already works — Azure resolves the alias at apply time. But you can't *see* which version you got, which matters for auditing and reproducibility.

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
- `location` — image catalogues vary slightly by region, so we say where
- `publisher / offer / sku` — the same identifiers, **without** the version. That's what we're asking it to find

Note references get a `data.` prefix: `data.azurerm_platform_image.ubuntu_latest.version`.

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform apply
```

No changes — we've looked something up but not used it. Inspect it:

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
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
![understanding-resource-provisioning-50](./resources/understanding-resource-provisioning-50.png)

This may or may not show a change, depending on whether a newer build shipped since you last applied. If it doesn't change, no new image has been published.

**NOTE FOR TRAINERS** <br>
This is a good moment to compare with the AWS material. There, students used `data "aws_default_vpc"` to *adopt* an already-existing default VPC and `data "aws_subnets"` to find its subnets — necessary because AWS provisions those automatically in every region. <br>
Here we needed neither, because we **created our own** VNet and subnets. `azurerm_subnet.public_subnets["1"].id` was already dynamic; there was never a hardcoded value to remove. Azure having no "default network" to adopt means less need for this flavour of data source. Worth a minute — it's a real platform difference, not an omission. <br>
**END OF NOTE**

#### terraform graph

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform graph
```
![understanding-resource-provisioning-51](./resources/understanding-resource-provisioning-51.png)

That output is a **digraph** in the DOT language — a description of every resource and what depends on what.

*(In your browser)* — search "graphviz online" and open [Graphviz Online](https://dreampuf.github.io/GraphvizOnline/). Paste the output in.

Arrows show dependency. So `azurerm_linux_virtual_machine.http_server` depends on:
- `data.azurerm_platform_image.ubuntu_latest`
- `azurerm_network_interface.http_server_nic`
- and through that, the subnet, VNet, public IP and NSG

![understanding-resource-provisioning-52](./resources/understanding-resource-provisioning-52.png)

**This graph is exactly what Terraform builds internally** to decide creation and destruction order, and what can safely run in parallel. It's genuinely useful for understanding an unfamiliar codebase.

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

Remember: Terraform reads **every `.tf` file in the directory** and concatenates them. Filenames are for humans.

That's the first VM mini-project complete. Tear it down before we move on:

*(Run from `~/terraform-training/05-virtual-machines`)*
```bash
terraform destroy
# type: yes
```

<br>
<br>

### 14:30–15:00 — Why Local State Doesn't Scale
*(Talk + discussion)*

One weakness remains in everything you've built. Your `terraform.tfstate` file is sitting on your laptop.

**ASK** <br>
Two engineers are working on the same infrastructure. Each has their own copy of the repo. Where's the state file, and what goes wrong? <br>
**ANSWER** <br>
Each has their **own local state file**, which means each has a different idea of what exists. Engineer A creates a VM; A's state knows about it, B's doesn't. B runs `apply` and Terraform — seeing no record of that VM — tries to create it again. You get duplicates, or errors, or worse: B's apply destroys something A created, because B's state says it shouldn't exist.

**ASK** <br>
So we just commit the state file to Git and share it that way? <br>
**ANSWER** <br>
No — two reasons. **Security**: state contains attribute values unencrypted, including secrets. **Reliability**: even ignoring security, you cannot guarantee everyone always adds, commits and pushes state at exactly the right moment. Someone forgets to push, someone else pulls a stale version, and you're back to conflicting views of reality — now with merge conflicts in a machine-generated JSON file nobody can resolve by hand.

**The answer is a remote backend** — state stored centrally in cloud storage, which every engineer and every pipeline reads from and writes to.

#### Locking

There's a second problem a remote backend solves. Two engineers run `terraform apply` at the same moment. Both read state, both compute a plan from it, both start making changes. Neither knows about the other, and state ends up corrupted — caught between two conflicting realities.

The fix is **locking**:
1. **Lock** the state before applying, so nobody else can start
2. Make the changes and update state
3. **Unlock** it

**NOTE FOR TRAINERS** <br>
Another good comparison with the AWS material. There, students had to stand up a whole extra resource — a **DynamoDB table** — purely to hold locks alongside the S3 bucket. <br>
Azure's `azurerm` backend needs no equivalent: locking is handled automatically with a **blob lease** on the state file itself, a native feature of Azure Blob Storage. One fewer resource to create, pay for and maintain. A genuine simplification worth calling out. <br>
**END OF NOTE**

An Azure Storage Account gives us everything we need:
- **Locking** — automatic, via blob lease
- **Encryption at rest** — on by default, no extra resource required
- **Versioning** — we can turn on blob versioning, so a corrupted state can be rolled back
- **Access control** — RBAC, so only the right identities can read it

That's what we'll build in the capstone.

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>
### 15:15–16:45 — Capstone: Load-Balanced VMs & a Remote Backend
*(Exercise — 1 hour 30 minutes)*

Two connected pieces of work. **Part A** scales your single VM into a load-balanced fleet of three, one per availability zone. **Part B** moves state off your laptop into Azure Blob Storage.

Work individually or in pairs. Get Part A running and verified before starting Part B.

---

#### Part A1 (≈20 min) — Set up the project and multiply the network resources

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

Now rename the VM resource to a plural, because it'll represent many:

**06-vms-with-lb/main.tf**
```tf
# NEW CONFIG — plural name to reflect many resources
resource "azurerm_linux_virtual_machine" "http_servers" {
  [ . . . ]
}
```

Each VM needs **its own NIC** and **its own public IP**, so those need to iterate too. We'll loop them over the subnets map we already have:

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

This is the pattern that makes `for_each` powerful, so slow down here:

**`for_each = azurerm_subnet.public_subnets`** — we're iterating over **another resource** that was itself created with `for_each`. Because that resource is a map keyed by `"1"`, `"2"`, `"3"`, iterating it gives us the same keys. So:
- `each.key` is `"1"`, `"2"`, `"3"` — used in names
- `each.value` is the whole **subnet object**, so `each.value.id` is that subnet's ID

**`azurerm_public_ip.http_server_pips[each.key].id`** — reach into the *matching* public IP by key. Because every collection shares the same keys, everything lines up: NIC `"2"` gets public IP `"2"` in subnet `"2"`.

**ASK** <br>
Why is keying everything consistently like this so much safer than three sets of numbered resources? <br>
**ANSWER** <br>
Because the key is the identity. Remove subnet `"2"` from the map and Terraform removes exactly that subnet, its NIC, its public IP and its VM — leaving `"1"` and `"3"` untouched. With positional indexes everything after the removal would shift and get rebuilt. It's yesterday's list-versus-set lesson, now applied to real infrastructure where "rebuilt" means real downtime.

Add an output so we can see the addresses:

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

That's a **`for` expression** — HCL's comprehension syntax, similar to a JavaScript `.map()`. Read it as: "for each key `k` and value `pip` in the collection, produce an entry mapping `k` to `pip.ip_address`". The `{ }` means we're building a map; `[ ]` would build a list.

---

#### Part A2 (≈20 min) — Three VMs

**ASK** <br>
What's the keyword for looping in HCL? <br>
**ANSWER** <br>
`for_each`

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

  # NEW CONFIG
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

Note the message now includes `${each.key}` — so each server identifies itself. That's how we'll prove the load balancer is actually distributing traffic.

*(Run from `~/terraform-training/06-vms-with-lb`)*
```bash
terraform init
terraform apply
```
![understanding-resource-provisioning-53](./resources/understanding-resource-provisioning-53.png)

Three HTTP servers.

- Type: `yes`

There's a lot of output — three VMs, each being SSH'd into and configured. Terraform does this **in parallel** where the graph allows, which is why the output interleaves.

*(In your browser)* — paste one of the output public IPs.

**NOTE FOR TRAINERS** <br>
Watch the cost here. A single B1s is free-tier for 12 months, but **three at once burns that allowance three times as fast**, and the Standard-SKU public IPs and load balancer are chargeable from the first minute. Keep runtime short and make sure everyone destroys at the end. This is a good live example of the cost-consciousness point from the Azure Fundamentals session. <br>
**END OF NOTE**

---

#### Part A3 (≈20 min) — The Load Balancer

A load balancer needs **its own NSG** — we don't want SSH open on the public entry point to our whole application.

**network-security-group.tf**
```tf
# HTTP_SERVER NSG ABOVE
[ . . . ]

# LOAD BALANCER CONFIG
# NEW CONFIG
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
  # REFERENCES NEW NAME
  network_security_group_name = azurerm_network_security_group.lb_nsg.name
}
```

**ASK** <br>
Why does the load balancer's NSG have an HTTP rule but no SSH rule? <br>
**ANSWER** <br>
**Least privilege.** The load balancer's job is to accept web traffic and pass it on — nobody should ever SSH into it. Every port you open is attack surface, so you open only what's needed. It's the same principle as scoping a Service Principal to one resource group rather than a whole subscription.

Again no explicit outbound rule — Azure's default `AllowInternetOutBound` covers it.

Now the load balancer itself. **Unlike AWS's all-in-one Classic Load Balancer, Azure spreads this across several linked resources:** a frontend Public IP, the Load Balancer, a Backend Address Pool, a Health Probe, and a Load Balancing Rule.

Worth naming the options before we build:

| Option | What it's for |
|---|---|
| **Basic SKU** | The oldest tier, comparable in spirit to a Classic ELB. Being retired |
| **Standard SKU** | Zone-redundant, higher scale, secure by default. The modern choice — what we'll use |
| **Application Gateway** | Azure's ALB equivalent. HTTP(S) content-based routing, path-based rules, WebSockets |
| **Azure Front Door** | Global routing and caching, for worldwide content delivery at very high volume |

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
```

**`frontend_ip_configuration`** is the **front door** — the single address users hit. Note its `name` (`"lb-frontend"`); the rule refers back to it by that name shortly.

Now the **backend pool** — where the traffic goes:

**main.tf**
```tf
# NEW CONFIG
resource "azurerm_lb_backend_address_pool" "lb_backend_pool" {
  loadbalancer_id = azurerm_lb.lb.id
  name            = "http-servers-pool"
}
```

The pool starts **empty**. We add members by associating each NIC — another join resource, iterated:

**main.tf**
```tf
# NEW CONFIG
resource "azurerm_network_interface_backend_address_pool_association" "http_server_nics_pool" {
  for_each                = azurerm_network_interface.http_server_nics
  network_interface_id    = each.value.id
  ip_configuration_name   = "internal"
  backend_address_pool_id = azurerm_lb_backend_address_pool.lb_backend_pool.id
}
```

Note `ip_configuration_name = "internal"` — matching the name we gave the `ip_configuration` block inside the NIC. A NIC can have several IP configurations, so we say which one joins the pool.

Now the **health probe**:

**main.tf**
```tf
# NEW CONFIG
resource "azurerm_lb_probe" "http_probe" {
  loadbalancer_id = azurerm_lb.lb.id
  name            = "http-probe"
  port            = 80
  protocol        = "Http"
  request_path    = "/"
}
```

**ASK** <br>
Why does a load balancer need a health probe at all — why not just send traffic to all three servers? <br>
**ANSWER** <br>
Because a server can be *running* while being *broken* — Apache crashed, the disk filled, the app is wedged. The probe repeatedly requests `/`; if a server stops responding correctly it's taken out of rotation automatically, and put back when it recovers. Without it the load balancer would cheerfully send a third of your users to a dead machine. This is what turns three servers into genuine resilience rather than just three servers.

Finally the **rule** tying it together:

**main.tf**
```tf
# NEW CONFIG
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

Read it as a sentence: *traffic arriving on frontend port 80, at the frontend named `lb-frontend`, should be sent to backend port 80 on members of this pool, provided the probe says they're healthy.*

Note `frontend_port` and `backend_port` are separate — they don't have to match. A common real pattern is 443 in, 80 out, terminating TLS at the load balancer.

Add an output:

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
Don't panic if the first request fails — the load balancer and its health probe take a few minutes to settle, and until the probe has confirmed a server is healthy it won't route to it. <br>
**END OF NOTE**

Keep refreshing. You'll see it alternate between servers 1, 2 and 3 — the message you wrote with `${each.key}` telling you which one you hit.

![understanding-resource-provisioning-59](./resources/understanding-resource-provisioning-59.png)
![understanding-resource-provisioning-60](./resources/understanding-resource-provisioning-60.png)
![understanding-resource-provisioning-61](./resources/understanding-resource-provisioning-61.png)

Have a look at the graph now:

*(Run from `~/terraform-training/06-vms-with-lb`)*
```bash
terraform graph
```

Paste it into [Graphviz Online](https://dreampuf.github.io/GraphvizOnline/) — considerably more interesting than this morning's.

![understanding-resource-provisioning-63](./resources/understanding-resource-provisioning-63.png)

**Destroy before moving on** — this is the expensive part of the day:

*(Run from `~/terraform-training/06-vms-with-lb`)*
```bash
terraform destroy
# type: yes
```

**Solution — Part A**

The complete `main.tf` for `06-vms-with-lb`:

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

#### Part B1 (≈15 min) — Set up the backend-state project

Now let's fix the state problem.

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

resource "azuread_user" "my_azuread_user" {
  user_principal_name = "my_iam_user_def@jbloggsdevops.onmicrosoft.com"
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
ls -la          # note terraform.tfstate is here, locally
```

**Why two sub-projects?** `backend-state` creates the storage; `users` consumes it. They must be separate, because you can't store a project's state in a storage account that the same project is responsible for creating — a chicken-and-egg problem. And one backend storage account will serve *many* projects: users, VMs, load balancers, all of them.

---

#### Part B2 (≈20 min) — Build the storage account

*(Run from `~/terraform-training/07-backend-state/backend-state`)*
```bash
touch main.tf
```

**backend-state/main.tf**
```tf
# NEW CONFIG
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

Remember: no hyphens or underscores, globally unique. We're using `stdevappsbackend<yourname>` — dev environment, all apps, plus a name for uniqueness.

Three things this gives us:
- **`versioning_enabled = true`** — every state write keeps the previous version, so a corrupted state can be rolled back
- **Encryption at rest** — on by default. No extra resource needed, unlike AWS
- **`container_access_type = "private"`** — no anonymous access. Absolutely not optional for a file containing secrets

*(Run from `~/terraform-training/07-backend-state/backend-state`)*
```bash
terraform init
terraform validate
terraform apply
# type: yes
```

---

#### Part B3 (≈15 min) — Migrate the users project

Now point `users` at that storage. Add a **`backend`** block inside the `terraform { }` block:

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

- `backend "azurerm"` — the backend *type*. Others include `s3`, `gcs`, `consul`, and `remote` for Terraform Cloud
- `resource_group_name` / `storage_account_name` / `container_name` — where the storage lives
- `key` — the **blob name** the state is stored under. This is what separates one project's state from another's in the same container

**A note on the key.** It should encode enough to be unambiguous:
- **application** — what links these directories together (`07-backend-state`)
- **environment** — `dev`
- **project** — the sub-project (`users`)

**NOTE FOR TRAINERS** <br>
Values in a `backend` block **cannot be interpolated** — no `var.` references, no expressions. It's a genuine limitation that surprises people, and it exists because the backend must be resolved before Terraform has evaluated any variables. Teams handle it with **partial configuration**: leave values out of the block and supply them at init time with `terraform init -backend-config=dev.hcl`. Worth mentioning as the answer to "but how do we vary this per environment?" <br>
**END OF NOTE**

*(Run from `~/terraform-training/07-backend-state/users`)*
```bash
terraform init
```

![understanding-resource-provisioning-65](./resources/understanding-resource-provisioning-65.png)

Terraform detects the backend change and asks whether to **copy the existing local state** to the new backend.

- Type: `yes`

![understanding-resource-provisioning-66](./resources/understanding-resource-provisioning-66.png)

If it errors, use `terraform init -reconfigure`.

Now delete the local copies — they're no longer authoritative:

*(Run from `~/terraform-training/07-backend-state/users`)*
```bash
rm terraform.tfstate terraform.tfstate.backup
ls -la
```

*(In the Azure Portal — **Storage accounts → stdevappsbackend... → Containers → tfstate**)*

![understanding-resource-provisioning-67](./resources/understanding-resource-provisioning-67.png)

There's a blob named exactly what you set as `key`. Open it — it's your Known State.

![understanding-resource-provisioning-68](./resources/understanding-resource-provisioning-68.png)
![understanding-resource-provisioning-69](./resources/understanding-resource-provisioning-69.png)

*(Run from `~/terraform-training/07-backend-state/users`)*
```bash
terraform plan
```

It works with no local state file at all — Terraform fetched it from Azure.

#### A better key structure

A flat filename works, but a **hierarchical** key using forward slashes gives Azure pseudo-folders and scales far better.

First destroy, because we're about to change where state lives — otherwise you'd end up with a fresh empty state trying to create a user that already exists:

*(Run from `~/terraform-training/07-backend-state/users`)*
```bash
terraform destroy
# type: yes
```

**users/main.tf**
```tf
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-backend-state-jbloggs-devops"
    storage_account_name = "stdevappsbackendjbloggs"
    container_name       = "tfstate"
    # key = "environment / application / project / chosen state"
    # NEW CONFIG
    key = "dev/07-backend-state/users/backend-state.tfstate"
  }
}
```

*(Run from `~/terraform-training/07-backend-state/users`)*
```bash
terraform init -reconfigure
terraform apply
# type: yes
```

*(In the Azure Portal — the `tfstate` container)* — refresh until you see the `dev/` prefix. It may take a moment.

**ASK** <br>
Blob storage is flat — there are no real folders. So what does putting slashes in the key actually achieve? <br>
**ANSWER** <br>
Azure's tooling *displays* slash-separated prefixes as a folder hierarchy, so it's browsable. More importantly it gives you a consistent, predictable convention: any engineer can work out where a given project's state lives, and you can grant RBAC access by prefix — letting a team read `dev/` state but not `prod/`. The structure carries meaning even though the storage is flat.

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
  user_principal_name = "my_iam_user_def@jbloggsdevops.onmicrosoft.com"
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
terraform destroy       # type: yes, before changing the key
# ...update key to the hierarchical form...
terraform init -reconfigure
terraform apply         # type: yes
```

<br>
<br>

### 16:45–17:00 — Wrap-up, Destroy, & Q&A

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

**Destroy `users` before `backend-state`.** If you delete the storage account first, the `users` project loses its state entirely and has no idea it owns an Azure AD user — leaving an orphaned resource Terraform can no longer manage.

**ASK** <br>
That ordering problem is interesting. `users` depends on `backend-state`, but Terraform has no idea — there's no reference between them. What does that tell us? <br>
**ANSWER** <br>
That the dependency graph only works **within a single project**. Across projects, ordering is a human responsibility. That's the trade-off for splitting infrastructure by lifecycle: smaller blast radius and faster plans, but you have to know and document the ordering yourself. Teams handle this with naming conventions, README files, and pipelines that run projects in a defined order.

#### What you built today

You went from a single storage account to a networked, load-balanced, self-configuring fleet — and moved your state somewhere a team could actually share.

- **A network from scratch** — VNet, three subnets across availability zones, NSG with explicit priority-ordered rules
- **A VM that couldn't exist alone** — public IP, NIC, NSG association, SSH key, all wired by the dependency graph
- **Configuration at creation** — `remote-exec` over SSH installing and starting Apache
- **Immutability** — why you replace servers rather than patch them
- **`data` sources** — reading things Terraform doesn't own
- **`terraform graph`** — seeing the dependency graph you'd been building implicitly all along
- **Scaling with `for_each`** — one block, three servers, keyed consistently across every related resource
- **A load balancer** — frontend, backend pool, health probe, rule
- **A remote backend** — state in Azure Blob Storage, with locking via blob lease and encryption by default

**ASK** <br>
Think back to the Jenkins session. What single thing that we did today makes it *possible* to run Terraform from a pipeline at all? <br>
**ANSWER** <br>
**The remote backend.** A Jenkins agent is a disposable container — it has no local state file, and anything it writes disappears. With state in Azure Blob Storage, the pipeline reads the same authoritative state everyone else uses, takes a lock so a concurrent run can't corrupt it, and writes back. Without it, automated Terraform is impossible. That's why this is the last thing we did today and the first thing the pipeline session will rely on.

Where this goes next:
- **Today** — real infrastructure, provisioned and configured, with shared state
- **Next** — **modules**, packaging this into reusable components, and running Terraform *in a pipeline* with a plan/approve/apply gate
- **Kubernetes** — running containers on infrastructure like this
- **Integration** — a Git push flowing through test, build, `terraform plan`, human review, `terraform apply`, deploy

**Q&A** — take remaining questions. And have everyone confirm `az resource list -o table` is empty before they close their laptop.

<br>
<br>

### Exercise (take-home / reinforcement)

Before the next session, individually or in pairs:

1. Rebuild the `05-virtual-machines` project **from memory** — network, NSG, public IP, NIC, VM. Rebuilding from scratch is the fastest way to make it stick.
2. Change the NSG so SSH is only allowed from **your own IP address** rather than `0.0.0.0/0`. (Find it with `curl ifconfig.me`.) Confirm you can still connect.
3. Add an `environment` variable defaulting to `dev`, and use it in every resource name and as a tag.
4. Add a second load balancing rule for **HTTPS on port 443**, and a matching NSG rule. You won't have a certificate, so just confirm the plan is correct — don't chase a working TLS setup.
5. Move the `05-virtual-machines` state into the remote backend you built, under a sensible hierarchical key.
6. **(Stretch)** Replace the `remote-exec` provisioner with **`custom_data`** on the VM — Azure's cloud-init mechanism. Work out why it's generally preferred over provisioners.
7. **(Stretch)** Read about `prevent_destroy` in a `lifecycle` block and explain when you'd want it on your backend storage account.

**Solution** *(for the guided ones — 2, 3, 6)*

**Take-home 2** — restricting SSH to your own IP:

*(Run from anywhere)*
```bash
curl ifconfig.me      # prints your public IP, e.g. 81.134.22.7
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
  # /32 means exactly this one address — no range
  source_address_prefix       = "81.134.22.7/32"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.vm_resource_group.name
  network_security_group_name = azurerm_network_security_group.http_server_nsg.name
}
```

`/32` fixes all 32 bits, so the range contains exactly one address. Note this breaks if your IP changes — which is why real setups use a VPN range or Azure Bastion.

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

resource "azurerm_linux_virtual_machine" "http_server" {
  name = "${var.environment}-http-server"
  # ...

  tags = {
    environment = var.environment
    name        = "${var.environment}-http-server"
  }
}
```

Now `terraform apply -var="environment=test"` gives you a completely separate, clearly-labelled set of resources.

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

**Why it's preferred:** the provisioner runs from *your machine*, needs SSH reachable, needs the private key present, and fails the whole apply if the network hiccups. `custom_data` hands the script to Azure, which runs it **on the VM at first boot** via cloud-init — no SSH, no key, no inbound port 22, and it works identically from a Jenkins agent that has no key at all. HashiCorp themselves describe provisioners as a last resort. Note `base64encode` — Azure requires the payload base64-encoded, and `<<-EOF` is a **heredoc** (the `-` allows the closing marker to be indented), which is exactly the syntax from the bash session.

<br>
<br>

### Conclusion

- **Inform** students this marks the end of the resource provisioning session
- **Reinforce** the two headline ideas: **infrastructure has dependencies**, and Terraform's graph manages them for you in both directions; and **state belongs somewhere shared**, or a team can't work and a pipeline can't run
- **Remind** everyone to check `az resource list -o table` before closing their laptop — VMs, public IPs and load balancers all cost money
- **Preview** the next session: **modules** for reusable infrastructure, and running Terraform inside a Jenkins pipeline with a plan/approve/apply gate — which the remote backend they built today is what makes possible
- **Direct** students to the take-home exercises and to the [azurerm provider docs](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs), particularly the Virtual Machine and Load Balancer pages

---

[Back](./README.md)

---
