# Setting up Terraform

An introduction to Terraform and state

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
  - **/infrastructure-as-code-terraform/intro-to-infrastructure-as-code/starter-code**
- **Make sure** 
  - You clone the entire repo

## Learning objectives

- **Understand** how Terraform enables DevOps to automate a lot of work
- **Understand** how to define **tf** files to provision resources
- **Understand** the basic syntax about resource provisioing
- **Apply** different configuration files to create resources in the cloud
- **Analyze** different state files and the information they contain
- **Evaluate** the different ways we can create multiple resources from one resource configuration block


## Sequence

### Getting started with Terraform

**Terraform** is an **IaC** tool, **Infrastructure as Code**, as we know.

It's what we'll be using to automate and provision servers on the cloud. 

We're doing this because we want to access the architecture to install software onto and deploy our applications.  

Using **cloud** resources in conjunction with **Terraform** allows us to quickly and cheaply gain access to servers in all corners of the world and reach those key markets. 

*REFER TO RESOURCE 1 - SLIDEE* <br>

![intro-to-terraform-1](./resources/intro-to-terraform-1.png)

**Terraform** is typically used to provision resources on the cloud, we can provision:
* virtual servers
* load balancers
* storage
* databases <br>
We'll be doing all of those things. 

So in the diagram **terraform** is most often used in the second step: **Provision Server**.

**Terraform** also provides some basic tools to do **Server Confguration** but typically we'll use **Terraform** just for **provisioning**. 

The steps: **Install Software & Configure Software** are generally left to **Configuration Management Tools** like:
* Ansile
* Chef
* Puppet

#### Prerequisites

There are some prerequisites to working with Terraform.

* Azure account (or the CSP you want to use Terraform with).
* Azure CLI installed and signed in (`az login`).
* VSCode (or a software text editor).
* Terraform Installation
  * [Install Terraform | Terraform | HashiCorp Developer](https://developer.hashicorp.com/terraform/tutorials/azure-get-started/install-cli)
  * Mac
    * `brew tap hashicorp/tap`
      - this allows us to manage and install HashiCorp software, one of them being **terraform**
    * `brew install terraform`
    * `terraform --version`
 * Windows
  * `choco install terraform`

<br>
<br>

### Creating and Initialising a first Terraform project

So we have **terraform** on our machine, good. <br>
We're going to be using **terraform** to create resources in the cloud and the provider we'll be using is **Azure**. 

We'll start with using basic instances and resources
* We'll create a few 
  * Azure AD App Registrations (Service Principals)
  * Resource Groups
  * Storage Accounts (with Blob Containers)
  * Virtual Machines <br>

It's fine if we're not familiar with what means, we'll be discussing them later on. 

To start off let's create a new folder on the same level as **completed-code** called **starter-code** and inside **starter-code** create another one called **terraform**. 

Then I'll just create a sub-folder called: **01-terraform-basics**

**Terraform** uses configuration files with the extension **tf** for **terraform**. <br>

So from **terraform/01-terraform-basics**: `touch main.tf`

This is where we'll define our configuration. 

**ASK** <br>
Which cloud provider are we looking to talk to? <br>
**ANWER** <br>
Azure 

In terraform, if we want to speak to a cloud provider we'll need to do some initial config and pass in a **provider** <br>

**main.tf**
```tf
# NEW CODE
# Declares the beginning of the terraform configuration block
terraform {
  # Used to specifiy the required providers and their versions 
  required_providers {
    # Defines the required provider is Azure
    azurerm = {
      # Defines the source of the Azure provider which is HashiCorp
      source  = "hashicorp/azurerm"
      # Use any version greater than 3.0
      version = "~> 3.0"
    }
  }
}

# Start of config block for the Azure provider
provider "azurerm" {
  # Required empty features block for the azurerm provider
  features {}
}

```

There's a number of providers as we know. <br>

When we're talking with Azure we need to specify a **location** (Azure's term for region) for the resources to be created in. <br>

It's something we've briefly looked at before on **Google Cloud Platform** but most of the cloud providers give us something called a **region**. <br> 
- There's multiple **regions** (Azure calls these **locations**) around the world
  - This is to give us as cloud users something called **availability** and **low latency**
  - I'll be using **uksouth**
    - It's the primary UK region and a good default for testing.

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

```

Note: unlike some other providers, `location` isn't set on the `provider` block in Azure — it's set individually on each resource (starting with the Resource Group), which we'll see shortly.

Let's use Google to see some of the other **terraform providers** <br>
*GOOGLE SEARCH: terraform providers* <br>
Then I'll take the top link: [Terraform Registry](https://registry.terraform.io/browse/providers)

We can see there's a number of providers **Terraform** supports so we don't have to be using **Azure**. 
It says:
```
Providers are responsible for understanding API interactions and exposing resources
```

I'll be showing you how to create a few useful resources but if you want to see more resources available on each provider it's a good idea to bookmark this page and you'll be able to see what's available. 

So whenever we want to create a resource or get details using terraform, we'll be using one of these providers.

Just incase there's any doubt, we haven't given ourselves access to the providers by downloading **terraform**. 

Once we add a provider key to our **main.tf** file, we'll need to run a: `terraform init` which would then download a provider onto our local machine. 

Let's do that now. 

We'll want to navigate in the terminal to where our **main.tf** lives.
* From here run: `terraform init` <br>
From the output we can see that it's:
- Initialised the backend
  - We'll talk about what that means later
- Initialised the provider plugins
  - We configured Azure as a provider, each provider has different plugins and it's downloading the plugins for that provider within the directory we're working in.
  - We can see under **Initializing provider plugins** we've got a version number. We've specified that in our configuration.  

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

```

This code is basically saying we want at least this version of the **provider plugins**.

One thing we may have noticed is we've got a new folder and file in our directory
- `.terraform` 
  - A hidden folder
  - If we `cd` all the way to the end of this folder path and run: `ls -lh` we can see it's quite a large file
  - It's the **Azure terraform provider**
  - If we're creating a lot of terraform projects just be careful because they'll end up taking a large amount of space on your machine
-  a new file got created: `.terraform.lock.hcl`
  - Similar to our **package.lock.json**
  - Serves as a reference point across the configuration to evaluate compatible dependencies


Cool, so anyway there's a few more steps we need to take before we can actually start engaging with Azure's resources. 

<br>
<br>

### Create an Azure AD App Registration (Service Principal)

We'll want to be able to talk to Azure from our command line, to do that we'll need to be able to create a **Service Principal** and get its credentials, which **authenticates** us to use Azure resources.

I'm going to sign in using my Azure AD user I created earlier: **emilesherrott_dev**
- **Subscription**: `Pay-As-You-Go`
- **Tenant ID**: `9f4b1c2a-3d5e-4a6b-8c7d-1e2f3a4b5c6d`
- **Username**: `emilesherrott_dev@outlook.com`
- **password**: `g<>!`

The quickest way to do this is via the **Azure CLI**, rather than the Portal GUI.

From the terminal, first sign in: <br>
* `az login`

This opens a browser window to authenticate, and then lists the subscriptions available to your account.

Let's now create a **Service Principal** scoped to our subscription. I'm going to give it a **name**: **command_line_user**

* `az ad sp create-for-rbac --name "command_line_user" --role="Contributor" --scopes="/subscriptions/<your-subscription-id>"`

We want to just give it **Contributor** access at the subscription level for now — this is broadly equivalent to giving programmatic access with a wide set of permissions.

Running this command returns a JSON block containing:
* `appId` (this is our **Client ID**)
* `password` (this is our **Client Secret**)
* `tenant` (this is our **Tenant ID**)

![intro-to-terraform-2](./resources/intro-to-terraform-2.png)

One of the things we can do is copy this output somewhere safe as a backup incase we forget. 

This is because, if we lose this secret, we won't be able to retrieve it — you'd need to generate a new one — so it's important to keep it secure. 

You can create a new **secret** but the existing one will be gone.

I'm going to store it somewhere which makes sense on my system. 

I have a folder: `~/azure/azure_service_principals`

<br>
<br>

### Configure Terraform Environment Variables for Azure Credentials

So let's configure our credentials as **Terraform environment variables**. 

I'm going to Google search: **Terraform provider Azure** and use the first link: [Terraform Registry](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)

We can see a few examples of how we can use the **Azure provider** in our **.tf** file. 

If we scroll down we can see a subheading **Authenticating using a Service Principal** and different ways we can provide **authentication or our credentials**. 

![intro-to-terraform-5](./resources/intro-to-terraform-5.png)

It does let us know that it doesn't like **hard-coded** credentials in Terraform configuration. It can be a security risk. 

Further down we can see an example of using hard-coded credentials, by defining the **client id, client secret, subscription id and tenant id** directly in the `provider` block, using the values we got back from `az ad sp create-for-rbac`. 

![intro-to-terraform-6](./resources/intro-to-terraform-6.png)

The problem is **Terraform** config is usually stored within our version control or tracked by Git, again, it's not something we want to make public. 

Instead we'll be using **environment variables** to handle our credentials.

**NOTE FOR STUDENTS** <br>
It may be better to run these commands from VSCode's terminal as if run from other terminals the environment variables don't set.  <br>
**END OF NOTE**

*REFER TO RESOURCE 2 - SLIDEE* <br>

* `export ARM_CLIENT_ID=<appId-taken-from-az-command>`
  * on Windows: use `set` or `setx` instead of `export`
* `export ARM_CLIENT_SECRET=<password-taken-from-az-command>`
* `export ARM_SUBSCRIPTION_ID=<your-subscription-id>`
* `export ARM_TENANT_ID=<tenant-taken-from-az-command>`

Run those four commands. 

- It may also be a good idea to add them to your `.zshrc` file or `.bashrc` file. 

- If you don't have a `.bashrc` you should create one at the root of your network. 

**.zshrc**
```
# NEW CODE
export ARM_CLIENT_ID="2b1e4c6a-8f3d-4a1b-9c7e-5d2f1a3b4c5d"
export ARM_CLIENT_SECRET="Qx9~vK2pL.8mN4rT7wZ1uY6bA3sC0dEf"
export ARM_SUBSCRIPTION_ID="7c3a9e21-1b4d-4f6a-9c8e-2d5f7a1b3c6e"
export ARM_TENANT_ID="9f4b1c2a-3d5e-4a6b-8c7d-1e2f3a4b5c6d"
```

#### MAC export 
* `export` run by itself shows all the local machines environment variables
* `export <variable-name>=<variable-value>` defines new local environment variables
* `unset <variable-name>` to delete an environment variable


#### WINDOWS set
- `printenv` shows environment variables
- `setx <variable-name> <variable-value>` defines new local environment variables
- `setx <variable-name> ""` followed by an empty space will remove variable


Let's see if we can now use them! 

<br>
<br>

### Creating a Resource Group

Before we can create almost anything in Azure, we need somewhere to put it. Azure organises resources into **Resource Groups** — a logical container that groups related resources together and doesn't exist in AWS in quite the same way.

**ASK** <br>
Why might Azure need this extra layer that other providers don't? <br>
**ANSWER** <br>
It gives us a single unit to manage lifecycle, permissions, cost tracking and clean-up for a related set of resources — deleting a resource group deletes everything inside it. 

In our **main.tf** we'll define a **resource** for our resource group.

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

<br>
<br>

### Creating Azure Storage (the equivalent of an S3 Bucket) with Terraform

Let's start by creating a **Storage Account** and a **Blob Container**

**ASK** <br>
What's Azure's equivalent of an **S3 Bucket**? <br>
**ANSWER** <br>
Azure splits this into two things: a **Storage Account**, which is the top-level container for your storage resources, and a **Blob Container** inside it, which is where the actual files (blobs) live. 

We can think of a blob container like a **file store**, we can provide a **key** (blob name) and place a **file** as the **value** against that **key**.

Azure Storage provides really high **durability**, which means once we've added data, there's a very small chance we'd lose it.

It also provides high **availability**, which means we can access it very easily.  

In the **Azure Portal**, I'll click onto **Create a resource > Storage account**

I'm going to give it the name "**storage**"

Storage account names have to be **DNS Compliant** and even more restrictive than S3 bucket names — they can only contain:
* lowercase letters
* numbers
* (no hyphens allowed, unlike S3)

We may get a response saying "**storage**" already exists.

The name needs to be unique across all **Azure accounts** globally, same idea as S3.

I'll choose: **stemilesherrottdevops**

![intro-to-terraform-9](./resources/intro-to-terraform-9.png)

We'll leave all the remaining config as default (Standard performance, LRS redundancy). 

Scroll to the very bottom and hit **Review + create**, then **Create**.

If we click into the **storage account** and then **Containers**, we can create a container and start uploading our files. 

This is cool, but we're wanting to achieve the same thing using **terraform**. 

#### Create a Storage Account and Container
So the first thing I want us to do is create a **Storage Account** and **Blob Container** with **terraform**. <br>

In our **main.tf** we'll define a **resource**.
* A resource is object we want to manage in the cloud
* To use a resource we'll need to provide two things

1. The first thing is the type of resource, for us that's an: Azure Storage Account <br>

In terraform, typically the resource type will include the cloud provider name <br>

So `<provider-name><resource-type>` <br>

For us that'd look something like:

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
# the 'resource' keyword
# the 'provider' underscore the 'specific resource'
resource "azurerm_storage_account" "" {

}
```

2. The second thing we provide is the internal name terraform uses for the resource. <br>
So whenever we want to refer to this resource in through **terraform**, we'll be able to use the name we define here. I'll use **my_storage_account**

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


# UPDATED CODE
resource "azurerm_storage_account" "my_storage_account" {

}
```


We'll also need to give the storage account in Azure a name, and — unlike S3 — tell it explicitly which resource group and location it belongs to. Using the GUI I created a storage account called **stemilesherrottdevops** <br>

We can't use the same name, as we said storage account names are globally unique. <br>

Remember they have to be DNS compatible and can't contain **hyphens** or **underscores** <br>
I'll go for: **stemilesherrottdevops01**

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

resource "azurerm_storage_account" "my_storage_account" {
    # NEW CODE
    name                     = "stemilesherrottdevops01"
    resource_group_name      = azurerm_resource_group.my_resource_group.name
    location                 = azurerm_resource_group.my_resource_group.location
    account_tier             = "Standard"
    account_replication_type = "LRS"
}
```

So in the terminal, I'd want to be in the folder our **main.tf** file exists in and then to execute the file to create our resource. 

In **terraform** if being sensible use a **two step execution approach**.

First we check what will happen if we execute the command. Then we execute after that.

### Plan
* Run: `terraform plan` <br>
This will return the details of what actions terraform would do if I executed. 

![intro-to-terraform-10](./resources/intro-to-terraform-10.png)

We can see it's **creating** a lot of things, mainly a **resource group** called: "**azurerm_resource_group.my_resource_group**" and a **resource** called: "**azurerm_storage_account.my_storage_account**"

Following that it'll create a list of things:
* primary_access_key
* primary_blob_endpoint
* primary_connection_string
* id <br>

All things we'll know after we've executed the command. <br>

The only thing we do know is the **storage account name** and that's because we've defined it ourselves. 

#### Apply
* Run: `terraform apply` <br>
**apply** is basically the execution command in terraform 

![intro-to-terraform-11](./resources/intro-to-terraform-11.png)

The output looks very similar. <br>
At the bottom we can see:
* **Plan**
  * This is telling us what we're doing and we have an output of **add / change / destroy**
  * Right now we're just performing **2 adds** (resource group + storage account)
  * At the bottom it's asking if we want to perform these actions and the only value it'll accept is: 'yes'
  * Type: `yes`

From this point terraform will try to create these resources for us. 

![intro-to-terraform-12](./resources/intro-to-terraform-12.png)

So we know our credentials we defined using `export` are working and they've been applied. 

Let's double check it's been created using the **Azure Portal**

Back in the Portal, if we go to **Resource groups > rg-emilesherrott-devops** and refresh, we can see our newly created storage account.

![intro-to-terraform-13](./resources/intro-to-terraform-13.png)
Awesome! 

If it's not worked the main issues will be
* Credentials / Environment Variables
* Unique Storage Account Name

<br>
<br>

### Playing with Terraform State: Desired, Known and Actual

If we were able to execute the **.tf** file successfully we should see a new file has been created: **terraform.tfstate**

It contains a lot of **JSON** <br>

To explain this lets: <br>
* Run: `terraform apply`

So we'll try executing our **main.tf** file again without any changes being made. 

![intro-to-terraform-14](./resources/intro-to-terraform-14.png) <br>

We can see that **terraform** understands that **No changes** have been made and therefore no actions were taken. <br>

Let's take a look into how **terraform** knew that no action was needed to take place. 

#### State
So we have to move onto the idea of state

##### Desired State
In **main.tf** file what we're specifying is something called a **desired state**.
* We want a Storage Account, some Virtual Machines…, whatever it may be.

##### Known State
* Known state is the result of previous executions. So when we executed `terraform apply` the first time, it stored the state as it understands it to be into the file **terraform.tfstate**

- "According to Azure, through Terraform this service principal has these resources provisioned on the cloud with these details"

##### Actual State
The **actual state** of the storage account is a little different, hypothetically if we went into the Azure Portal in the browser and made changes to our storage account without using **terraform**. <br>
Then the **known state** would be different to the **actual state**.

So there's 3 different states. **Desired State** is what we want, defined in **main.tf**. <br>
The state terraform has executed is the **Known State** and the **Actual State** is the state of our resources as they actually are within the provider. 

When we run: `terraform apply`, it looks at the file **terraform.tfstate** and it'll know there's a storage account been created with a specific name and **id**. <br> 
It'll then go to **Azure** and check the state regarding the resource and checks to see if there's actually been any changes. 

The ability for it to check our **known state** in **terraform.tfstate** makes the actual request to Azure much quicker. 

Based on those conditions it'll be able to execute the commands or not.

Let's make a small change to our **desired state** <br>

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

resource "azurerm_storage_account" "my_storage_account" {
  # NEW CODE
  name                     = "stemilesherrottdevops02"
  resource_group_name      = azurerm_resource_group.my_resource_group.name
  location                 = azurerm_resource_group.my_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```

Then let's run: `terraform apply` again

![intro-to-terraform-15](./resources/intro-to-terraform-15.png)

We can see from the output, it's saying **destroy and then create replacement** <br>

Then a list of the actions it's going to perform. As part of that is says: **azurerm_storage_account.my_storage_account must be replaced** <br>

So it identifies we already have a storage account, we can't directly change the storage account name so it's going to be deleted and then recreated. <br>

At the bottom we get the line: **Plan: 1 to add, 0 to change, 1 to destroy** which basically showcases what we've just read. 

Again we'll need to type: `yes` and then it'll be actioned. <br>

*TYPE: `yes`*

If we go to Azure Portal and refresh, we should see these changes come through. 

**ASK** <br>
Where is our **Known State** kept? <br>
**ANSWER** <br>
**terraform.tfstate** <br>

If we look into **terraform.tfstate** we should see these changed pulled through for us there. 

Near the bottom of our **terraform.tfstate** file we can see a key related to **blob_properties**

**terraform.tfstate**
```json
[ . . . ]
            "blob_properties": [
              {
                "versioning_enabled": false,
                "change_feed_enabled": false
              }
            ],
            "primary_blob_endpoint": "https://stemilesherrottdevops02.blob.core.windows.net/",
            "primary_blob_host": "stemilesherrottdevops02.blob.core.windows.net"
          },
          "sensitive_attributes": [],
          "private": "eyJlMmJmYjczMC1lY2FhLTExZTYtOGY4OC0zNDM2M2JjN2M0YzAiOnsiY3JlYXRlIjoxMjAwMDAwMDAwMDAwLCJkZWxldGUiOjM2MDAwMDAwMDAwMDAsInJlYWQiOjEyMDAwMDAwMDAwMDAsInVwZGF0ZSI6MTIwMDAwMDAwMDAwMH19"
        }
      ]
    }
  ],
  "check_results": null
}
```

Right now **versioning_enabled** is set to false. It's something we may want to enable. Let's make that change.

What that'll do is store multiple versions of the same blob for a period of time. We'll do this from **main.tf**, this time by adding a `blob_properties` block directly onto the storage account resource (Azure handles this a little differently to S3, which uses a separate resource).

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

So remember the syntax of these objects
```tf
resource "<provider>_<resource-type>" "<chosen_resource_name>" {
    name = "<name-provided-to-azure>"
    blob_properties {
        versioning_enabled = true
    }
}
```

So what we're doing is enabling **versioning**. <br>
**Desired State**: Is that versioning is enabled <br>
**Known State**: The result of the previous version in **terraform.tfstate** <br>
**Actual State**: What's actually in Azure <br>

* Run: `terraform apply`
What we should do before applying is: `terraform plan` to see what's happening but let's skip it. 

So what's the output suggesting? <br>
It's basically saying we need to update the existing storage account resource in place.

Because Azure exposes blob versioning as a *property* of the storage account rather than a separate resource, terraform will show this as an **update**, not an **add** — a useful contrast with how the AWS provider models the same idea.

Let's tell the terminal we want to perform these actions: Run `yes` <br>

If we go to the browser in the **Azure Portal** we can see we have the same storage account as we did before. 

If we click into the **storage account: stemilesherrottdevops02** we can click on **Data protection** under **Data management** and now see that **Blob versioning** is now **Enabled**. <br>
This should also be replicated in the **terraform.tfstate** too.

Sometimes the **terraform.tfstate** is only updated after a addition of a entirely new resource, not just a configuration change.

If you suspect your **Known State** maybe behind the **Actual State**, you can run: `terraform refresh` to update.

**terraform.tfstate**
```tf
[ . . . ]
            "blob_properties": [
              {
                "versioning_enabled": true,
                "change_feed_enabled": false
              }
            ],
```

To summerise what we've done:
* we define the **Desired State**, we can have any number of **.tf** files for individual resources
* the result of the execution is shown in the **terraform.tfstate** file which holds the **Known State**
* Of course the **Actual State** is what's in Azure
  * Typically we wouldn't want to go into a cloud provider and make changes manually
  * If a resource is managed by terraform, we'd want to continuously manage it through terraform. 

One of the things we're seeing is that terraform is **declarative**, in the sense that we're saying what the **desired state** should be and terraform is comparing it with the **actual state** and applying the differences. 

So we're not actually saying **enable versioning**, what we've done is suggest that we want **versioning** to be enabled and terraform is doing that for us. 

We'll talk more about **state** later. 

It's not necessarily the point of this section but noticing how Azure models the same feature as a property rather than a resource is a pattern we'll build into when using more complicated resources. 

<br>
<br>

### Playing with Terraform Console

Let's look at something we have access to: the **terraform console**
* Run: `terraform console` <br>

When in the console we can run lots of terraform commands, especially useful to query about the **current state**

Let's query the resource of my current storage account. 
* In the console, let's run: `azurerm_storage_account.my_storage_account`
* Syntax: `<provider>_<resource-type>.<chosen_resource_name>`

![intro-to-terraform-16](./resources/intro-to-terraform-16.png)

What's responded to us are the details of the storage account. <br>
The syntax is simple, it's called a **reference**. 

We're not using the **storage account name** we provided to Azure, terraform uses the name we've provided for our resource within our **.tf** file. 

**main.tf**
```tf
resource "azurerm_storage_account" "my_storage_account" {
  name = "stemilesherrottdevops02"
}
```

**main.tf**
```tf
resource "<provider>_<type_of_obejct>" "<name_of_object>" {
  name = "<name-provided-to-azure>"
}
```

So just to reiterate when we want to refer to a resource in terraform we use the syntax:
* Syntax: `<provider>_<type_of_resource>.<name_of_resource>`
* i.e: `azurerm_storage_account.my_storage_account`

**NOTE FOR TRAINERS** <br>
Sometimes we may need to run: `terraform refresh` out of the console to ensure that Terraform fetches the latest state from the Azure API and updates the internal state accordingly. <br>

This is because when we ran: `terraform apply` the resource was destroyed and then created with the **versioning_enabled** key set to it's default value of false and the **Known State** has been placed in our **terraform.tfstate** file before it's been updated with the latest information from the additional rule. <br>
**END OF NODE**

So running this command in the **terraform console** we can see all the details of our storage account
* Primary Blob Endpoint
* Storage Account ID
* Location
* Tags
* etc. 

We can dig down into our object to see more specific information.

* Run: `azurerm_storage_account.my_storage_account.blob_properties` <br>
![intro-to-terraform-17](./resources/intro-to-terraform-17.png)

We can also see that this is displayed as a **list** which means it's indexed so if we wanted to we could run: `azurerm_storage_account.my_storage_account.blob_properties[0]` <br>
 ![intro-to-terraform-18](./resources/intro-to-terraform-18.png)

Again we can use the methods we're familiar with from JavaScript. <br>

**ASK** <br>
If we wanted to see the value on the key **versioning_enabled**, how could we do this? <br>
**ANSWER** <br>
`azurerm_storage_account.my_storage_account.blob_properties[0].versioning_enabled`

Let's say we want this information to be outputted from our terraform console, there's a way we can do this from our configuration file. 

```tf
output "<output-name>" {
  value = <key-of-value>
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

resource "azurerm_resource_group" "my_resource_group" {
  name     = "rg-emilesherrott-devops"
  location = "uksouth"
}

resource "azurerm_storage_account" "my_storage_account" {
  name                     = "stemilesherrottdevops03"
  resource_group_name      = azurerm_resource_group.my_resource_group.name
  location                 = azurerm_resource_group.my_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "LRS"

  blob_properties {
    versioning_enabled = true
  }
}

# NEW CODE
output "my_storage_account_versioning" {
  value = azurerm_storage_account.my_storage_account.blob_properties[0].versioning_enabled
}
```

To see this we need to exit out of the **terraform console** and then run an `apply`: *DON'T RUN*

If I executed this command now it'd try to refresh the state, which we don't necessarily want. 

We're at a point where we know the **Desired State** and the **Known State** are the same. All we want to do is find the value of our **output** name: **my_storage_account_versioning**

* Run: `terraform apply -refresh=false` <br>
So we don't want to refresh this time. Terraform will compare the **Desired State** against the **Known State**, understand we've added an **output** and execute it.

![intro-to-terraform-19](./resources/intro-to-terraform-19.png)

As it's saying, **we can apply this plan to save these new output values to the Terraform state, without changing any real infrastructure**

Be careful when using `-refresh=false` in production, we always want to be checking against the **actual state** in the cloud because ultimately that's where our applications will live. 

Anyway see can see our output in the terminal. 

We're not restricted to one output either. 

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

resource "azurerm_storage_account" "my_storage_account" {
  name                     = "stemilesherrottdevops03"
  resource_group_name      = azurerm_resource_group.my_resource_group.name
  location                 = azurerm_resource_group.my_resource_group.location
  account_tier             = "Standard"
  account_replication_type = "LRS"

  blob_properties {
    versioning_enabled = true
  }
}

output "my_storage_account_versioning" {
  value = azurerm_storage_account.my_storage_account.blob_properties[0].versioning_enabled
}


output "my_storage_account_complete_details" {
  value = azurerm_storage_account.my_storage_account
}
```

* Run: `terraform apply -refresh=false` <br>
Again we can see the entire output now regarding the complete details about our storage account. 

We've been looking at this mainly to see some things we can do with terraform. Output can be used practically though:

* **Communicating Information**: We can use it to give information from our config to other parts of our infrastructure. Things like Blob Endpoint, DNS names which other parts of our application may need.
* **Debugging**: By exposing key information about our infrastructure we can keep a better eye on ensuring our infrastructure has been provisioned correctly. 
* **Integration with External Systems**: We've spoken about **Ansible** as our configuration management tool, we can give **Ansible** information it needs to perform its job through an **output**. 


That's where we're going to finish for this session. 

Now if we're using **cloud resources** there may be a cost implication so good practice is to always delete resources when we're doine using them. 
- `terraform destroy`
- `yes` 


<br>
<br>

### Conclusion

- **Inform** students this marks the end of the lecture for today
- **Tell** students we will be continuing to look at more complex resources tomorrow. 
- **Inform** the rest of the day if to finish the Kubernete exercises or playaround with **Storage Accounts** on **Terraform/Azure**

