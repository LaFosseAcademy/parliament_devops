# Understanding Resource Provisioning

Provisioing more interesting resources and creating remote backend storage. 

## Organisation

### Duration

2 hours (includes a 10 min break)

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
- **Direct** students to the starter code for this modile
  - [Student Facing Repo](https://github.com/LaFosseAcademy/cloud-devops-student-repos/tree/main)
- **Use** 
  - **infrastructure-as-code-terraform/understanding-resource-provisioning-on-azure/starter-code**
- **Make sure** 
  - You clone the entire repo

## Learning objectives

- **Understand** the different resources we can create in Terraform and Azure
- **Understand** how Azure segments its virtual servers and the layers of security we can provide
- **Apply** resources with knowledge of Virtual Networks / Subnets / Network Security Groups
- **Evaluate** different approaches for environment seperation


## Sequence

### Understanding creation of Virtual Machines in the Azure Portal

Let's move onto **Virtual Machines**. 

Let's search for **Virtual machines** in the Azure Portal and figure out some of the high level configuration we'll need for **virtual servers in the cloud**. 

What is a virtual server? <br>
Basically whenever we have a server in a data centre, it's a **physical server**, that's where we deploy our: <br>
* applications
* databases

When we deploy onto **virtual machines**, we deploy onto servers in the cloud. <br> 

I do have to say they're not actually in the cloud but in physical locations around the world but we just access them over the internet. 

In **Azure** they're called **Virtual Machines**, or **VMs** for short. <br>
We can see the **location** picker as part of the creation wizard. <br>

![understanding-resource-provisioning-1](./resources/understanding-resource-provisioning-1.png)

I'm going to switch to **North Europe** (Dublin) right now, we've seen in our terraform configuration we've been using **uksouth**. But there's resources all around the world.

**ASK** <br>
By having multiple regions what can we improve? <br>
**ANSWER** <br>
Availability <br>
Low latency <br>

So our users will get responses from regions closer to them and more consistent access to those applications. 

I'm going to make a **markdown** in our **/terraform** folder to keep track of the configuration we'll be seeing which we'll hopefully translate to **terraform** <br>

* `touch config.md`

So I'm going to start off by noting the region we're in. 

**config.md**
```md
Location - northeurope
```

Let's stay on the browser and look at launching a **VM**. <br>

![understanding-resource-provisioning-2](./resources/understanding-resource-provisioning-2.png) <br>

We can start by just looking at the different steps to creating a **Virtual Machine**

#### Basics: Project details and Instance details <br>
Fairly simple, like we've seen with **Azure AD Users**, we can give them **names** and organise them under a **resource group**. <br>

#### Image (Marketplace Image) <br>
This is the software configuration we want to launch our instance with. <br>
We can see there's: <br>
* Ubuntu Server
* Windows Server
* Red Hat Enterprise Linux
* Debian <br>
So on and so forth.

We'll choose **Ubuntu Server 22.04 LTS** which is a common, well-supported default. <br> 
Then we can be more specific and look at the underlying VM generation <br>
![understanding-resource-provisioning-3](./resources/understanding-resource-provisioning-3.png) <br>

We can see there's different settings we can choose. <br>
The main one we care about right now is that our chosen **VM size** is eligible under the **Azure free account**, but there's so many different ways we can configure this:

* **urn**: the unique identifier for a Marketplace image, made up of `publisher:offer:sku:version`. Ensures the one we've chosen is the one we're using.

* **Gen2 VM**: tells us the image is configured to boot using **UEFI** rather than the older **Generation 1 / BIOS** boot path.
  * **NOTE FOR TRAINERS**
  * **BIOS** is the old standard **Basic Input / Output System**
  * It's the fundamental software stored on a chip on the motherboard and performs funamental tasks:
    * **power-on-self-test (POST)**: checks hardware components like processor, memory, storage are working
    * **bootstrapping**: locates the **bootloader** which in turn loads the OS into memory
  * **UEFI** is the modern replacement for **BIOS**, the **Unified Extensible Firmware Interface**
    * **Graphical Interface**: It's more user-friendly compared to text-based BIOS set up screen
    * **Secure Boot**: UEFI ensures only trusted software components are loaded during the boot process to protect against malware. 
    * **Faster Boot Times**: Basically it's more efficient
  * Both are firmware interfaces that initialise the hardware and load the operating system. UEFI is newer and more advanced. Azure's **Generation 2 (Gen2)** VMs use UEFI; **Generation 1** VMs use the older BIOS-style boot.
  * **END OFF NOTE**

* **Architecture: x64**: tells us the image supports a 64-bit version of the x86 architecture. 
  * **x86 architecture** is a very common CPU architecture used in computers, it supports a wide range of software and hardware

* **Accelerated Networking: Supported**: this is Azure's equivalent of AWS's ENA. It's a high-performance networking path for VMs. When enabled the instance can take advantage of networking capabilities like:
  1. **higher throughput**: increased rate at which data can be processed
  2. **lower latency**

* **Disk type: Managed Disk**: refers to the type of storage used for the OS disk of the instance — Azure calls this a **Managed Disk**, roughly equivalent to an EBS volume in AWS. 

We don't need to understand at this stage what this means but we'll use some of this information to use as filters later on in the day. 

I'm going to make a note of the image identifier, our **urn**, so we know we're using the right Marketplace image
![understanding-resource-provisioning-4](./resources/understanding-resource-provisioning-4.png)

**config.md**
```md
Location - northeurope
# NEW CONFIG EXAMPLE
Marketplace Image (urn) - Canonical:0001-com-ubuntu-server-jammy:22_04-lts-gen2:latest
```

#### Instance type (VM size)
So we've chosen the **software**, now it's time to choose the hardware. How powerful is our machine going to be. <br>
We have options over:
* vCPUs
* Memory
* Storage
* Network Performance <br>

Again for us, what I care about is that it's eligible under the **Azure free account** (Azure gives you a **B1s** burstable VM free for 12 months as part of the free tier). <br>

![understanding-resource-provisioning-5](./resources/understanding-resource-provisioning-5.png) <br>

Ok, let's make a note of the **VM size** in our **md** file <br>

**config.md**
```md
northeurope
Marketplace Image (urn) - Canonical:0001-com-ubuntu-server-jammy:22_04-lts-gen2:latest
# NEW CONFIG
VM size - Standard_B1s
```

#### Administrator account (SSH key)
We'll ignore for now. 

#### Networking
In Networking there's two things we really need to consider. The **Virtual Network** and the **subnet**. <br>

*REFER TO RESOURCE 3 - SLIDEE* <br>
![understanding-resource-provisioning-6](./resources/understanding-resource-provisioning-6.png) <br>

When we create resources in a **data centre** they're protected by a on the device firewall. <br>
* Basically you can inspect incoming and outgoing network packets, the traffic and blocking or allowing them based on our security policies. 

In the Cloud we can achieve that on our virtual servers by creating a **Virtual Network (VNet)**, it's like our own network in the cloud. In the **VNet** we can create multiple smaller **subnets** to keep our resources in.

One of the first things we need to choose is which **subnet** to put our resource or **VM** on. 

- What we can see in the graph is that inside a **region**, in this example **North Europe** we have a **VNet**. 
- Inside that **VNet** we have 3 **availability zones** which are physically seperated locations within a **region**
- Inside each **availability zone** we then have **subnets** which is where the resources exist. 

**NOTE FOR TRAINERS** <br>
Unlike AWS, Azure doesn't pre-provision a **default VNet** in every region on your behalf. When you use the Portal wizard to create your first VM and no VNet exists yet, Azure offers to create one for you as part of the wizard — but it's created on demand, not sitting there in advance. This is a genuine difference worth flagging to students. <br>
**END OF NOTE**

If we create our **VM** inside our **private subnet** only the resources that are also inside the **VNet** will be able to communicate or talk to it. 
Things like a: <br>
* **database**: where we may only want the model on an MVC application to be able retrieve information
* **some servers**: which handle sensitive information, would be safer in a private subnet too

If we created our **VM** inside a **public subnet** (one with a public IP attached) then other resources inside and outside the **VNet** would be able to interact with it. <br>

We'd put things like: <br>
* **http servers** which should be accessible over the internet

We're going to create our own **VNet** since Azure doesn't hand us a pre-existing default one. <br>

![understanding-resource-provisioning-8](./resources/understanding-resource-provisioning-8.png)

Let's go and open a new Azure tab. I'll duplicate the one I have to keep that open and then search for **Virtual networks** in the services.  <br>

![understanding-resource-provisioning-9](./resources/understanding-resource-provisioning-9.png) <br>

Because we haven't created one yet, we'll let the VM wizard create it for us with **1** VNet and **3** subnets to match our three availability zones.

![understanding-resource-provisioning-10](./resources/understanding-resource-provisioning-10.png)

Let's click onto the newly created VNet. What we can see here is the **VNet** created in the region. <br>
I'll copy the **VNet resource ID** and we'll want to use this in **terraform** <br>

![understanding-resource-provisioning-11](./resources/understanding-resource-provisioning-11.png)

We should see that the value of this **VNet ID** matches the **VNet** referenced in the previous tab when we were creating our **VM** under the **Networking** tab. 

![understanding-resource-provisioning-12](./resources/understanding-resource-provisioning-12.png) <br>

**ORIGINAL TAB** <br>

So this VNet is now available for us to reference in this region. <br>

![understanding-resource-provisioning-13](./resources/understanding-resource-provisioning-13.png)

Let's paste that config into our **config.md** file.

**config.md**
```md
northeurope
Marketplace Image (urn) - Canonical:0001-com-ubuntu-server-jammy:22_04-lts-gen2:latest
VM size - Standard_B1s
# NEW CONFIG
VNet - vnet-emilesherrott-devops
```

*BACK ON THE ORIGINAL TAB* <br>

##### Firewall (Network Security Group)
We can also see under **Networking** a subheader called **NIC network security group**. <br>

**Network Security Groups (NSGs)** are the way we control traffic to our resources on Azure beyond placing them on **private** or **public subnets**.

They exist as their own resource and the resources within our **subnet** (or a specific network interface) can be associated with the rules we set up in our **NSG**. 

- So we may want to set up our **VM** inside a **public subnet**. <br>
- But we may also want to stop any traffic which isn't HTTP or HTTPS arriving on their respective ports 80 and 443. 

So even though we allow traffic to the subnet we can block certain traffic from reaching the **VM**.

This is what a **Network Security Group** allows. 

We can associate a **VM's network interface** with a **Network Security Group** which basically says only allow this kind of traffic. <br>

So we can think of a **Network Security Group** as a set of **firewall rules**.

If we want to set up a web server and allow internet traffic, generally we'd need unrestricted access to HTTP and HTTPS ports. 

We'll create a **Network Security Group**, we can see we're offered a set of common **inbound port rules** we can quickly apply. 

**ASK** <br>
Does anyone remember know what **SSH** traffic is?
**ANSWER** <br>
It can allow us to remotely access and control a computer. 

I'll **Allow SSH (22)** <br>
We also want to allow all **HTTP (80)** traffic

![understanding-resource-provisioning-15](./resources/understanding-resource-provisioning-15.png)

#### Disks
The default **Managed Disk** size and type is fine. 

#### Advanced Settings

We'll leave the advanced settings. 

The purpose of all this is I want to show that there's lots of things we need to think about when setting up a **VM**
* regions
* availability zones
* virtual networks
* subnets
* network security groups <br>

We'll automate all of this in **terraform**, removing the ability to make easy human mistakes and it'll actually speed up the process. <br>

We won't actually launch from the browser so let's go back and do it from **VSCode**.

<br>
<br>

### Creating new Terraform Project for Azure Virtual Machines

I'm going to start off by making a new folder for our **VM Project**
* Run: `mkdir 05-virtual-machines` <br>
Then let's actually move the **config.md** file into the new folder we've just made. <br>
* Run: `mv config.md 05-virtual-machines/`

We'll need to have **main.tf** <br>
* Run: `touch main.tf` <br>

We'll add the config we've been writing down.<br>
We've been using **uksouth** for our earlier sessions, let's actually switch it and use **North Europe** <br>

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

* Run: `terraform init`

So let's get started!

#### Virtual Network and Subnets
Because Azure doesn't hand us a pre-existing default VNet the way AWS does, our first job is to create our own — an equivalent starting point to AWS's default VPC, just as an explicit managed resource instead of something we "adopt".

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

We're already making use of the **for_each** loop we covered earlier, this time over a small **map** of subnet numbers to their address ranges, one per **availability zone**.

#### Network Security Group
To create a **VM** in the browser we saw we needed a **network security group**. <br>

We're going to build a **HTTP Server** which is a **web server**. Typically they're accessed on **port 80** via **TCP**

How do **TCP** and **HTTP** fit together? <br>
A **TCP** (Transmission Control Protocol) is basically like two friends chatting over the phone. <br>
* **Connection**: You have a connection when you dial a number and they answer
* **Conversation**: Your friend and you take turns talking and listening, like data being sent back and fourth between computers
* **Closing Call**: When you're done talking, you both say goodbye. Like a **TCP connection** being closed. 

**HTTP** comes into the conversation, because it's the structure of how **web browsers** and **servers** communicate. <br>
We send **requests**, receive **responses** so **TCP** is the connection where the **HTTP** **request & responses** are made. 


We also want to connect to the **VM** using **SSH** again this is conventionally done on **port 22** <br>

Also because it's a **HTTP server** we're creating, we'll want to make sure it's accessible from anywhere. <br> 

We'll need to define a **CIDR block**. It's used to specify a range of **IP addresses** allowed access a service. <br> 

*REFER TO RESOURCE 4 - SLIDEE* <br>
We can specify it quite easily that we accept access from anywhere with five **zeros** split by dots and the last one split by a forward slash. **"0.0.0.0/0"**. Azure also accepts the wildcard `"*"` for the same purpose. <br>

We usually refer to it as the **default route**. <br>
* **0.0.0.0**: represents any address <br>
* **/0**: this indicates that all IP addresses are included <br>

We'll need to apply all that into a **Network Security Group** so the **VM** knows what traffic can reach it and where. <br>
Once that **NSG** is defined we'll associate the **HTTP Server** or the **VM's network interface** with the **NSG**. <br>

**main.tf**
```tf
[ . . . ]

# NEW CODE
# "<provider>_<service-name>" "<resource-name>"
resource "azurerm_network_security_group" "http_server_nsg" {
  # Let's call the NSG: "http_server_nsg"
  name                = "http-server-nsg"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
}
```

Unlike AWS's `aws_security_group`, an Azure NSG doesn't take its rules inline as `ingress`/`egress` blocks by default in the pattern we'll use here — we'll define each rule as its own resource, which actually maps nicely onto the lesson we already learned with AWS: that separating rules into their own resource is often the cleaner pattern anyway.

To allow **HTTP** traffic into the instance we need to configure a rule with a **destination_port_range**. We'll use **80**. <br>

Azure NSG rules need a handful of things AWS's didn't: a **priority** (lower number = evaluated first), a **direction** (`Inbound`/`Outbound`), and an **access** value (`Allow`/`Deny`) — because unlike AWS security groups (implicit allow-list only), Azure NSGs let you explicitly **deny** traffic too.

**main.tf**
```tf
[ . . . ]

resource "azurerm_network_security_group" "http_server_nsg" {
  name                = "http-server-nsg"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
}

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

We'll need to do something very similar for our **SSH connection** as well. So we can communicate with the **instance**

**main.tf**
```tf
[ . . . ]

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

**NOTE FOR TRAINERS** <br>
Unlike AWS, where a security group denies all outbound traffic by default unless you explicitly define an `egress` rule, Azure NSGs ship with a built-in default rule (`AllowInternetOutBound`) that already permits all outbound traffic. So we don't strictly need an explicit outbound rule here the way we did in AWS — it's worth calling this out as a genuine platform difference rather than an oversight. <br>
**END OF NOTE**

Let's go back and finish connecting our **VNet, subnets and NSG** together, then check it's happy <br>
* Run: `terraform validate` <br>
If successful… <br>
* Run: `terraform apply` <br>
So it'll create the **VNet, subnets and NSG** <br>

![understanding-resource-provisioning-16](./resources/understanding-resource-provisioning-16.png)

And then 2 complimentary resources or **rules** for the **NSG**
* Inbound traffic
  * **http_ingress**
  * **ssh_ingress** <br>

![understanding-resource-provisioning-17](./resources/understanding-resource-provisioning-17.png) <br>

Terraform knows there's no current state for these resources so it's creating everything for us. 

* Run: `yes` <br>

If you get any errors, make sure the **location** value matches the region you're working in. 

Let's go to Azure in the browser and take a look at **Virtual machines** <br>
On **Networking** we should be able to select our own existing **network security group**: **http-server-nsg** <br>

![understanding-resource-provisioning-18](./resources/understanding-resource-provisioning-18.png)

Whilst in the browser, we can look for **Network security groups** in the search bar. If we click onto the one we've made we can see the information should align. <br>

We've defined our inbound rules. This is from the **port numbers** we've stated. <br>

![understanding-resource-provisioning-19](./resources/understanding-resource-provisioning-19.png)

We've also got our default **outbound rules**, which already match our requirements without us needing to define anything. 

Let's make some small changes and add some tags. <br>

**main.tf**
```tf
[ . . . ]

resource "azurerm_network_security_group" "http_server_nsg" {
  name                = "http-server-nsg"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
  # NEW CONFIG
  tags = {
    name = "http-server-nsg"
  }
}

[ . . . ]
```

* Run: `terraform apply`
* Run: `yes`

If we wanted to we can see this in the **tags** on the browser. <br>

We should get into the habit of adding tags for things like the **environment**, when we have a lot of resources in the cloud it can become difficult to manage them.

We've just looked at a lot of config so it's always good to look at documentation to make sure we understand the different things we're doing. <br>

*GOOGLE terraform azurerm network security group* <br>
https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/network_security_group <br>

If we scroll near the bottom, we can see the **argument reference**. <br>
These are the different arguments our **azurerm_network_security_group** can take. <br>
For certain arguments we can see it says **ForceNew**. 

We can see that things like **name** carry this behaviour. <br>

What it means is if I make a change to the **name** of the resource then a new one will be made and the original will be deleted. 

Just something that is good to be aware of. <br>
![understanding-resource-provisioning-20](./resources/understanding-resource-provisioning-20.png)

<br>
<br>

### Creating a new SSH Key and Setting Up

Another resource we need to create is an **SSH Key**. 

We've allowed **SSH** traffic but if we want to connect into a **VM** using **SSH** and make any direct configurations, we need an **SSH key pair** and we can associate the public half with a **VM**.<br>

And that association has to be made when the **VM** is first created.<br> 

So we should create the **SSH Key** first. 

In the browser we can search for **SSH keys** as its own service under Compute. <br>

![understanding-resource-provisioning-21](./resources/understanding-resource-provisioning-21.png) <br>

If we click into it and look at creating a new one. We can see it takes a:
* **name**
* **key pair type**:
  * **RSA**: Widely used public-key encryption algorithm. Standard for many cryptographic operations
  * **Ed25519**: A newer algorithm, popular for efficiency and security, considered faster and in some cases stronger. 
* **key pair source**
  * **Generate new key pair**: Azure creates both halves for you and lets you download the private key, just like AWS's key pair flow.
  * **Use existing public key**: you've already run `ssh-keygen` locally and just want Azure to store the public half.
* and optional **tags**

I'm going to create one in the UI, I'll give it the:
* **name**: `default-vm-ssh`
* **key pair type**: `RSA`
* **source**: `Generate new key pair` <br>

![understanding-resource-provisioning-22](./resources/understanding-resource-provisioning-22.png) <br>

Then I'll hit **Create** <br>

Immediately it'll try to save the private key file on my machine, downloaded as `default-vm-ssh.pem`. It's really important this is secure, this is how anyone can get access to our **VM** and do what they want. <br>

Similar to our **terraform state**, this should be kept away from GitHub. 

![understanding-resource-provisioning-23](./resources/understanding-resource-provisioning-23.png) <br>

For now, I'll store it somewhere I'll remember in my file system.

We should navigate to where the file is stored in the terminal. We'll need to give us, as the owner, **Read permissions**. <br> 

So from the parent directory where this file lives run: `chmod 400 default-vm-ssh.pem` <br>
* `chmod` means **change mode**
* `400` gives us reading rights
  * Each digit represents a different group, so
    * `4` is for us, the owner represents READ ONLY
    * The second character, our first `0` is for members in the same group
    * The third character, second `0` is other users and we give them both **0** permissions. 

The structure I want us to use for keeping our keys is `~/azure/azure_ssh_keys`
* `mkdir ~/azure/azure_ssh_keys`
* `mv <current-pem-location> ~/azure/azure_ssh_keys`

Cool, so now when we make our **VM** we can pair it with our **azure ssh key**

We could generate a key pair with Terraform too (the `tls_private_key` resource, or `azurerm_ssh_public_key` if we already have a public key to store), but we still have to manually download the private half in one of these flows, so we'll stick with what we've done. 

This is an important step to accessing our virtual servers which is easily forgotten so I encourage you to make a mental or physical note of this. 

<br>
<br>

### Adding Azure VM Configuration to Terraform Configuration 


So we've got our **NSG** and **SSH key**, let's actually create our **VM**. 

Because a VM in Azure needs a network interface connected to a subnet before it can be created, we'll need one more resource first: a **Network Interface (NIC)**, along with a **Public IP** so we can reach it from the internet.

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
    subnet_id                     = ""
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.http_server_pip.id
  }
}

resource "azurerm_network_interface_security_group_association" "http_server_nic_nsg" {
  network_interface_id     = azurerm_network_interface.http_server_nic.id
  network_security_group_id = azurerm_network_security_group.http_server_nsg.id
}
```

Our resource is called **azurerm_linux_virtual_machine**, and we can give it any name. <br>


**main.tf**
```tf
[ . . . ]

resource "azurerm_linux_virtual_machine" "http_server" {
  
}
```

When creating a **VM**, we need to provide lots of details. <br>

* **source_image_reference**: the marketplace image (software) installed on the machine, made up of `publisher / offer / sku / version`
* **admin_ssh_key**: which is used to associate the **public key** with the instance
* **size**: the type of instance, how much computing power
* **network_interface_ids**: the id of the network interface (and by extension, the NSG and subnet) we want to associate with the instance

We've already noted down a few of these details so let's use them.

**main.tf**
```tf
[ . . . ]

resource "azurerm_linux_virtual_machine" "http_server" {
  # NEW CODE EXAMPLE
  name                = "http-server"
  resource_group_name = azurerm_resource_group.vm_resource_group.name
  location            = azurerm_resource_group.vm_resource_group.location
  size                = "Standard_B1s"
  admin_username      = "azureuser"
  # We can look in terraform.tfstate for the id of our network interface
  network_interface_ids = ["nic-0eeb7f785f204d6b6"]

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
```

What we've been doing is hardcoding the value of **network_interface_ids**. It's likely to change frequently so we'll make it dynamic. 

The **size** and **image reference** are less likely to change as often but generally it's considered a bad habit to use hardcoded values. 

We know we can use the console to dig into our resources and extract information. 

Let's do the same thing to update our **network interface id**.

**main.tf**
```tf
[ . . . ]

resource "azurerm_linux_virtual_machine" "http_server" {
  name                = "http-server"
  resource_group_name = azurerm_resource_group.vm_resource_group.name
  location            = azurerm_resource_group.vm_resource_group.location
  size                = "Standard_B1s"
  admin_username      = "azureuser"
  # UPDATED CODE
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
```

So we're just establishing a link between our **network interface** and **VM**. 

Let's fill in the info for our **subnet_id** on the network interface, back where we left it blank earlier <br>

We'll need to go into the browser and search for **Virtual networks** in the services. Then click onto **subnets** and see those within the **VNet** which matches the **ID** we copied earlier. 

We can use any of those **subnets** <br>
![understanding-resource-provisioning-26](./resources/understanding-resource-provisioning-26.png) <br>

All of these are **public subnets** and if we scroll over we can see they're spread across different **availability zones** too. <br>


We can see that **North Europe** has 3 different availability zones. 

![understanding-resource-provisioning-27](./resources/understanding-resource-provisioning-27.png)

So let's grab a **subnet id** and we can use this as the value for **subnet_id** in **main.tf**

**main.tf**
```tf
[ . . . ]

resource "azurerm_network_interface" "http_server_nic" {
  name                = "nic-http-server"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name

  ip_configuration {
    name                          = "internal"
    # NEW CONFIG
    subnet_id                     = azurerm_subnet.public_subnets["1"].id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.http_server_pip.id
  }
}
```

So that's what we need to create our **VM**

* Run: `terraform apply`
* Run: `yes`

![understanding-resource-provisioning-28](./resources/understanding-resource-provisioning-28.png)

<br>
<br>

### Installing Http Server on a VM with Terraform - Part 1

Looking at the **terraform.tfstate** we can do a quick search for our new resource: **http_server** <br>

We can see lots of information about our **instance** <br>

![understanding-resource-provisioning-29](./resources/understanding-resource-provisioning-29.png)

We have the option of using the`terraform console` as well. 
* Run: `terrform console`
* Pass in: `azurerm_linux_virtual_machine.http_server` <br>


We have a **http_server**, we've made it accessible via HTTP to the world let's actually do some configuration of this server and add some **html**.

I said we'd use **Ansible** for this but let's showcase what we can do.

We'll need to connect to the server and we do that using the **SSH key** we made earlier. 

#### Connecting to instance
* So we'll need to define a **connection** on our resource which holds our connection details. 
* We'll then add a **provisioner** to execute a few commands on the **instance**, this block we'll define basically takes all the commands we want to run on the instance. 

Let's start by adding the **connection details**.

**main.tf**
```tf
[ . . . ]

resource "azurerm_linux_virtual_machine" "http_server" {
  name                = "http-server"
  resource_group_name = azurerm_resource_group.vm_resource_group.name
  location            = azurerm_resource_group.vm_resource_group.location
  size                = "Standard_B1s"
  admin_username      = "azureuser"
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

  # NEW CONFIG
  connection {
    type = "ssh"
    host = azurerm_public_ip.http_server_pip.ip_address
    user = "azureuser"
  }
}
```

It needs to know:
* **type of connection**: **ssh**
* **host**: what are we connecting to — because a VM in Azure doesn't expose its own public IP as an attribute (that lives on the separate `azurerm_public_ip` resource), we reference that resource's `ip_address` directly, rather than `self` like we did in AWS
* **user**: we pass in a user with the value **azureuser**
  * this is the admin username we chose ourselves when defining `admin_username` — unlike AWS, where the `ec2-user` login is baked into the AMI, in Azure we pick this username

* **private key**: we have the path to our private key, we stored it earlier on our machine
  * `~/azure/azure_ssh_keys`
  * What we'll do is create a **variable** with this value and we can use it to connect. 

**main.tf**
```tf
[ . . . ]
resource "azurerm_linux_virtual_machine" "http_server" {
  [ . . . ]
  connection {
    type = "ssh"
    host = azurerm_public_ip.http_server_pip.ip_address
    user = "azureuser"
    # NEW CONFIG
    private_key = file(var.azure_ssh_private_key)
  }
}

# NEW CONFIG
variable "azure_ssh_private_key" {
  default = "~/azure/azure_ssh_keys/default-vm-ssh.pem"
}

variable "azure_ssh_public_key" {
  default = "~/azure/azure_ssh_keys/default-vm-ssh.pub"
}
```

Now let's think about executing some commands now we should be able to connect. 

**main.tf**
```tf
[ . . . ]
resource "azurerm_linux_virtual_machine" "http_server" {
  [ . . . ]
  connection {
    type        = "ssh"
    host        = azurerm_public_ip.http_server_pip.ip_address
    user        = "azureuser"
    private_key = file(var.azure_ssh_private_key)
  }
  # NEW CONFIG
  provisioner "remote-exec" {
  }
}
```

We defined a **provisioner**, we can execute commands remotely so we've passed in **remote-exec**. 

There's three things we'll need to do:
1. Install software
2. Start software
3. Copy over our files

##### So **installing software** <br>

**main.tf**
```tf
provisioner "remote-exec" {
  # NEW CONFIG
  inline = [ 
      "sudo apt-get update -y",
      "sudo apt-get install apache2 -y",
   ]
}
```
* **inline**: takes a list of commands we'll provide
  * we'll need to install **http software**
    * **apache2**
    * `sudo apt-get install apache2 -y`
      * `sudo` allows us to execute commands as the root user
      * `apt-get` is the package manager for Debian/Ubuntu machines, which is what we chose for the **image**
      * `apache2`: the Apache HTTP web server, the Ubuntu-packaged equivalent of `httpd`.
      * It allows us to respond to requests with static files or route requests 
  * we'll also need to start the server
  * copy a file

##### Starting our server

**main.tf**
```tf
provisioner "remote-exec" {
  inline = [ 
      "sudo apt-get update -y",
      "sudo apt-get install apache2 -y",
      # NEW CONFIG
      "sudo systemctl start apache2",
   ]
}
```
* Fairly easy command to start an **apache2** server
  * `sudo systemctl start apache2`
* **NOTE FOR TRAINERS**: on Ubuntu, the `apache2` package actually starts the service automatically on install, so this line is often a no-op — we're including it anyway to keep the same explicit "install, then start" teaching structure we used with AWS.

#### Copying over files

**main.tf**
```tf
provisioner "remote-exec" {
  inline = [ 
      "sudo apt-get update -y",
      "sudo apt-get install apache2 -y",
      "sudo systemctl start apache2",
      # NEW CONFIG
      "echo message | sudo tee /var/www/html/index.html"
   ]
}
```

* Lastly we want to copy over our **html** file
  * `echo`: we've seen before, it gives an output of the argument that follows
  * `|`: the pipe passes what's on the left hand side and uses it as an input for what's one the right
  * `sudo tee`: **tee** reads from standard input and writes to output files
    * in this case, **tee** takes our **message** as it's input and writes it to the **index.html** file within a few folders
    * **sudo** is important because writing to **var** files typically requires superuser priviages. 

*SHOW STUDENTS YOUR MACHINES /var FILE ON CLI* <br>

The **/var** file at the root of my OS usually holds data files for my computer to run and on our Ubuntu machine, we'll have that **/var** folder too — conveniently, Apache's default web root is `/var/www/html` on both Amazon Linux and Ubuntu, so this path doesn't actually need to change from our AWS example. <br>

It was by installing **apache2** that we created the **index.html** file which is accessible on the server. 

Let's write a better message anyway <br>

**main.tf**
```tf
provisioner "remote-exec" {
  inline = [ 
      "sudo apt-get update -y",
      "sudo apt-get install apache2 -y",
      "sudo systemctl start apache2",
      # NEW CONFIG
      "echo Welcome - Virtual Server is at ${azurerm_public_ip.http_server_pip.ip_address} | sudo tee /var/www/html/index.html"
   ]
}
```

Because a VM resource doesn't carry its own public IP or DNS attribute the way an AWS `aws_instance` does, we refer to the separate `azurerm_public_ip` resource's `ip_address` attribute instead of `self.public_dns`. 

**complete main.tf**
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
  location = "northeurope"
}

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
  name                   = "http-server"
  resource_group_name    = azurerm_resource_group.vm_resource_group.name
  location               = azurerm_resource_group.vm_resource_group.location
  size                   = "Standard_B1s"
  admin_username         = "azureuser"
  network_interface_ids  = [azurerm_network_interface.http_server_nic.id]

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


* Run
  * `terraform validate`
  * `terraform apply`

![understanding-resource-provisioning-30](./resources/understanding-resource-provisioning-30.png)

Looks like no changes are expected to be made to our **VM**.
Most of the things we do with **terraform** are **immutable** and that's the case when adding files to an instance after created. 

We'll need to destroy and provision a new instance. 
* Run:
  * `terraform destroy`
  * `terraform apply`

<br>
<br>

### Installing Http Server on a VM with Terraform - Part 2 

If we look back at the order in which our resources were destroyed we could see it was roughly:
1. **azurerm_linux_virtual_machine**: first
2. **azurerm_network_interface_security_group_association**: then the NIC/NSG link
3. **azurerm_network_interface** and **azurerm_public_ip**: next
4. **azurerm_network_security_rule** and **azurerm_network_security_group**: last <br>

![understanding-resource-provisioning-31](./resources/understanding-resource-provisioning-31.png) <br>

This is because terraform knows we shouldn't have a **network security group** deleted while a **VM** is still relying on it — the dependency graph enforces a safe order in both directions, just as it did in our AWS example.

Equally when we create the resources: <br>
![understanding-resource-provisioning-32](./resources/understanding-resource-provisioning-32.png) <br>

We can see we first create the **VNet, subnets, NSG and rules**, then the **public IP and network interface**, then the **VM** <br>
We can also see it's connecting via **SSH** and then running all our remote commands. 

If we go to **terraform.tfstate** and search for the value of our **public_ip's ip_address**, we can copy that value into our browser and we should be able to see our message. <br>

![understanding-resource-provisioning-33.png](./resources/understanding-resource-provisioning-33.png)

The steps are when we visit the **IP address**
1. We're routing to that **IP address**, connected by **TCP** and we send an automatic **HTTP get request** which by default goes to **port 80**
2. We installed **apache2** using **remote-exec**, which created an **index.html** file with some content. 
3. The **/var/www/html** is the default web server directory on many Linux distributions, so we see our message in the response in our browser. 

So we've automated the deployment of our first server in Azure. 

<br>
<br>

### Immutable Servers with Infrastructure as Code

So we saw earlier that we couldn't change the message our **HTML** file receives and run: `terraform apply`.

We had to destroy and re-create. This was because our servers are **immutable**. <br>
Let's take a moment just to talk about why that's a good thing. 

Typically when using **Infrastructe as Code** and we're **Provisioning our servers** <br>

*REFER TO RESOURCE 5 - SLIDEE* <br>
![understanding-resource-provisioning-34](./resources/understanding-resource-provisioning-34.png) <br>


If our **instances** were **mutable** and we needed to make different changes to them with scripts. 

We may need to run a different script to execute a different change to our config. <br> 
Each time we'd be updating our **state**

If we wanted to create a new server and have it reach the same configuration of the previous insances. We'd need to run all the same scripts in the same order as all the previous instances. 

When writing **Infrastructure as Code** it's advised to use **immutable servers** <br>

*REFER TO RESOURCE 6 - SLIDEE* <br>
![understanding-resource-provisioning-35](./resources/understanding-resource-provisioning-35.png)

So once we've provisioned a server, if we want to make any changes we can create a new resource with the second edition of the configuration and the previous one is destroyed. 

If we wanted to keep the old version of servers, we'd just define a new resource. 

This is the recommended approach. 

Updates we're possibly talking about may be a security patch or software upgrade.

<br>
<br>

### Remove hardcoding of the Subnet with a Data Provider

Let's get back to addressing our hardcoded values. 

**main.tf**
```tf
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
```

**NOTE FOR TRAINERS** <br>
This is a good spot to compare notes with the AWS session. There, students used a `data "aws_default_vpc"` resource to *adopt* an already-existing default VPC, and a `data "aws_subnets"` provider to look up its subnets, because AWS provisions those resources automatically in every region. <br>
Here, we didn't need either — because we created our own `azurerm_virtual_network` and `azurerm_subnet` resources ourselves earlier in this project, referencing `azurerm_subnet.public_subnets["1"].id` is *already* dynamic; there was never a hardcoded value to remove in the first place. This is a genuine platform difference worth spending a minute on: Azure not having a "default network" to adopt means there's less need for this particular flavour of data provider. <br>
**END OF NOTE**

<br>
<br>

### Remove hardcoding of the VM Image with a Data Provider

Let's still make use of a **data provider**, this time to make our **image version** dynamic instead of hardcoding `"latest"` as a literal string. <br>

Just to remind ourselves, the image is identified by a **publisher / offer / sku / version** combination. <br>

*GO TO AZURE CREATE VM* <br>

![understanding-resource-provisioning-43](./resources/understanding-resource-provisioning-43.png)

So it's the "template that contains the software configuration required to launch our instance". 

And for each different base OS there's lots of different configurations. 

We chose **Canonical:0001-com-ubuntu-server-jammy:22_04-lts-gen2**, and simply put `"latest"` as the version. 

That actually already works and stays dynamic without any extra effort — Azure resolves `"latest"` for us automatically at apply-time. But if we want to *know* and pin the actual resolved version number (useful for auditing, or if we ever want a reproducible, non-moving target), we can look it up explicitly with a **data provider**. <br>

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

We'll want to specify a few things. <br>

* **location**: the image catalogue can differ slightly by region, so we tell Azure where we're deploying
* **publisher / offer / sku**: the same three identifiers we already knew, just without the version

Let's try executing this code now. <br>
* Run: `terraform apply` <br>

It should say no changes which makes sense, we've not assigned this data anywhere yet but let's inspect it in the terraform console.

* Run:
  * `terraform console`
  * `data.azurerm_platform_image.ubuntu_latest`

![understanding-resource-provisioning-46](./resources/understanding-resource-provisioning-46.png)

So it's returned information, including the resolved **version** attribute — a real build number rather than the literal string `"latest"`.

Let's look at the **version** and use it in our **VM**.

**main.tf**
```tf
resource "azurerm_linux_virtual_machine" "http_server" {
  [ . . . ]
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts-gen2"
    # NEW CONFIG
    version   = data.azurerm_platform_image.ubuntu_latest.version
  }
}
```

* Run: `terraform apply` <br>

![understanding-resource-provisioning-50](./resources/understanding-resource-provisioning-50.png)

Depending on whether a newer image build has shipped since we last applied, this might show a change — because we've now swapped a static `"latest"` alias for a real, resolved version number. If it doesn't change, that's a strong hint no new image build has been published since we last checked.

<br>
<br>

### Playing with Terraform Graph and Destroy Virtual Machines

Let's look at a fun new command <br>

* Run: `terraform graph` <br>

![understanding-resource-provisioning-51](./resources/understanding-resource-provisioning-51.png) <br>

It's outputting a **diagraph**, it's a graph showing the resources we're using and the connections between the resources and what they depend on. 

Let's google search: **graphviz online**, and click on the **dreampuf link** <br>

[Graphviz Online](https://dreampuf.github.io/GraphvizOnline/)

Then copy and paste the **diagraph** into the console.

It's basically an online viewer for our projects. <br>
The arrows show what our resources are dependent on so: <br>
* **azurerm_linux_virtual_machine.http_server** is dependent on:
  * **data.azurerm_platform_image.ubuntu_latest**
  * **azurerm_network_interface.http_server_nic**
  * **azurerm_network_security_group.http_server_nsg** (via the NIC association) <br>
Etc..

![understanding-resource-provisioning-52](./resources/understanding-resource-provisioning-52.png)

It basically makes it easier for us to understand our code. 

Another thing we could and should do is refactor our file so we separate the different pieces of functionality into logical files. <br>
I will make: <br>
* **variables.tf**
* **data-providers.tf**

* Run: `touch variables.tf data-providers.tf`

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

I can see we've got a lot of resources dedicated to my **network security group** so I'll separate that code as well. <br>

* Run: `touch network-security-group.tf`

**network-security-group.tf**
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
  location = "northeurope"
}

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
  name                   = "http-server"
  resource_group_name    = azurerm_resource_group.vm_resource_group.name
  location               = azurerm_resource_group.vm_resource_group.location
  size                   = "Standard_B1s"
  admin_username         = "azureuser"
  network_interface_ids  = [azurerm_network_interface.http_server_nic.id]

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
```

I can run: `terraform apply`

And everything should be the same, we've just refactored things.

That's the first mini-project we're doing with **Virtual Machines**, congratulations.

Let's run: `terraform destroy`  <br>

<br>
<br>

### Creating new Terraform Project for VMs with a Load Balancer 

Let's go ahead and create a project with multiple **VMs** which are kept behind a **Load Balancer**.

* Run: `mkdir 06-vms-with-lb`

I'm going to copy over some files from project 5 to project 6.  <br>
* from **05-virtual-machines** run: `cp data-providers.tf main.tf network-security-group.tf variables.tf ../06-vms-with-lb`

I'll start by making a few updates. I don't want to make use of one instance but many so I'm going to change the name of my **VM resource**.

**06-vms-with-lb main.tf**
```tf
# NEW CONFIG
# Change of name to reflect the resource
resource "azurerm_linux_virtual_machine" "http_servers" {
  name                   = "http-server"
  resource_group_name    = azurerm_resource_group.vm_resource_group.name
  location               = azurerm_resource_group.vm_resource_group.location
  size                   = "Standard_B1s"
  admin_username         = "azureuser"
  network_interface_ids  = [azurerm_network_interface.http_server_nic.id]

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
```

We're going to want to interact with all our instances and state values may start to become hard to navigate. 

Because each VM needs its own network interface (and, in our earlier example, its own public IP), we'll also need to switch our **network interface** and **public IP** resources over to `for_each`, one per subnet/availability zone. 

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

What I'll do is create an **outputs.tf** file and add some config just to output our **public IP addresses** for the **instances** we're making. 

* from **06-vms-with-lb** run: `touch outputs.tf` <br>

**outputs.tf**
```tf
# NEW CONFIG
output "http_server_public_ips" {
  value = { for k, pip in azurerm_public_ip.http_server_pips : k => pip.ip_address }
}
```

This should just define our **public IP addresses**. <br>

Let's create multiple http servers then.

We can leave the:
* Image reference
* SSH key
* Size

What we want to do is create a **VM** in each of the **subnets**. <br>

**ASK** <br>
What's the key word for looping in HashiCorp Control Language <br>
**ASNWER** <br>
for_each <br>

Just like we did with the network interfaces, we'll loop the VM resource over our **subnets** map. <br>

**main.tf**
```tf
resource "azurerm_linux_virtual_machine" "http_servers" {
  # NEW CONFIG
  for_each               = azurerm_subnet.public_subnets
  name                    = "http-server-${each.key}"
  resource_group_name    = azurerm_resource_group.vm_resource_group.name
  location               = azurerm_resource_group.vm_resource_group.location
  size                    = "Standard_B1s"
  admin_username          = "azureuser"
  network_interface_ids   = [azurerm_network_interface.http_server_nics[each.key].id]

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
      "echo Welcome - Virtual Server is at ${azurerm_public_ip.http_server_pips[each.key].ip_address} | sudo tee /var/www/html/index.html"
    ]
  }
}
```

We should have **3 VMs** being spun up, one for each availability zone/subnet, each with **tags** to identify them from each other.

Let's go ahead and run a few commands:
* `terraform init`
* `terraform apply` <br>

![understanding-resource-provisioning-53](./resources/understanding-resource-provisioning-53.png)

In the output we can see that there's three **http servers** which would be created so that's great! <br>

* Run: `yes` <br>

Again, it knows which resources are dependant on each other and the **network security group** is created before the **VMs**. <br>

There's a lot of output, we're creating three instances and installing the software and html files on all of them. 

Let's make sure it's working before hand though. Search the terminal output for one of the **public IP addresses** and see if we can use it in our browser. <br>

*PASTE IP INTO BROWSER* <br>

It looks good! So we've got three instances running. Although it's within the Azure free account allowance for a single B1s VM, running three at once means we're consuming that allowance at a much quicker rate. So let's keep the runtime as small as possible.  <br>

<br>
<br>

### Create Network Security Group and Load Balancer in Terraform

Let's now try and create our **load balancer** between the different **VMs**

To create a **Load Balancer** we have to create a new **Network Security Group**. <br> 

We wouldn't want to use the same one for both. Because we don't want anybody to be able to get into it using **SSH** and make any updates. 

So I'll copy our previous configuration and change some of the names. <br>

We'll call it **lb_sg** instead of **http_server_nsg**

We can remove the tags but just need to delete the **security rule** for **SSH ingress**. 
<br>

We'll also want to change the **network_security_group_name values** <br>

**network-security-group.tf**
```tf
# HTTP_SERVER NSG ABOVE
[ . . . ]

# LOAD BALANCER CONFIG
# NEW CONFIG
# NEW NAME
resource "azurerm_network_security_group" "lb_nsg" {
  # NEW NAME
  name                = "lb-nsg"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
}

# NEW NAME
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

Notice we haven't defined an explicit **egress** rule here either — same as before, Azure's default `AllowInternetOutBound` rule already covers that.

Let's actually create the **load balancer** resource. <br>

Unlike AWS's Classic Load Balancer, which is one all-in-one resource, Azure spreads a Load Balancer across several linked resources: a frontend **Public IP**, the **Load Balancer** itself, a **Backend Address Pool**, a **Health Probe**, and a **Load Balancing Rule**. There's also a real distinction between:

* **Basic SKU**: the oldest tier in Azure's catalogue, comparable in spirit to a Classic ELB
* **Standard SKU**: supports zone redundancy, higher scale, and stronger security defaults — the modern recommended choice
* **Application Gateway**: Azure's equivalent of an AWS Application Load Balancer, supports HTTP(S) content-based routing
  * Highly scalable and good for path-based routing and WebSockets
* **Azure Front Door**: Ultra-high performance, global routing and caching, good for handling large volumes of requests
  * Popular for global content delivery


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
  # We'll need to name it
  name                = "lb"
  location            = azurerm_resource_group.vm_resource_group.location
  resource_group_name = azurerm_resource_group.vm_resource_group.name
  sku                 = "Standard"

  # It needs a frontend IP configuration, tied to our public IP
  frontend_ip_configuration {
    name                 = "lb-frontend"
    public_ip_address_id = azurerm_public_ip.lb_pip.id
  }
}
```

Now let's look at the **backend pool**, **health probe** and **rule** <br>

Our backend pool is where our **VMs** live. And we've distributed those **VM network interfaces** between each of the **subnets**. 

**main.tf**
```tf
[ . . . ]

# NEW CONFIG
resource "azurerm_lb_backend_address_pool" "lb_backend_pool" {
  loadbalancer_id = azurerm_lb.lb.id
  name            = "http-servers-pool"
}
```

For attaching our **VMs' network interfaces** we want to associate each one with the pool we've just made. <br>

We'll do this per-network-interface, using a **for_each** over our set of NICs — much like `instances = [...]` did all at once for AWS's Classic ELB, but here each association is its own resource. <br>

**main.tf**
```tf
[ . . . ]

# NEW CONFIG
resource "azurerm_network_interface_backend_address_pool_association" "http_server_nics_pool" {
  for_each                = azurerm_network_interface.http_server_nics
  network_interface_id    = each.value.id
  ip_configuration_name   = "internal"
  backend_address_pool_id = azurerm_lb_backend_address_pool.lb_backend_pool.id
}
```

We also need a **health probe** so the load balancer knows which backends are actually healthy. <br>

**main.tf**
```tf
[ . . . ]

# NEW CONFIG
resource "azurerm_lb_probe" "http_probe" {
  loadbalancer_id = azurerm_lb.lb.id
  name            = "http-probe"
  port            = 80
  protocol        = "Http"
  request_path    = "/"
}
```

And finally the **rule** that ties the frontend port to the backend port, using our probe and pool. <br>

**main.tf**
```tf
[ . . . ]

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

I want to look at my **load balancer** in some more detail so let's define a new **output**. <br>

Whilst we're here we can also update our first **output**

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

Let's apply our new config. <br>
* Run: `terraform apply` <br>

And it should want to create our new **load balancer** and its related resources, along with our new **network security group**.

<br>
<br>

### Review and Destroy VMs with a Load Balancer

So we should have a couple of outputs. In the **lb_public_ip** output we should see the address we can browse to. <br>
We can find it in the console too if necessary. <br>

![understanding-resource-provisioning-58](./resources/understanding-resource-provisioning-58.png)

We can copy that **IP** and paste it into the browser. <br>
We'll see there should be a response. <br>

**NOTE FOR STUDENTS** <br>
Don't worry if not, sometimes it takes a few minutes for the load balancer and its health probe to fully settle. <br>
**END OF NOTE**

If we keep refreshing though, we can see it **load balancing** between all the different **VMs**

![understanding-resource-provisioning-59](./resources/understanding-resource-provisioning-59.png)

![understanding-resource-provisioning-60](./resources/understanding-resource-provisioning-60.png)

![understanding-resource-provisioning-61](./resources/understanding-resource-provisioning-61.png)

So we've created a pretty comprehensive structure
* Network security groups
* Different VMs split by Availability Zones
* Load Balancer <br>
All automated through **terraform**.

I'm also interested to see a new **diagraph** <br>

* Run: `terraform graph`

![understanding-resource-provisioning-62](./resources/understanding-resource-provisioning-62.png)

I'll copy that and put it into our website we used earlier. 

[Graphviz Online](https://dreampuf.github.io/GraphvizOnline/)

![understanding-resource-provisioning-63](./resources/understanding-resource-provisioning-63.png)


Let's destroy our resources because these things now consume more of our free-tier allowance than we want. 

- `terrafrom destroy`
- `yes`

<br>
<br>

### Creating Terraform Project for storing Remote State in Azure Storage

So our destruction is complete! Let's now play around with something called **Remote State**. <br>

Right now all the state we have is on our **local machine**.

We don't want to use GitHub for security reasons but even without security to think about we can't guarantee the latest versions of our **Known State** would always be **added, committed and pushed**. 

This is why **Remote Backends** are commonly used.

We're going to use our first project: **02-terraform-basics** so I'm just going to copy the folder and create a duplicate of it called **07-backend-state**

From **parent of projects folder** run:
* `cp -r 02-terraform-basics/ 07-backend-state`

I'll also delete:
* .terraform.lock.hcl
* aduser.tfplan
* terraform.tfstate
* terraform.tfstate.backup

* from **07-backend-state**: `rm .terraform.lock.hcl aduser.tfplan terraform.tfstate terraform.tfstate.backup `

Let's actually make it really simple.

**main.tf**
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
  user_principal_name = "my_iam_user_def@emilesherrottdevops.onmicrosoft.com"
  display_name        = "my_iam_user_def"
  mail_nickname        = "my_iam_user_def"
  password             = "ChangeMe123!ChangeMe"
}
```

For the **outputs.tf** all I'll do is keep the one related to the **Azure AD user** <br>

**outputs.tf**
```tf
output "my_azuread_user_complete_details" {
  value = azuread_user.my_azuread_user
}
```

To get this set up we'll need to run: 
* `terraform init`
* `terraform apply`
  * `yes`

So our state file has been locally created: **terraform.tfstate**

Just to recap what've we've gone through already. When we try to update our state, our **Desired State** is in **main.tf** and it'll check the name of the resources. <br>

Right now we only have **my_azuread_user** and it'll look in **terraform.tfstate** to see the the **id** for that resource on Azure then query whether the **Desired State** matches the **Actual State**. Then refresh the state. 

We've said that keeping the state here can be an issue when working with multiple people. <br>

So let's restructure the project slightly and pretend **07-backend-state** is the entire project. 

We'll create some **sub-projects**
1. **users**: which will contain our current files:
   * **main.tf**
   * **outputs.tf**
   * **terraform.tfstate**

* `mkdir users`
  * `mv .terraform.lock.hcl main.tf outputs.tf terraform.tfstate ./users `

So we don't want the state locally but remotely. <br> 
**Azure** offers us the ability to store the state in a **Storage Account**, inside a **blob container**

From **07-backend-state** I'm going to create a new folder. <br>
* `mkdir backend-state` <br>

It should be separate because we'll create a **Storage Account** in **backend-state** and we may want to use the **backend-state** for multiple projects whether it's for resources related to **Azure AD users**, **load balancers** or **virtual machines**. <br>

So **backend-state** should live on it's own, we'll create the resources we need to create the storage and we'll then need to reconfigure our **users config** to make use off that. 

<br>
<br>

### Create Remote Backend Project for creating an Azure Storage Account

In **backend-state** we'll need to initiate a new project. 

* `touch main.tf` <br>

In **main.tf** we'll need to add the initial configuration, adding in the **provider** and **version**

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

resource "azurerm_resource_group" "backend_resource_group" {
  name     = "rg-backend-state-emilesherrott-devops"
  location = "uksouth"
}
```

So what we want to do is create a **Storage Account** which stores the state of all our different projects, whether that's for **users**, **load balancers** etc.


#### Locking

Something else which is really important is the idea of **locking**, we could have multiple team members, executing `terraform commands` at the same time and **state** could become corrupted and caught between two **Actual States**. <br>

* We'd want to **lock** the state store to stop any other changes when we run an **apply**
* Then we update the state
* Then **unlock** it to be used and potentially updated again. 

**NOTE FOR TRAINERS** <br>
This is a nice moment to compare with the AWS session. There, students had to stand up a whole extra resource — a **DynamoDB table** — purely to hold locks alongside the S3 bucket. <br>
Azure's `azurerm` backend doesn't need an equivalent table at all: locking is handled automatically using a **blob lease** on the state file itself, a native feature of Azure Blob Storage. One less resource to create and maintain — a genuine simplification worth calling out. <br>
**END OF NOTE**

A good thing about **Storage Accounts** is that we can encrypt everything, so using a **remote-backend** we get automatic **locking** (via blob lease) to avoid state corruption, and encryption at rest by default too so it's really secure.  

Let's create our **Storage Account** and container

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

resource "azurerm_resource_group" "backend_resource_group" {
  name     = "rg-backend-state-emilesherrott-devops"
  location = "uksouth"
}

# NEW CONFIG
resource "azurerm_storage_account" "organisation_backend_state" {
  name                     = ""
  resource_group_name      = azurerm_resource_group.backend_resource_group.name
  location                 = azurerm_resource_group.backend_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```

When it comes to the storage account name, we can choose how we want to structure it. 
* One storage account could hold all of the state related to all projects in all environments.
* Or we could store data related to specific environments to specific storage accounts.

*REFER TO RESOURCE 7 - SLIDEE* <br>
* If we decide the storage account is only storing state, relating to one application in one environment we can use a name like:
  * `stapplicationnamebackend`
* If it's for all applications related to a specific environment:
  * `stdevbackend`

Remember, storage account names can't contain hyphens or underscores. I'm going to use this **Storage Account** on a specific application and in a specific environment so I'm going to name it: <br>
`stdevappsbackendemilesherrott` <br>

I'm adding **emilesherrott** because storage account names need to be unique across all of Azure

**main.tf**
```tf
resource "azurerm_storage_account" "organisation_backend_state" {
  # NEW CONFIG
  # Obviously don't use the exact same name
  name                     = "stdevappsbackendemilesherrott"
  resource_group_name      = azurerm_resource_group.backend_resource_group.name
  location                 = azurerm_resource_group.backend_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```

I want to enable versioning and we know from previous examples this is defined via the `blob_properties` block on the storage account. <br>

**main.tf**
```tf
[ . . . ]

resource "azurerm_storage_account" "organisation_backend_state" {
  name                     = "stdevappsbackendemilesherrott"
  resource_group_name      = azurerm_resource_group.backend_resource_group.name
  location                 = azurerm_resource_group.backend_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "LRS"

  # NEW CONFIG
  blob_properties {
    versioning_enabled = true
  }
}
```
 
Encryption at rest is enabled by default on every Azure Storage Account — unlike AWS, there's no separate resource we need to attach for it, it's just always on.

Now we need somewhere inside the storage account to actually put our state files — a **blob container**. <br>

**main.tf**
```tf
resource "azurerm_storage_account" "organisation_backend_state" {
  name                     = "stdevappsbackendemilesherrott"
  resource_group_name      = azurerm_resource_group.backend_resource_group.name
  location                 = azurerm_resource_group.backend_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "LRS"

  blob_properties {
    versioning_enabled = true
  }
}

# NEW CONFIG
resource "azurerm_storage_container" "tfstate_container" {
  name                  = "tfstate"
  storage_account_name  = azurerm_storage_account.organisation_backend_state.name
  container_access_type = "private"
}
```

So we've got our config. We've enabled versioning incase we've made a mistake. <br>
We've got encryption by default for the sensitive information. <br>
Things like: 
* database passwords
* secrets

And because Azure Blob Storage handles **locking** natively via a lease on the blob, we don't need to configure anything extra for that — unlike the separate DynamoDB table AWS required.

Let's see if it works. <br>

From **07-backend-state/backend-state** run:
* `terraform init`
* `terraform validate`
* `terraform apply`

So it's creating our resources <br>
* The main ones:
  * **Resource Group**
  * **Storage Account**
  * **Blob Container**

* `yes`

<br>
<br>

### Update User Project to use Azure Storage Remote Backend

Our **apply** has succeeded. So we should now have:
* A **Storage Account**
* And a **Blob Container**

We've got two projects now **users** and **backend-state** and we want to store the state of users into our **backend Storage Account**.

Ideally near the top of **main.tf** we need to define a **backend block** to link our **users / main.tf** with our **backend Storage Account**

**07-backend-state / users / main.tf**
```tf
terraform {
  required_providers {
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.0"
    }
  }
  # NEW CONFIG
  # Specify the type of backend
  # It could be S3, GCS
  backend "azurerm" {
    # Which resource group is our storage account in
    resource_group_name = ""
    # Name of the storage account
    storage_account_name = ""
    # Name of the blob container
    container_name = "tfstate"
    # What is the key (blob name) to store state information on
    # This should contain infomration about:
    # # Application
    # # Environment
    # # Project
    key = "app-env-prj"
  }
}

provider "azuread" {}

resource "azuread_user" "my_azuread_user" {
  user_principal_name = "my_iam_user_def@emilesherrottdevops.onmicrosoft.com"
  display_name        = "my_iam_user_def"
  mail_nickname        = "my_iam_user_def"
  password             = "ChangeMe123!ChangeMe"
}
```

Let's actually populate some of the values.

We know the:
- resource group name
- storage account name

**main.tf**
```tf
[ . . . ]


terraform {
  backend "azurerm" {
    # NEW CONFIG
    resource_group_name  = "rg-backend-state-emilesherrott-devops"
    storage_account_name = "stdevappsbackendemilesherrott"
    container_name        = "tfstate"
    key                   = "app-env-prj"
  }
}

[ . . . ]
```

Let's adjust our **key** too. <br>
We need to define the: <br>
* application
* environment
* project

* **application**
  * what links to two directories together
  * let's use the value `07-backend-state`
* **environment**
  * `dev`
* **project**
  * the sub-project within **07-backend-state**
  * is `users`

We can order this slightly differently we can have the **environment** at the end too. 

**main.tf**
```tf
[ . . . ]


terraform {
  backend "azurerm" {
    resource_group_name  = "rg-backend-state-emilesherrott-devops"
    storage_account_name = "stdevappsbackendemilesherrott"
    container_name        = "tfstate"
    # UPDATED CONFIG
    # commented out
    # key = "app-env-prj"
    # UPDATED CONFIG
    key = "07-backend-dev-users.tfstate"
  }
}

[ . . . ]
```

That's everything configured. We've got the:
* the storage account with a specific:
  * resource group
  * container
  * key

**from /07-backend-state/users**
* Run: `terraform init`
  * We do this because we're changing where state is stored. If you get errors run: `terraform init -reconfigure`
  * So it's asking do we want to copy existing state to the backend. We do have a **terraform.tfstate** in the **users project**
  
![understanding-resource-provisioning-65](./resources/understanding-resource-provisioning-65.png) <br>

* Run: `yes`	
  * What would happen is the local state will be copied over <br>

![understanding-resource-provisioning-66](./resources/understanding-resource-provisioning-66.png) <br>

What we can do know is delete:
* **terraform.tfstate**
* **terraform.tfstate.backup** <br>
As we won't be using these anymore. 
* Run: `rm terraform.tfstate terraform.tfstate.backup` <br>
That **Known State** should now present in our **blob container**

If we go to the browser I can search for **Storage accounts** and see our accounts. I called mine: <br>
**stdevappsbackendemilesherrott** 

If we click on it, then **Containers > tfstate**, we can see there's a blob which has the same name as the value we provided to the **key** in the **terraform backend configuration** in **users / main.tf** <br>

![understanding-resource-provisioning-67](./resources/understanding-resource-provisioning-67.png)

If we open it, we'll see it contains all the information about our **Known State**

![understanding-resource-provisioning-68](./resources/understanding-resource-provisioning-68.png)


![understanding-resource-provisioning-69](./resources/understanding-resource-provisioning-69.png)


We can see the state about the **user**

So we had an isolated directory containing our **desired state** for our **Azure AD users**.

What we could do is add more directories for resources like servers, load balancers and everything else we've looked at and use the same **remote backend** but we'd just change the **key value** and all the state would be seperated within this storage account. 

Another approach to the **key** we gave the **remote state** in **users / main.tf** is a hierarchal approach, using forward slashes as pseudo-folders in the blob name.

If we want to change the approach we should run `terraform destroy` to delete the **Azure AD User** because what we'll be doing to changing the location of the **terraform.tfstate** file and we could end up in the position where trying to define a new user in a new **terraform.tfstate** file when the resource already exists elsewhere. 

- run: `terraform destroy`

**main.tf**
```tf
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-backend-state-emilesherrott-devops"
    storage_account_name = "stdevappsbackendemilesherrott"
    container_name        = "tfstate"
    # key = "environment / application / project / chosen state"
    # NEW CONFIG
    key = "dev/07-backend-state/users/backend-state.tfstate"
  }
}
```

Let's do another `terraform init -reconfigure`

It's up to us how we specify a key, we just need to ensure it contains:
* environment
* application
* project <br>
- run: `terraform apply`


*REFRESH BLOB CONTAINER UNTIL WE SEE THE /dev PREFIX* <br>
It may take some time.


With this structure we can see a folder-like naming system instead of a single file and this generally gives us much better organisation and control over the state we hold. 
 

<br>
<br>

### Conclusion

- **Inform** students this marks the end of the lecture on Terraform
- **Tell** students we'll move onto pipelines tomorrow 
- **Direct** students to the exercises for the rest of this afternoon

---

[Back](./README.md)

---

