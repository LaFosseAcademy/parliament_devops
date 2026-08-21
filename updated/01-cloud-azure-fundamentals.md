# Azure Fundamentals — Trainer Script

An introduction to DevOps as a way of working, and to the core building blocks of Azure that everything else this course covers will sit on top of. This is written as a script — read it out, adapt where it feels natural, but the intent is that you could deliver this cold from the page.

## Organisation

### Audience

People **new to both DevOps and Cloud**. Assume no prior Azure experience and no assumption of comfort with the command line — some students may have neither. Check the room early (see the opening ASK) and adjust pace accordingly.

### Duration & schedule

Full day, **09:00–17:00**, with a one-hour lunch and three 15-minute breaks.

| Time | Section |
|---|---|
| 09:00–09:15 | Welcome & introductions |
| 09:15–09:45 | What DevOps looks like to an engineer |
| 09:45–10:15 | Cloud computing & service models |
| **10:15–10:30** | **Break** |
| 10:30–11:15 | Setting up your Azure account |
| 11:15–12:00 | The Azure resource hierarchy |
| **12:00–12:15** | **Break** |
| 12:15–13:00 | Portal tour, resource groups & your first resources |
| **13:00–14:00** | **Lunch** |
| 14:00–14:30 | Regions and availability |
| 14:30–15:00 | Networking basics |
| **15:00–15:15** | **Break** |
| 15:15–16:00 | Identity & access: Azure AD, RBAC, Service Principals |
| 16:00–16:30 | The problem with ClickOps — why we're doing all this |
| 16:30–17:00 | Roadmap, wrap-up & Q&A |

### Set-up

#### For Trainers

**For the slidee presentation** <br>
**Open** the [Slidee](https://github.com/mklilley/slidee) presentation:

- Run `npx slidee` from within the `curriculum` folder to see the available slide decks (those with extension `.slides.md`)
- Click on `See your presentations`
- Open `Weeks X Y > CDO > Azure Fundamentals`
- Hit `s` to open the presenter notes
- Resource to be kept open on a tab throughout lecture to refer to

#### For Students
- **Direct** students to the starter code for this module
  - [Student Facing Repo](https://github.com/LaFosseAcademy/cloud-devops-student-repos/tree/main)
- **Use**
  - **/azure-fundamentals/starter-code**
- **Make sure**
  - Students bring a laptop and, ideally, a payment card for Azure free-trial verification (see Section 4 — it is not charged unless they explicitly upgrade)
  - Students have the [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli) installed, or are able to install it during the account setup section
  - No Terraform, Jenkins, or Kubernetes needed today — those are separate sessions, previewed at the end

## Learning objectives

- **Understand** DevOps as a mindset and set of daily habits, not just a job title or a toolchain
- **Understand** the core Azure resource hierarchy: tenants, subscriptions, resource groups, and resources
- **Set up** a working Azure account and navigate the Portal and CLI
- **Understand** the basics of Azure networking and identity/access (RBAC, Service Principals)
- **Recognise** the limitations of manually managing infrastructure through the Portal ("ClickOps")
- **Leave** with a clear picture of how today connects to the sessions that follow: Linux & automation, Jenkins & pipelines, Terraform & IaC on Azure, Kubernetes, and finally integrating all of it together

<br>

## Sequence

### 09:00–09:15 — Welcome & Introductions


Morning everyone, welcome. Today's the first of a run of sessions that'll take you from the basics to understanding more about cloud or DevOps, through to actually being able to build, automate, and ship infrastructure the way a real engineering team does it.

Today specifically is about two things. First half is: what actually *is* DevOps, and what does it change about how you work day to day. Second half is: getting properly comfortable in Azure — the cloud platform PDS uses — because every one of our future sessions is going to assume you already know your way around it.

We've got a full day. Lunch is at 1, an hour, and we'll take a proper 15-minute break roughly every 90 minutes to two hours — flag it if you need one sooner, there's no need to wait.

<br>
<br>

### 09:15–09:45 — What DevOps Looks Like to an Engineer


Let's start with DevOps itself, before we touch a single Azure resource — because the tools we'll spend the rest of today and the coming sessions on, only make sense once you understand the problem they exist to solve.

Historically, software teams were split into two camps. **'Dev'** — the people who wrote application code — and **'Ops'** — the people who kept servers running, deployed changes, and were the ones who got paged at 3am when something broke. These two teams had, in practice, opposite incentives. Dev were rewarded for shipping features fast. Ops were rewarded for stability, which in practice usually meant resisting change, because change is what breaks things. That tension is a big part of why deployments used to be huge, dreaded, quarterly events — and why they went wrong so often when they did happen.

DevOps grew out of that dysfunction. There's a mantra that captures the core idea, and we'll come back to it more than once today:

> **'You build it, you run it.'**

The same people — or at least the same team — who write a piece of software are also accountable for deploying it, monitoring it, and fixing it at 3am if it breaks.

**ASK** <br>
What do we think changes about how someone writes code, if they know *they* are the one who'll get the phone call if it breaks in production? <br>
**ANSWER** <br>
They write more defensively. They add their own logging and monitoring instead of assuming someone else will. They test more thoroughly. They design for failure. Because the consequence of a bad decision now lands directly on them, not on a separate team down the corridor.

That one incentive shift explains a surprising amount of DevOps culture and tooling.

One common way people summarise the pillars of DevOps is the acronym **CALMS**:

- **Culture** — breaking down the wall between Dev and Ops. Shared responsibility instead of finger-pointing when something breaks
- **Automation** — anything a human does repeatedly and manually is a future source of mistakes. If you're doing it by hand every week, it should probably be a script
- **Lean** — small, frequent changes are safer than big, infrequent ones. A ten-line change is easy to review and easy to roll back; a thousand-line change that's sat in a branch for three months is terrifying
- **Measurement** — you can't improve what you don't measure. Deployment frequency, time to recover from an incident, error rates
- **Sharing** — knowledge shouldn't live in one person's head. Document it, pair programming, review it

**ASK** <br>
Which of those five do you think is hardest for an established, traditional company to actually change? <br>
**ANSWER** <br>
Usually Culture. Buying a new tool or installing a pipeline is comparatively easy. Changing how teams are structured, how success is measured, and how blame is (or isn't) handled after an incident is a much slower, harder organisational shift — and it's the one that makes all the others stick.

So practically — what does this actually look like in an engineer's daily habits? A handful of things we'll keep coming back to across this whole course:

1. **Everything as code, everything in version control.** Not just application code which is the only thing you've seen so far — I mean, infrastructure, pipeline configuration, even documentation. If it's not in Git, it effectively doesn't exist to the team, because it can't be reviewed, reverted, or reliably reproduced.
2. **Small, frequent changes over big, risky ones.**
3. **Automate the boring, repeatable stuff.** If you find yourself doing the same five manual steps every Friday, that's a script — and eventually a pipeline — waiting to be written.
4. **Treat failure as information, not blame.** Things will break. The habit isn't 'never fail', it's 'fail safely, recover quickly, and write up what happened' — often called a blameless postmortem.
5. **Monitor and measure rather than guess.** A good engineer can usually tell you how their system is performing right now, not just when something's already on fire.

**ASK** <br>
Can anyone think of a task — in any job or life, not just tech — that someone would do the exact same way, by hand, every single week? <br>
**ANSWER** <br>
- We create Zoom channels for every cohort which runs, could we save time automating it?
- I've created a script to clone all your repo's once you've sat a debug assignment, it saves me time from doing it manually

This mindset is genuinely what the rest of this course is going to build hands-on skill around. Over the coming sessions we'll cover:

- **Linux & automation** — the fundamental skill of working comfortably on a server and a command line, and scripting the boring stuff away
- **Jenkins & pipelines** — how 'small, frequent changes' actually get built, tested, and deployed automatically, rather than by someone clicking a deploy button
- **Terraform & Infrastructure as Code with Azure** — 'everything as code' applied directly to the infrastructure we're about to spend the rest of today exploring by hand
- **Kubernetes** — how we run and scale applications reliably once they're built and deployed
- **Integrating it all together** — a final session pulling every one of those threads into one real pipeline: a code change flowing through to running infrastructure, automatically, safely, and reviewably

Everything today is building the vocabulary and the map for those five sessions. By the end of today, when we say 'resource group' or 'Service Principal' in the Terraform session, it shouldn't be the first time you've heard it."

<br>
<br>

### 09:45–10:15 — Cloud Computing & Service Models


So — cloud computing. In one sentence: it's renting compute, storage, and networking in someone else's data centre, and paying for what you use, instead of buying and running your own physical servers.

**ASK** <br>
Why do we think so many companies choose this over running their own hardware? <br>
**ANSWER** <br>
- No large upfront cost for buying servers
- No need to employ people to physically rack, cable, and power hardware
- Easy to scale — if you start with 1,000 users and grow to 100,000, it's straightforward to provision more from the cloud; the reverse is also true if you need to scale down

There are several models for accessing cloud resources. Everyone here has probably used something like iCloud, Google Drive, or Dropbox — those are cloud services, but really they only offer storage and networking. The three models I want you to actually know are:

- **SaaS** — Software as a Service
- **PaaS** — Platform as a Service
- **IaaS** — Infrastructure as a Service

**Software as a Service** is the simplest — tools like Gmail, Salesforce, Zoom. Complete, finished applications you just use.

**Platform as a Service** is a platform for developers to build and run applications on — Netlify, Heroku, Render. It requires more technical understanding, and gives some ability to configure the underlying environment, but the provider still manages the servers.

**Infrastructure as a Service** takes us closest to the bare metal. You're choosing what kind of computer you want — how powerful, what storage, what networking — and it's your job to manage the operating system onwards. Full control, but also full responsibility.

Quick way to remember it:

- **IaaS** → *'Give me a computer.'* (Amazon EC2, Azure VM, Google Compute Engine — this is what we're focusing on for this Deep Dive)
- **PaaS** → *'Give me somewhere to run my code.'* (App Engine, Heroku, Azure App Service)
- **SaaS** → *'Give me the finished software.'* (Slack, Spotify, Zoom, Microsoft 365)

Let's play a quick game to bed this in — I'll name something, you tell me IaaS, PaaS, SaaS, or None."

| Example | Answer | Why |
|---|---|---|
| Amazon EC2 | IaaS | You control the OS, software and configuration; Amazon manages the physical infrastructure |
| Slack | SaaS | You use a finished application through a UI; you manage nothing underneath |
| Google App Engine | PaaS | You deploy your app; Google manages servers, OS and runtime |
| Toyota | None | Manufactures cars, not cloud services |
| Azure Virtual Machines | IaaS | You control the VM, OS and applications; Microsoft manages the physical infrastructure |
| Microsoft 365 | SaaS | Finished applications; Microsoft manages everything beneath them |
| Heroku | PaaS | You provide the code; Heroku handles infrastructure and runtime |
| McDonald's | None | Food and restaurant services |
| Google Compute Engine | IaaS | You control the VM; Google manages the hardware |
| Spotify | SaaS | A finished, consumed application |
| Azure App Service | PaaS | Deploy your app without managing servers or OS |
| Nike | None | Sells physical products |
| Dropbox | SaaS | A finished cloud application |
| DigitalOcean Droplets | IaaS | You get a virtual server and manage the OS |
| AWS Elastic Beanstalk | PaaS | You deploy; AWS manages most infrastructure |
| Coca-Cola | None | Produces beverages |
| Zoom | SaaS | Finished video-conferencing software |
| Salesforce Platform | PaaS | Build on a managed platform |
| Google Workspace | SaaS | Finished applications, consumed through a browser |
| Disney | None | Entertainment, media and theme parks |

And out of curiosity — what if instead of Disney I'd said Disney Plus? That one's genuinely fuzzy — you could reasonably call it SaaS, or just a cloud-based streaming/subscription service. Not everything falls neatly into one bucket, and that's fine.

**ASK** <br>
Which three providers dominate the cloud computing market? <br>
**ANSWER** <br>
AWS, Microsoft Azure, and Google Cloud Platform (GCP).

We're focusing on **Azure** because it's what PDS use, and because it's so deeply integrated with the Microsoft ecosystem that a lot of enterprises already run on — things like **Active Directory** (essentially a company's digital phonebook of user credentials), **Office 365**, and **Windows Server** a specific OS for servers. All of it plugs directly into Azure."

<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>

### 10:30–11:15 — Setting Up Your Azure Account

Right — let's actually get you into Azure. Everything from here on assumes you have your own account to click around in, so let's get that sorted properly rather than everyone just watching my screen.

Azure offers a **free trial**: normally around £150–£200 of credit valid for 30 days, plus a set of services that stay free for 12 months, plus a smaller set of 'always free' services. You'll need a Microsoft account (a free Outlook/Hotmail address works fine) and a payment card. The card is for identity verification only — you won't be charged unless you explicitly upgrade the subscription afterwards, and we'll make sure you know how to keep an eye on spend later today anyway.

If you already have an organisation-provided subscription, that's fine too, use that instead."

**HANDS ON (25 min)** <br>
Part A — sign up.
- Go to `azure.microsoft.com/free`
- Sign in with (or create) a Microsoft account
- Follow the verification steps — identity and card details
- Once through, you'll land in the Azure Portal at `portal.azure.com`

Part B — orient yourself in the Portal.
- The **search bar** at the top ("Search resources, services, and docs") is the fastest way to jump anywhere — much quicker than clicking through menus
- **Resource groups** — a filtered view of everything grouped by lifecycle
- **All resources** — every resource across every resource group you can see
- **Cost Management + Billing** — where the bill actually lives. Worth a proper look now, before your free trial credit vanishes unexpectedly later
- Search **Subscriptions** and find your own — note the **Subscription ID**, you'll need this repeatedly today and in future sessions

Part C — install and log in via the CLI.
- **Windows**: https://community.chocolatey.org/packages/azure-cli
- **Mac**: https://formulae.brew.sh/formula/azure-cli
- Run `az --version` to confirm it installed correctly
- Run `az login` — this opens a browser to authenticate, the same account as the Portal
- Run `az account show` — this confirms which subscription the CLI is currently pointed at
**END OF NOTE**

"A quick note on what just happened with the CLI: everything we do by clicking in the Portal, we can also do from the terminal. Under the hood, both are talking to the same **Azure REST API** — the Portal just wraps it in a UI. That equivalence is important, and it's the whole bridge into automation and Infrastructure as Code later in the course: anything clickable has a scriptable equivalent."

<br>
<br>

### 11:15–12:00 — The Azure Resource Hierarchy


Azure organises everything you own into a hierarchy. Get comfortable with this now, because it'll save a lot of confusion once we start writing Terraform in a few weeks.

*REFER TO RESOURCE 2 - SLIDEE* <br>
![azure-fundamentals-1](./slidee/images/azure_hierarchy.png)

From the top down:

* **Tenant** — your organisation's identity in **Azure AD** (Azure Active Directory). Almost always one tenant per organisation. Every user, group, and application registration lives inside a tenant.
* **Subscription** — a billing and access boundary *inside* a tenant. One tenant can hold multiple subscriptions — commonly one per environment (`Dev`, `Test`, `Prod`) or one per business unit.
* **Resource Group** — a logical container inside a subscription that groups related resources sharing a lifecycle. If you have an application that needs a Virtual Machine, a Database, and monitoring, they'd all live in the same Resource Group.
* **Resource** — the actual thing being managed: a Virtual Machine, a Storage Account, a Virtual Network, and so on.

**ASK** <br>
Why might an organisation want several subscriptions, rather than just using more resource groups inside one big subscription? <br>
**ANSWER** <br>
- Cleaner cost separation on the bill — easy to see if one department or environment has an unusually large bill, and different departments often have different budgets
- Harder access-control boundaries — someone with access to `Dev` doesn't automatically get anywhere near `Prod`
- Separate quotas/limits, and a smaller **blast radius** if something goes badly wrong. A `Test` environment might limit how long a VM can run, because tests should finish quickly and you don't want to burn budget accidentally — that would be a bad limit for `Prod`. And if you had a script that deletes resources and everything lived in one subscription, you could accidentally wipe out everything at once.

I'm signed in via my own email, and my subscription is called **Azure subscription 1**. It's got an ID, and if you scroll right, a **Parent management group** too — all default values.

- *In Azure Portal, go to 'Subscriptions'*

![azure-fundamentals-2](./slidee/images/azure_subscription_1.png)
![azure-fundamentals-2](./slidee/images/azure_subscription_2.png)

**HANDS ON (20 min)** <br>
Part A — the subscription. Explore your **Subscription** resource in the Portal: look at what information is available, then rename it to your name plus `lfa-deep-dive` — for me that'd be `emilesherrott-lfa-deep-dive`.

Part B — the tenant. Now go a level up. Search **Microsoft Entra ID** (the current name for Azure AD) in the Portal.
- Note your **Tenant ID** and **Primary domain** on the Overview page
- Click into **Users** and find your own account — this is the identity that's been authenticating you all morning
- Click into **Groups** — even if empty, this is where an organisation would put collections of users to manage access in bulk rather than one at a time

*You can also explore 'Devices'*

Compare the two: the tenant sits *above* the subscription and holds identity — who you are. The subscription holds billing and resources — what you're allowed to touch and spend. Keep that distinction in mind, we'll come back to it in the identity and access section this afternoon.


<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>

### 12:15–13:00 — Portal Tour, Resource Groups & Your First Resources

Let's put a real resource group and a real resource into that hierarchy we've just drawn.

When we started this session, I mentioned one of the benefits of cloud is no upfront cost. That benefit disappears fast if we're not careful about what we spend — if the solution to a problem at PDS costs £100 million, it's not a good solution. So being cost-conscious is part of the job from day one, not an afterthought.

Let's create something by hand, the way you might if you were just getting started and hadn't heard of Terraform yet."

* Search **Resource groups** > **Create**
* **Subscription**: your subscription
* **Resource group name**: `pds-deep-dive`
* **Region**: `UK South`
* **Review + create** > **Create**

![azure-fundamentals-2](./slidee/images/azure_resource_group_1.png)

"That took maybe 30 seconds. Quick, easy — and, as we'll get to this afternoon, exactly the kind of thing that becomes a real problem at scale.

Remember the diagram from earlier: **Tenant**, then **Subscription**, then **Resource Group** — now we can provision an actual **Resource**."

**HANDS ON (35 min)** <br>
Part A (10 min) — create the resource group above by hand, following the steps just shown.

Part B (15 min) — your first real resource.
* From inside `pds-deep-dive`, click **Create** and search for **Storage account**
* **Subscription**: your subscription <br> **Resource group**: `pds-deep-dive` <br> **Storage account name**: something unique, e.g. `<yourname>pdsdeepdive` <br> **Region**: `UK South` <br> **Primary service**: `Azure Blob` 
  * Azure Blob is like DropBox, files, images, videos etc..
  * Azure Files is like OneDrive, where may many can connect and exchange documents
  * In other, there's 'Queue' and 'Table'. Queue is for messages between application components and 'Table' is NoSQL, similar to JSON <br>

  **Performance**: Standard <br> **Redundancy**: leave as default
* **Review + create** > **Create**, and wait for deployment to finish
* Once deployed, explore **Containers**, **Access keys**, and **Networking** on it

Part C (10 min) — tagging and cost. Real organisations rarely track spend by resource group name alone — they use **tags**, key/value pairs attached to a resource.
- Go to `pds-deep-dive` > **Tags**. Add `environment` = `training` and `owner` = your name
- Click Apply
- Back in the Resource Group we can also create a budget specifically for the resource group
  - Inside Resource Group, `pds-deep-dive` > **Cost Manangement** > **Budgets**
    - Name: `pds-deep-dive-budget`
    - Budget Amount > Amount: 5
    - *Click Next*
    - Alert Conditions > Type: Actual Cost & 10
    - Add email
    - *Click create*

- So this budget is scoped specifically to the **Resource Group**

- If I went to **Cost Management + Billing** > **Budgets**. I could create a budget scoped to the tenant on the very top level.

- The way we created the budget on the resource group, we could do the same with our *Subscription* 

*Go to create a new budget on your subscription*

You'll notice I've got the ability to filter, to be as specfic or as broad as I want regarding the bugets I make. 

**ASK** <br>
Why might a large organisation insist on every resource being tagged with an `environment` and  value before it's allowed to be created? <br>
**ANSWER** <br>
Without consistent tags, the bill becomes an undifferentiated list of resources, with no way to answer "how much is Dev costing us" or "which team owns this" — tags make cost and ownership reporting possible once there are hundreds of resource groups, not just one.

<br>
<br>

*(Take a 1 hour lunch break here.)*

<br>
<br>

### 14:00–14:30 — Regions and Availability

Azure runs datacentres in **regions** all over the world — `uksouth`, `northeurope`, `eastus`, and so on. Choosing a region affects **latency** (how close your users are to the servers) and **data residency** 

**ASK** <br>
Speaking of which, why might a UK-based healthcare company specifically care which region their data lives in? <br>
**ANSWER** <br>
Regulatory requirements — UK GDPR, NHS data-handling rules — can require patient data to stay within the UK, ruling out other regions entirely regardless of cost or latency.

**ASK** <br>
Why might a company want resources in both **UK South** and **UK West**? <br>
**ANSWER** <br>
In case of a disaster affecting one region.

Some regions are designed as **paired** sets for disaster recovery — UK South is paired with UK West. Microsoft coordinates updates and prioritises recovery between paired regions. Within a single region, Azure also offers **Availability Zones** — physically separate datacentres, each with independent power and networking, so a failure in one zone doesn't take the others down.

My assumption is that anything PDS builds will need to stay within UK regions, for data integrity reasons."

**HANDS ON (15 min)** <br>
Start creating a **Storage account** (don't finish it) and open the **Region** dropdown on the Basics tab.
- Count roughly how many UK-specific options there are
- In a new tab, search "Azure regions map" and find where `uksouth` and `ukwest` actually sit
- Discuss with the person next to you: if PDS only had UK users, is there ever a good reason to pick a non-UK region anyway?


**ANSWERS** <br>
- Service isn't available in the UK
- Cost (could be cheaper elsewhere)
- Disaster recovery
- Performance for external dependency (if our application, is built in an API based in Swizterland, having our infrastructure in Switzerland may sometimes outweight the extra latency to UK users)

Cancel out of the wizard once you're done — we're not creating this one.


<br>
<br>

### 14:30–15:00 — Networking Basics

Quick grounding in Azure networking before we move on to identity — you'll hear these terms constantly once Terraform starts.

A **Virtual Network (VNet)** is your own private, isolated network inside Azure — think of it as the Azure equivalent of the network inside an office building. Resources inside the same VNet can talk to each other by default; anything outside it can't, unless you explicitly allow it.

A VNet is divided into **subnets** — smaller address ranges within it, often used to separate different tiers of an application, for example a subnet for web servers and a separate subnet for databases.

A **Network Security Group (NSG)** is a set of allow/deny rules — a basic firewall — controlling what traffic can reach a subnet or resource, based on things like port number and source IP.

**ASK** <br>
Why might we want database servers in a completely separate subnet from web servers, rather than everything in one big subnet? <br>
**ANSWER** <br>
It lets us apply different NSG rules per tier — for example, only allowing inbound traffic to the database subnet from the web subnet, and nothing at all from the public internet, so even if the web tier is compromised, the database isn't directly reachable.

We won't go deep into networking today — that's a topic in its own right — but you should leave recognising these three terms, because we'll be creating all three in Terraform before long."

**HANDS ON (15 min)** <br>
Inside `pds-deep-dive`:
* **Create** > search **Virtual network**
* **Name**: `pds-vnet` <br> **Region**: `UK South`
* On **Address space**, you'll notice it says **Starting Address** and `10.0.0.0` and a size of `/16`. <br> What that's showing is the range of private IP addresses this Virtual Network can assign to computers and resources created in the network
  * Underneath there's a default subnet which occupies a range within the virtual network. This could be for a computer which hosts our API
  * And we could then add a second subnet named `pds-subnet-data` with its own address range
  * *Create 'pds-subnet-data*

What you're seeing with the `/16` and `/24` is the **CIDR** notation. Essentially how large the network is. 

The bigger the number, the smaller the network. 

So with `/16` we're occupying the `10` and `0` from: `10.0.0.0`. Each number takes up 8 bytes of space, leaving the last two `0`'s to be assingable for resources. 

With `/24`, we're essentially saying the first three numbers are occupied and only the last number can be changed for resources in the subnet. 

It's not necessarily anything you need to obsess over but it gives an impression of how big a network is. 

We max out at 255 just because that's the largest number a 8 bytes can hold. 

* **Review + create** > **Create**

Essentially though we can create different subnets to isolate the type of resources in a network. 

Once created, click into `pds-vnet` > **Subnets** and confirm both are listed. Click into `pds-subnet-data` subnet > **Network security group** — note that none is attached by default; that's something we'll configure properly with Terraform later in the course.

But this is where we could say, for the computers in this subnet, only allow traffic from devices from an IP addresses associated with our other subnet. 


<br>
<br>

*(Take a 15 minute break here.)*

<br>
<br>

### 15:15–16:00 — Identity & Access: Azure AD, RBAC and Service Principals


We need a foundational understanding of identity before we can talk about automation properly.

**Azure AD** — 'Azure Active Directory' — is Azure's identity provider. It's where **users**, **groups**, and **application identities** live, and it sits *above* subscriptions in the hierarchy we drew this morning — one tenant can be linked to many subscriptions.

**ASK** <br>
So far, who or what has been authenticating to Azure in everything we've done today? <br>
**ANSWER** <br>
A human — you, via `az login` or the Portal sign-in.

That's fine for a person clicking around, but it doesn't work well for automation. That's where **Service Principals** come in — an identity in Azure AD that represents an **application or automated process**, rather than a human.

Let's create one. First, `az account show`, and note the value against `id` — that's your subscription ID.

```bash
az ad sp create-for-rbac --name "example-automation-identity" --role="Reader" --scopes="/subscriptions/<your-subscription-id>"
```

- `az` — the Azure CLI executable
- `ad` — Azure Active Directory (now called Microsoft Entra ID) — Microsoft's identity system
- `sp` — Service Principal, an identity for an application or automation rather than a human. Imagine a script that needs to authenticate — instead of it logging in as you, it uses its own Service Principal
- `create-for-rbac` — creates the Service Principal and configures RBAC for it
- We give it a name, a role — `Reader` here, you'll often see `Contributor` too — and a scope, where it's actually allowed to work

Running this returns an `appId`, `displayName`, `password`, and `tenant` — credentials a script, pipeline, or tool can authenticate with, entirely independent of any human's login.

**ASK** <br>
Why would we want automation to use its own separate identity, rather than running everything under a person's own login? <br>
**ANSWER** <br>
It doesn't break when that person is on holiday or leaves the company; it can be scoped down to only what it needs; it can be rotated or revoked without touching a human account; and it works headlessly, without anyone needing to be logged in — essential for anything running in a pipeline.

Access itself is controlled through **RBAC** — **Role-Based Access Control**. A **role** — `Owner`, `Contributor`, `Reader` — is assigned to an identity, human or Service Principal, **at a scope**: a Management Group, a Subscription, a Resource Group, or an individual Resource.

**ASK** <br>
If a script only ever needs to manage resources inside one resource group, should we grant its Service Principal `Contributor` across the whole subscription? <br>
**ANSWER** <br>
No — scope the role assignment down to just that resource group. This is the principle of **least privilege**: grant only the access actually needed, at the narrowest scope that still works. If those credentials leak, the blast radius is one resource group, not everything you own."

**HANDS ON (25 min)** <br>
Part A (10 min) — create the Service Principal above via the CLI, scoped to your subscription. Confirm it exists with `az ad sp list -o table`.

Part B (15 min) — RBAC in the Portal.
- Go to your subscription > **Access control (IAM)** > **Role assignments**. Find `example-automation-identity` and confirm the **Reader** role at subscription scope
- Now go into `pds-deep-dive` > **Access control (IAM)** > **Role assignments** — this view is scoped only to this resource group. Do you see `example-automation-identity` listed here directly, or only inherited from the subscription above?
- Click **Add > Add role assignment**, but stop before confirming — just explore the available roles and scope granularity

These roles define what someone can do, the main ones are:
- Owner: They can do everything, including giving permissions to others
- Contributor: They're able to create, change, delete resources, but can't grant permissions
- Reader: Can view all resorces but can't change them

On members I'll be able to select `+ Select members` and choose a person or the service principle I created earlier `example-automation-identity`

**ASK** <br>
If `example-automation-identity` has `Reader` at the subscription, and we look at the resource group's own IAM blade (blade meaning that section of access), do we expect to see it listed as an assignment made *at* that resource group? <br>
**ANSWER** <br>
No — RBAC is inherited downward. Granted once at the subscription, every resource group and resource beneath it inherits it automatically. That's exactly why scope matters: grant at the highest point that still respects least privilege, no higher, no lower than necessary.
**END OF NOTE**

<br>
<br>

### 16:00–16:30 — The Problem with "ClickOps"

*(Read as script)*

"Let's go back to the resource group we built by hand this morning, and really interrogate it.

**ASK** <br>
If I asked you right now to recreate `pds-deep-dive` — identically — in a brand new subscription, how would you do it? <br>
**ANSWER** <br>
Probably from memory, or by clicking back through and reading off the settings — slowly, with a real chance of getting something slightly wrong.

This pattern — manually clicking through a cloud console to build infrastructure — is sometimes called **'ClickOps'**, usually not as a compliment. Some of the problems with it:

* **Not repeatable** — there's no reliable way to do exactly the same thing twice
* **No audit trail** — the Portal shows *that* a resource was created and by whom, but not *why*
* **Environment drift** — Dev, Test, and Prod slowly diverge because someone made a 'quick fix' by hand in one and not the others
* **No review process** — nothing stops one person making a risky change directly, with nobody else checking first
* **Tribal knowledge** — the exact steps to build something correctly often only exist in one engineer's head

Let's not just take the 'no audit trail' point on faith."

**HANDS ON (10 min)** <br>
- Go to `pds-deep-dive` > **Activity log**
- You should see entries for creating the resource group, the storage account, the VNet, and adding tags
- Click into one entry and look at the **JSON** tab

**ASK** <br>
Looking at that log entry — can you tell *why* the storage account was created, what it was for, or whether anyone reviewed the decision beforehand? <br>
**ANSWER** <br>
No — the Activity Log tells us *what* happened, *who* did it, and *when*. It has no concept of intent or review. That gap — no conversation, no justification, no "someone checked this before it happened" — is exactly what Pull Requests give us once we move to Infrastructure as Code, which is what the Terraform session in a few weeks is all about.
**END OF NOTE**

"None of this means the Portal is bad — it's genuinely great for exploring, learning what a service looks like, and one-off investigation, which is exactly what we've used it for today. The problem is relying on it as the *primary* way real infrastructure gets built.

This is exactly the gap Infrastructure as Code tools like Terraform close: instead of clicking through a wizard, you write your desired infrastructure state in text files, and a tool reconciles what actually exists against what you've declared — changing only what's needed.

And because those files are just text, they belong in **version control**. That's the same 'you build it, you run it' thread from this morning: a proposed infrastructure change becomes a Pull Request, a teammate reviews the `terraform plan` output before anything happens, and only once it's approved and merged does a pipeline — authenticating as a Service Principal, not a person — actually apply it. Every change becomes a discrete, reviewable, reversible record instead of an invisible click. That whole flow is what the Jenkins and Terraform sessions are going to build, hands-on."

<br>
<br>

### 16:30–17:00 — Roadmap, Wrap-up & Q&A

*(Read as script)*

**Clean-up (10 min)** <br>
"Before anything else — let's tidy up, so nothing keeps quietly costing money after today."
- Delete the `pds-deep-dive` resource group (**Resource groups** > select it > **Delete resource group** > type the name to confirm)
- Confirm from the CLI: `az group list -o table` should no longer show it
- Double check nothing else is lingering: `az resource list -o table`

**ASK** <br>
We had to type the resource group's name to confirm deletion, and everything inside it disappeared at once. Why the extra confirmation step, and what does that cascading delete tell us about treating resource groups as a real unit? <br>
**ANSWER** <br>
Typing the name is deliberate friction, stopping an accidental click from destroying everything inside a group — and the cascading delete is exactly why grouping resources by shared lifecycle matters: anything you don't want destroyed together shouldn't share a resource group.

**Roadmap (15 min)** <br>
"So — where does this lead? Today was deliberately broad: DevOps as a mindset, and Azure as the platform we'll be building on. From here, the course goes deep, one layer at a time:

- **Linux & automation** — comfortable, confident use of the command line and scripting, so you're not fighting the terminal while trying to learn everything else at once
- **Jenkins & pipelines** — how code changes actually get built, tested, and deployed automatically, tying back to that 'small, frequent changes' habit from this morning
- **Terraform & Infrastructure as Code with Azure** — everything we clicked through by hand today, written declaratively instead, reviewed through Pull Requests, applied by a pipeline
- **Kubernetes** — running and scaling applications reliably once they're built and deployed
- **Integration** — a final session bringing every one of those pieces together into one real, working pipeline: a code change flowing through to running Azure infrastructure, automatically and safely

Everything you did today — resource groups, tagging, budgets, regions, VNets and subnets, RBAC and Service Principals — is going to reappear, by name, in every one of those sessions. That's deliberate. Today was building the map."

**Q&A (5 min)**

<br>
<br>

### Exercise (take-home / reinforcement)

Students, working individually or in pairs, before the next session:
1. In the Azure Portal, find and note down your own **Subscription ID** and **Tenant ID**
2. Create a resource group by hand in the Portal, then create a second one using `az group create` — compare how each felt
3. Create a storage account inside one of your resource groups, tag it with `environment` and `owner`, and confirm the tag shows up when filtering costs in Cost Management
4. Open the **Activity log** for that resource group and identify the entries for every action you took
5. Check **Access control (IAM)** on your subscription and identify which roles are currently assigned to your own account, and at what scope
6. In small groups, discuss: if your team's infrastructure was entirely built via ClickOps today, what's the single biggest risk you'd worry about? Be ready to share back
7. (Stretch) Look up what `az role assignment list --scope /subscriptions/<your-subscription-id>` returns for your own account, and identify what role you currently hold and at what scope
8. (Stretch) Create a second Service Principal with `Reader` scoped only to one specific resource group rather than the whole subscription, and confirm in the Portal's IAM blade that it doesn't appear on any other resource group
9. (Stretch) Create a Virtual Network with two subnets, and sketch out — on paper, no need to build it — what NSG rules you'd want on each if one held web servers and the other a database

<br>
<br>

### Conclusion

- **Inform** students this marks the end of the Azure Fundamentals session
- **Tell** students the next session moves into Linux & automation — the terminal and scripting skills that everything from Jenkins onward assumes they already have
- **Direct** students to the exercises and to Microsoft's own [Azure Fundamentals documentation](https://learn.microsoft.com/en-us/training/paths/azure-fundamentals/) for further reading

---

[Back](./README.md)

---
