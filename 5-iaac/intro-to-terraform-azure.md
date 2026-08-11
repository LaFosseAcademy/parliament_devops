# Introduction to Terraform

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

So earlier we created a **Storage Account** and we added on some additional details regarding how we change the properties of the account (versioning). We're going to look at a lot more resources today.

**ASK** <br>
Can anyone remember the syntax we have to created Azure resources? <br>
**ANSWER** <br>
`resource "<provider>_<resource_name>" "<internal_name>"`

**ASK** <br>
Ok. Why is using Terraform to provision cloud resources good? <br>
**ANSWER** <br>
- Removes human error having the config defined in files (errors can be created if we're manually accessing each resource)
- Quick to get access to resources in different locations
- Quick to remove access to resources
- Developers who are more familiar with text editors like VSCode can be comfortable working with resources

### Creating an Azure AD User with Terraform

Let's go ahead and create an **Azure AD User**. 

Don't worry, we'll get to using terraform to create Virtual Machines and Load Balancers but we'll start simple just to get our bearings with terraform and these resources have no meaningful cost implications.

We'll start with the **resource syntax**

**main.tf**
```tf
resource "azuread_user" "my_azuread_user" {
  
}
```

Anyway, our:
* provider is **Azure AD** (a separate provider from `azurerm`, called `azuread`)
* resource is: **user**
* the name we're going to assign to that resource in terraform is **my_azuread_user**

The same way with our **Storage Account** resource, we gave our **account** a name, let's give our **Azure AD** user a name too. An Azure AD user needs three pieces of naming information: a **user_principal_name** (their sign-in email, which must sit on a domain your tenant owns), a **display_name**, and a **mail_nickname**. We'll also need to set a **password**.

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

# NEW CODE
resource "azuread_user" "my_azuread_user" {
  user_principal_name = "my_iam_user_abc@emilesherrottdevops.onmicrosoft.com"
  display_name        = "my_iam_user_abc"
  mail_nickname        = "my_iam_user_abc"
  password             = "ChangeMe123!ChangeMe"
}


output "my_storage_account_versioning" {
  value = azurerm_storage_account.my_storage_account.blob_properties[0].versioning_enabled
}


output "my_storage_account_complete_details" {
  value = azurerm_storage_account.my_storage_account
}
```

**NOTE FOR TRAINERS** <br>
Azure AD enforces a password complexity policy by default (a mix of upper/lowercase, numbers and symbols, minimum length 8). If `terraform apply` rejects the password, tweak it until it passes — this is a good moment to mention that in a real project we'd generate this with the `random_password` resource rather than hard-coding it. <br>
**END OF NOTE**

So we'll create an **Azure AD** user with this specific name. 

Let's be well behaved and use `terraform plan` before we execute. 

An interesting thing we can do as well is output that plan to a specific location.

* Run: `terraform plan -out aduser.tfplan`

The extension we'll use to store terraform plans will be **tfplan**, very verbose!<br>

![intro-to-terraform-20](./resources/intro-to-terraform-20.png)

If we read the output, we can see the plan would be to create a resource, an **azuread_user** and the only thing we can really be sure of is the name of the **Azure AD User: my_iam_user_abc@emilesherrottdevops.onmicrosoft.com**. 

It also tells us at the bottom if we want to execute plan we can do so by running: `terraform apply "aduser.tfplan"` which as we know is the name of the file we pushed this output to. <br>

We can't read these files, if I click onto it we can see the **aduser.tfplan** has been created but it's in a **binary format**. 

Let's execute our plan anyway:
* Run: `terraform apply "aduser.tfplan"` <br>

![intro-to-terraform-21](./resources/intro-to-terraform-21.png) <br>

Previously when running: `terraform apply` we've had to type `yes` as terraform has compared the **Desired State** with the **Actual State** and we've seen it delete resources, add resource and change them. 

When we've executed `terraform apply "aduser.tfplan"` from our file it's just executed the pre-existing plan. 

The execution is faster but we're not making the checks so it's something to be concious of if you're applying config from **.tfplan** file. 

Let's define another **output** to see some state about our **Azure AD User** 

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
  user_principal_name = "my_iam_user_abc@emilesherrottdevops.onmicrosoft.com"
  display_name        = "my_iam_user_abc"
  mail_nickname        = "my_iam_user_abc"
  password             = "ChangeMe123!ChangeMe"
}


output "my_storage_account_versioning" {
  value = azurerm_storage_account.my_storage_account.blob_properties[0].versioning_enabled
}


output "my_storage_account_complete_details" {
  value = azurerm_storage_account.my_storage_account
}

# NEW CODE
output "my_azuread_user_complete_details" {
  value = azuread_user.my_azuread_user
}
```

* Run: `terraform apply -refresh=false`
  * `Yes` <br>

![intro-to-terraform-22](./resources/intro-to-terraform-22.png) <br>

We've got three outputs, but we can see the details for our Azure AD user.
* **id**: this is the user's **Object ID**, a GUID that uniquely identifies them within your Azure AD tenant

**ASK** <br>
Is this the same idea as an AWS **arn**? <br>
**ANSWER** <br>
Sort of, in that it's used to identify the resource, but Azure's **Object ID** is just an opaque GUID — it doesn't encode the subscription, service or resource type into the string the way an AWS **arn** does (`arn:aws:iam::725625542800:user/name`). To know what kind of object a GUID refers to, you generally need to ask Azure AD, or already know it from context (like the `resource` block that created it).

Of course, we can see the same details in the terraform console.
* `terraform console`
* `azuread_user.my_azuread_user` <br>
*TAKE A LOOK*

Also if we look in our **Known State** file: **terraform.tfstate** we can see we now have an array of **resources** from the three we've created. 

<br>
<br>

### Updating Azure AD User Details with Terraform

Let's update the details of our **Azure AD** user. <br>

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

# UPDATED CODE
resource "azuread_user" "my_azuread_user" {
  user_principal_name = "my_iam_user_def@emilesherrottdevops.onmicrosoft.com"
  display_name        = "my_iam_user_def"
  mail_nickname        = "my_iam_user_def"
  password             = "ChangeMe123!ChangeMe"
}


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


So I'll change the **abc** to a **def** across the `user_principal_name`, `display_name` and `mail_nickname`. <br>

Terraform is really integrated with the cloud providers. It knows Azure's **Storage Account** name can't be updated — as we saw before, changing it means delete and recreate. 

Terraform also knows an **Azure AD user's** identifying details can generally be updated in place. 

Let's try and see this in action. <br>

We'll run it with a shortcut. 

We know there's only one resource we want to interact with and imagine if the API had to check hundreds of resources, so we can save time and effort by building out our command to specify a target. <br>

* Syntax: `terraform apply -target=<provider><resource-type>.<terraform-resource-name>`

- So we update our `terraform apply` command with a specific target to update.
* Run: `terraform apply -target=azuread_user.my_azuread_user`

![intro-to-terraform-23](./resources/intro-to-terraform-23.png)

We can see that the **Azure AD user resource** will have its details changed in place. Again, the fact that it knows what resources can be updated and those which can't (forcing a replace) is really great, it's one of the reasons **terraform** has had such wide adoption as the tool for infrastructure Provisioning. 

* Run: `yes`

<br>
<br>

### Understanding Terraform tfstate files in depth


Let's look at the folder structure we have now.<br>

![intro-to-terraform-24](./resources/intro-to-terraform-24.png)

We have a file called:
* **terraform.tfstate**
* **terraform.tfstate.backup**

### terraform.tfstate
If we look inside **terraform.tfstate** in the **outputs** object we've got our **Azure AD User** and **Storage Account**. This is replicated in the **resources** array.

**Outputs**: contains the values of outputs defined in your Terraform configuration, in our case **main.tf** <br>

**Resources**: Contains information about the resources that Terraform is managing, including:
* resource type
* resource name
* current configuration
* metadata etc. 

#### terraform.tfstate.backup
If anyone's checked, you may have noticed that  **terraform.tfstate.backup** is contains the previous **Known State**. <br>

If we look back quickly to our **Resources Array** in **terraform.tfstate**, we can see the **Azure AD User** has the substring **def** and then if we compare that the **terraform.tfstate.backup** it's the original **abc**.

Whenever we successfully `terraform apply` what happens is the **Known State** or the state before the execution of the command, is placed into **terraform.tfstate.backup**. 

Then the state after the execution of the command overrides the old config and is placed into **terraform.tfstate**. 

So **terraform.tfstate.backup** is essentially a backup incase our provisioning gets corrupted for whatever reason, we have a stable version of our configuration available. 

One thing to know is we shouldn't make any changes to these files directly. They contain the **Known State** and the previous **Known State**. 

Let's ignore that advise right now and change the:
* **terraform.tfstate** file. I'll just rename the file to: **terraform.tfstate.001**

* **terraform.tfstate.backup** I'll rename this to: **terraform.tfstate.backup.001**

If I run `ls` you can see we're in our **01-terraform basics** folder. 

Right now there's no files **terraform** recognises as holding the value of our **Known State**. <br>

This stops **terraform** because to execute it wants to know the difference between the **Known State** and **Actual State**. 

**ASK** <br>
Any guesses about what will happen if we try to execute this code?

* Run: `terraform plan` <br>
*SHOW OUTPUT PLAN*

Basically it thinks that no resources exist because there's no **Known State** whatsoever so the **terraform plan** is accurate to terraforms knowledge. 

It wants to build out the resources defined in **main.tf**.

Let's rename the files back to how they were previously named. 

Let's see what the difference is if we execute a terraform plan command now.

* Run: `terraform plan` <br>

With this response, we can see the changes are minimal. 


It's a fair question to ask, that if the **Desired State** in a terraform apply command is checked against the **Actual State** at that point, why do we need the **Known State**. 

To answer that we should understand a bit more about how **terraform operates**.

One of the most important details in our terraform configuration is the name of the resource we choose to provide. 

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
  mail_nickname        = "my_iam_user_def"
  password             = "ChangeMe123!ChangeMe"
}

[ . . . ]
```

I'm referring to: **my_storage_account** and **my_azuread_user**. <br>

As terraform is concerned, this is the **id** for the resource. So it uses this in our **Known State**, **terraform.tfstate** file. <br>

Looking at our **my_azuread_user** resource <br>
In the **resources** array, it'll find the resource.

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
          # HERE - the Object ID, Azure AD's equivalent identifier
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

So it'll find the **resource** by its **name key** then in the **instances** array find the **id** (Object ID) to map it to the actual resource present in the cloud. Unlike our AWS example, Azure AD doesn't hand us a separate `unique_id` attribute — the `id` itself does that job.

If the **terraform.tfstate** file isn't present, the service has no way of knowing that the **my_azuread_user** name of the resource matches to this **Object ID** of the Azure AD resource in the cloud or that the resource was created through terraform.
<br>

So it'll instinctively think we'll want to create the resource or in this case our user again. 

So **terraform.tfstate** maps between the **Known State** and **Actual State** and allows terraform to perform accurate creation, updating and deletion of resources. 

We've already created **Azure AD Users** in the Portal and terraform isn't tracking these resources. 

<br>
<br>

### gitignore Terraform tfstate files


If we have project where multiple people are working on it, we know the importance of the **terraform.tfstate** file. 

We'd want to share it with the team. 

We'd want to commit it to our **GitHub** repo of course…… or would we? <br>

The problem is if we're using any secrets to create our resource or things like that, they're stored unencrypted in the **terraform.tfstate** file. We don't want it to be part of any public repositories stored on **GitHub**. 

The chosen way to handle this issue is to use something called a **remote backend**, we can store this terraform state in an **Azure Storage Account** (inside a blob container). <br>

We want to get to the point that if multiple developers were to execute this, they'll all be able to pull the latest **Known State** from the cloud and any updates are made accurately. <br> 

We'll implement this **remote backend** a little later but the learning from this is that the **terraform.tfstate** file shouldn't be publicly available. 

To begin with though, let's create a **.gitignore** file:
* From the parent of **01-terraform-basics**, **terraform** run: `touch .gitignore` <br>

**.gitignore**
```gitignore
*.tfstate
*.tfstate.backup
**/.terraform/
```
And we can add all the files not necessary for Git to track. 

<br>
<br>

### Refactoring Terraform files: Variables, Main and Outputs

Until now we've been creating everything within our **main.tf** file. <br> 

If we had hundreds of resources, this may become quite complex to maintain so let's do some refactoring and separate our config into some separate files. 

Let's create a file for our **outputs**.
* Run: `touch outputs.tf` <br>
You've probably guessed what we're going to put into it. <br>

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

*ALSO REMOVE THE OUTPUTS FROM main.tf*

Let's run: `terraform plan -refresh=false` <br>

![intro-to-terraform-25](./resources/intro-to-terraform-25.png)

As we see, no changes. <br>

Which is right, we've just separated our config, but the configuration is the same and the infrastructure is up to date. 

All we need is the **.tf** extension, beyond that the files can be named what we believe to be suitable. <br> 

Then `terraform plan/apply` looks at the **.tf** files on that level. 

* Run: `touch resources.tf` <br>
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
  mail_nickname        = "my_iam_user_def"
  password             = "ChangeMe123!ChangeMe"
}
```
*REMOVE THE COPIED CONFIG FROM main.tf*

As we go further in the course and add more **terraform configuration**, separating out our code becomes really important. 


**ASK** <br>
We're finished with these resources for now. What should we do with resources we're not using? <br>
**ANSWER** <br>
Delete them. <br>

We'll generally follow the cycle of **creating, using** then **destroying**. <br>

Because it's automated it's easy to do this, if you're studying, at the start of the day we can create resources and easily remove them at the end. 

More widely in though, it becomes easy for us to say we want to release a deployment and have our infrastructure set up in Hong Kong or Melbourne. With terraform it's easy — Azure calls these **regions**, and each has a corresponding **location** value like `southeastasia` or `australiaeast`.

Before Cloud computing, an organisation would have to go out, buy office space, order components for a server and get someone out there to physically set it up. 

Let's get rid of our resources anyway.
* Run: `terraform destory`
* Type: `yes`

So like `terraform apply` the first thing that happens is the state is refreshed
* Looks at **Known State** and identifies what resources are being managed by terraform and to be deleted
* Refresh them
* Deletes them

<br>
<br>

### Creating Terraform project for Multiple Azure AD Users

Let's start a new project, where we create multiple users. 

We'll use **Azure AD Users**.

I'm going to create a new folder, outside of our current directory so we're not destroying any code we've previously been working with. 

from **/terraform**, run: `mkdir 02-terraform-basics` -> and `cd` inside <br>
I'll want to create some basic files.
* Run: `touch main.tf`

I'm going to go ahead and copy the config from our previous **main.tf** and then the resource in **resource.tf** which related to our **Azure AD User**

**02-terraform-basics/main.tf**
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

resource "azuread_user" "my_azuread_user" {
  user_principal_name = "my_iam_user_def@emilesherrottdevops.onmicrosoft.com"
  display_name        = "my_iam_user_def"
  mail_nickname        = "my_iam_user_def"
  password             = "ChangeMe123!ChangeMe"
}
```

I'll also show you how create multiple users. <br>

**main.tf**
```tf
[ . . . ]
# UPDATED CONFIG
resource "azuread_user" "my_azuread_users" {
  # NEW CONFIG
  count                = 2
  user_principal_name  = "my_iam_user_def@emilesherrottdevops.onmicrosoft.com"
  display_name         = "my_iam_user_def"
  mail_nickname        = "my_iam_user_def"
  password              = "ChangeMe123!ChangeMe"
}
```

So one thing we've changed is the name of the **terraform identifier** <br>

From **my_azuread_user** to **my_azuread_users**, we'll pluralise it because we want many and the name should make sense.  

With **HashiCorpControlLanguage** or **HCL** it's easy to define two users. 

We define a key of **count** and the **value** we want to give it. 

Now we've provided a fixed **user_principal_name**, **display_name** and **mail_nickname**, that won't work for multiple users since the sign-in name must be unique — so we'll need to change this. 

**main.tf**
```tf
[ . . . ]
resource "azuread_user" "my_azuread_users" {
  count = 2
  # UPDATED CONFIG
  user_principal_name  = "my_iam_user_${count.index}@emilesherrottdevops.onmicrosoft.com"
  display_name         = "my_iam_user_${count.index}"
  mail_nickname        = "my_iam_user_${count.index}"
  password              = "ChangeMe123!ChangeMe"
}
```

**ASK** <br>
Where have we seen syntax like this before? <br>
**ANSWER** <br>
JavaScript template literals. <br>

The idea is similar but it's **HCL** or the HashiCorp Control Language we're using. 

So we're saying, we want two resources, we'll use the **index** of the resource to name the resource.<br> 

The first one will be created with the user principal name: **my_iam_user_0@emilesherrottdevops.onmicrosoft.com** <br>

The second one will be created with the user principal name: **my_iam_user_1@emilesherrottdevops.onmicrosoft.com**

Don't worry if you don't like the idea of using numbers to define users, we'll make users with actual names in a little bit. 

Let's execute this code. We'll need to make sure we're in the right directory. We should just see a **main.tf** file if we were to run `ls`. 

**ASK** <br>
What do we need to do now? <br>
**ANSWER** <br>
* Run: `terraform init` <br>

We need the Azure AD Provider and the Plugins for each project. 

Let's now execute, I'm not going to plan, but apply <br>
* Run: `terraform apply` <br>

So it's creating our two resources for us, we didn't have to specify the creation of another individual resource object. <br>
* Type: `yes`

If we go to the **Azure Portal**, within **Azure Active Directory > Users** we'll see the users created. 

If we wanted to increase the number of resources to 3. We update our config, specifically the value of count. 

**main.tf**
```
[ . . . ]
resource "azuread_user" "my_azuread_users" {
  # UPDATED CONFIG
  count = 3
  user_principal_name  = "my_iam_user_${count.index}@emilesherrottdevops.onmicrosoft.com"
  display_name         = "my_iam_user_${count.index}"
  mail_nickname        = "my_iam_user_${count.index}"
  password              = "ChangeMe123!ChangeMe"
}
```

* Run: `terraform apply` <br>

![intro-to-terraform-26](./resources/intro-to-terraform-26.png)

Again all we're saying is we want **3** resources. We haven't told terraform to add 1. This just to reiterate is the **declarative nature** of terraform. 

Terraform is able to check the state, then it's let us know we need to create one more resource to fufill our **desired state**

* Run: `yes`

<br>
<br>

### Playing with Terraform Commands: fmt, show and console

Let's look at some other terraform commands. <br>

We'll start in the console: `terraform console`

**ASK** <br>
Can anyone remember the syntax to access information about one of our resources? <br>
**ANSWER** <br>
`<provider>_<resource-type>.<chosen_resource_name>`

So let's run: `azuread_user.my_azuread_users` <br>

![intro-to-terraform-27](./resources/intro-to-terraform-27.png)

So we can see the datatype here is an **array** or a **list**. <br>

Technically it's a **list** which is similar to arrays. If you've used Python before, python uses lists instead of arrays but ultimately we've got an ordered set of data.

Within this list we've got multiple objects of our three different **Azure AD** users. <br>

As we've seen we can access specific keys on that object. <br>

* `azuread_user.my_azuread_users[0].id`

**ASK** <br>
What's this returning? <br>
**ANSWER** <br>
The **Object ID** value on the first indexed **Azure AD User** <br>

Let's take a look at it: **"id" = "3fa85f64-5717-4562-b3fc-2c963f66afa6"** <br>
Unlike an AWS **arn**, this doesn't break down into readable segments — it's just a GUID Azure has generated to uniquely identify this object within our tenant. If we want a human-readable identifier for the same user, that's what `user_principal_name` is for:
* `azuread_user.my_azuread_users[0].user_principal_name` -> **"my_iam_user_0@emilesherrottdevops.onmicrosoft.com"**

Generally the **terraform console** is handy so we can dig around, see information and then define an **output** in our **tf** files if we want regular access. 

#### `terraform validate`
Let's exit the **terraform console** and introduce an error into our code. <br>

**main.tf**
```
[ . . . ]
resource "azuread_user" "my_azuread_users" {
  count = 3
  # UPDATED CONFIG
  user_principal_name  = "my_iam_user_${count.inde}@emilesherrottdevops.onmicrosoft.com"
  display_name         = "my_iam_user_${count.index}"
  mail_nickname        = "my_iam_user_${count.index}"
  password              = "ChangeMe123!ChangeMe"
}
```

I've changed the attribute of **index** to **inde** which won't be valid. 

* Let's run: `terraform validate` <br>

![intro-to-terraform-28](./resources/intro-to-terraform-28.png) <br>

Obviously really helpful to have this functionality which will tell you what the errors are and in this case even offer a solution. 

Let's correct that mistake and make another. <br>

**main.tf**
```tf
[ . . . ]
# UPDATED CONFIG
resource "azuread_use" "my_azuread_users" {
  count = 3
  # UPDATED CONFIG
  user_principal_name  = "my_iam_user_${count.index}@emilesherrottdevops.onmicrosoft.com"
  display_name         = "my_iam_user_${count.index}"
  mail_nickname        = "my_iam_user_${count.index}"
  password              = "ChangeMe123!ChangeMe"
}
```

So now I've fixed the attribute name, but now made a mistake in the **resource name**. <br>
`azuread_user` -> `azuread_use`

Will terraform validate pick it up. <br>
* `terraform validate` <br>

![intro-to-terraform-29](./resources/intro-to-terraform-29.png)

*FIX THE ISSUE*

#### `terraform fmt`
Through this course I've used `OPTION + SHIFT + F` to format my files. The terraform extension has enabled me to continue doing that but if I indent a line in **main.tf** I can simply run `terraform fmt` to format the file properly. Helpful! 

*DEMONSTRATE*

When writing in **HCL** (HashiCorp Control Language) ideally we'll use 2 spaces to indent. <br>

`terraform fmt` will perform this on every file in the location performing the command, handy to run before committing code.

#### `terraform show`
Another handy command is `terraform show` <br>

* Run: `terraform show` <br>

![intro-to-terraform-30](./resources/intro-to-terraform-30.png)

It shows what's in the **Known State**. The purpose is that it's a bit more readable than if we wanted to read the **terraform.tfstate** file directly or use the **outputs** we've seen earlier to show files. 

#### `terraform apply <options>`
There's also lots of **terraform apply** options which you can see in a resources document in the student facing repo.

<br>
<br>

### Recovering from Errors with Terraform

One of the important things about creating resources in the cloud is recovering from errors. If we've made a mistake how quickly can we recover from it. 

How easy is it to update the script and fix the problem? With terraform it's quite easy. 

Let's make a mistake <br>

**main.tf**
```tf
[ . . . ]
resource "azuread_user" "my_azuread_users" {
  count = 3
  # UPDATED CONFIG
  user_principal_name  = "my_iam_user@emilesherrottdevops.onmicrosoft.com"
  display_name         = "my_iam_user"
  mail_nickname        = "my_iam_user"
  password              = "ChangeMe123!ChangeMe"
}
```

So I've made an error, we're still trying to create 3 resources but we've not passed in the count and the index attribute to achieve this. 

Terraform doesn't know we can't have multiple Azure AD users with the same **user_principal_name**. Let's see how it reacts and how we can recover. <br>

Let's run: `terraform apply` <br>

![intro-to-terraform-31](./resources/intro-to-terraform-31.png) <br>

The start of the response shows the three individual resources being refreshed in parallel. <br> 

Terraform knows they're all independent. <br> 
![intro-to-terraform-32](./resources/intro-to-terraform-32.png) <br>

It's also saying 3 to change. What's it's doing is trying to change the details of each currently different **Azure AD User** to: the same value: **my_iam_user@emilesherrottdevops.onmicrosoft.com**

* Type: `yes` <br>

![intro-to-terraform-33](./resources/intro-to-terraform-33.png) <br>

So it's updated 1 but when it's tried to update the last two it's failed. <br>
We can't have two users with the same **user_principal_name** so that's why we're seeing this. 

If we go to **Azure Portal: Azure Active Directory > Users** we can see we still have the 3 users but only 1 has had their details updated. <br>

There's no automatic **rollback**, the error has carried over to the **Actual State** and hasn't been reversed when **terraform** saw the error. Something to remember. 

Let's undo the change in **main.tf** and run: `terraform apply` after. <br>

**main.tf**
```tf
[ . . . ]
resource "azuread_user" "my_azuread_users" {
  count = 3
  # UPDATED CONFIG
  user_principal_name  = "my_iam_user_${count.index}@emilesherrottdevops.onmicrosoft.com"
  display_name         = "my_iam_user_${count.index}"
  mail_nickname        = "my_iam_user_${count.index}"
  password              = "ChangeMe123!ChangeMe"
}
```

* Run: `terraform apply` <br>

![intro-to-terraform-34](./resources/intro-to-terraform-34.png)

Terraform compares the **Desired State** with the **Actual State** and realises that it only needs to update one resource to match the state. 

* Run: `yes`

And the error will be fixed. 

<br>
<br>

### Understanding Variables in Terraform


Let's take a look and see how we can start to use **variables** in terraform. <br>

Until now we've been using **constants**. 

Let's start by creating a simple variable to store the value of our **Azure AD User name prefix** <br>

So we'll store **my_iam_user** in a variable, hopefully giving us the ability to change that **variable** later and also all the **Azure AD users** along with it. 

**main.tf**
```tf
[ . . . ]

# NEW CONFIG
variable "iam_user_name_prefix" {
  default = "my_iam_user"
}

resource "azuread_user" "my_azuread_users" {
  count = 3
  # UPDATED CONFIG
  user_principal_name  = "${var.iam_user_name_prefix}_${count.index}@emilesherrottdevops.onmicrosoft.com"
  display_name         = "${var.iam_user_name_prefix}_${count.index}"
  mail_nickname        = "${var.iam_user_name_prefix}_${count.index}"
  password              = "ChangeMe123!ChangeMe"
}
```

So we define a variable, with the name **iam_user_name_prefix** and give it a **default** value of **my_iam_user**. <br>

Then in the **resource** config we pass in that variable, again in a similar way to JavaScript string interpolation with template literals. <br>

Note, we have to pass in the key word **var** and then the name of the variable. 

If we try and execute this, we'd hope there's no change. <br>

* Run: `terraform apply -refresh=false` <br>

![intro-to-terraform-35](./resources/intro-to-terraform-35.png)

Nice so it's being recognised as the same configuration, which it is. 

If we go to terraform console. 
* Run: `terraform console`
* Run: `var.iam_user_name_prefix` <br>

![intro-to-terraform-36](./resources/intro-to-terraform-36.png) <br>

Straight forward… it outputs the value of the **variable**.  <br>
* Run: `ctrl + c`

Let's look at the different things we can do with our **variable config** <br>

**main.tf**
```tf
variable "iam_user_name_prefix" {
  # NEW CONFIG
  type = string
  default = "my_iam_user"
}

resource "azuread_user" "my_azuread_users" {
  count = 3
  user_principal_name  = "${var.iam_user_name_prefix}_${count.index}@emilesherrottdevops.onmicrosoft.com"
  display_name         = "${var.iam_user_name_prefix}_${count.index}"
  mail_nickname        = "${var.iam_user_name_prefix}_${count.index}"
  password              = "ChangeMe123!ChangeMe"
}
```

We've defined a data type the variable is allowed to be.<br> 

There's other values we can specify for **type**:
* any (default)
* number
* bool
* list -> like arrays
* tuple -> like arrays but immutable 
* set -> indexed values, only unique values
* map -> set of key value pairs like objects


If I were to choose a type of number and pass a string, it wouldn't be happy. <br>

**main.tf**
```tf
variable "iam_user_name_prefix" {
  # UPDATED CONFIG
  type = number
  default = "my_iam_user"
}

resource "azuread_user" "my_azuread_users" {
  count = 3
  user_principal_name  = "${var.iam_user_name_prefix}_${count.index}@emilesherrottdevops.onmicrosoft.com"
  display_name         = "${var.iam_user_name_prefix}_${count.index}"
  mail_nickname        = "${var.iam_user_name_prefix}_${count.index}"
  password              = "ChangeMe123!ChangeMe"
}
```

**ASK** <br>
How can we check if our configuration is sound? <br>
**ASK** <br>
`terraform validate`

* Run:  `terraform validate` <br>
![intro-to-terraform-37](./resources/intro-to-terraform-37.png)

Let's return it back to **string** <br>

**main.tf**
```tf
variable "iam_user_name_prefix" {
  # UPDATED CONFIG
  type = string
  default = "my_iam_user"
}

resource "azuread_user" "my_azuread_users" {
  count = 3
  user_principal_name  = "${var.iam_user_name_prefix}_${count.index}@emilesherrottdevops.onmicrosoft.com"
  display_name         = "${var.iam_user_name_prefix}_${count.index}"
  mail_nickname        = "${var.iam_user_name_prefix}_${count.index}"
  password              = "ChangeMe123!ChangeMe"
}
```

We'll look at the other types later. 

What would happen if we commented out the default value? <br>

**main.tf**
```tf
variable "iam_user_name_prefix" {
  type = string
  # UPDATED CONFIG
  # default = "my_iam_user"
}

resource "azuread_user" "my_azuread_users" {
  count = 3
  user_principal_name  = "${var.iam_user_name_prefix}_${count.index}@emilesherrottdevops.onmicrosoft.com"
  display_name         = "${var.iam_user_name_prefix}_${count.index}"
  mail_nickname        = "${var.iam_user_name_prefix}_${count.index}"
  password              = "ChangeMe123!ChangeMe"
}
```

* Run: `terraform validate`<br>

![intro-to-terraform-38](./resources/intro-to-terraform-38.png)

It'd return valid…. Interesting <br>
Let's see how far we can push it. <br>

* Run: `terraform apply -refresh=false` <br>

![intro-to-terraform-39](./resources/intro-to-terraform-39.png)

Ok, so here it asks us to enter a value for the variable which is clever. <br>

* Type: `my_iam_user` <br>
![intro-to-terraform-40](./resources/intro-to-terraform-40.png) <br>

And it's told me **No changes** are needed. 

Let's try something else, we'll make the value of **count = 1** and run `terraform apply -refresh=false` again. <br>

**main.tf**
```tf
variable "iam_user_name_prefix" {
  type = string
  # default = "my_iam_user"
}

resource "azuread_user" "my_azuread_users" {
  # UPDATED CONFIG
  count = 1
  user_principal_name  = "${var.iam_user_name_prefix}_${count.index}@emilesherrottdevops.onmicrosoft.com"
  display_name         = "${var.iam_user_name_prefix}_${count.index}"
  mail_nickname        = "${var.iam_user_name_prefix}_${count.index}"
  password              = "ChangeMe123!ChangeMe"
}
```

* Run: `terraform apply -refresh=false` <br>
So it asks the our variable name again: `my_iam_user`<br>

Again, really just trying to show you it's declarative nature, we say we want 1 **Azure AD user** so terraform see's there's 3 and it knows to delete two. 

* Run: `yes`

*UNCOMMENT THE DEFAULT VALUE* <br>

**main.tf**
```tf
variable "iam_user_name_prefix" {
  type = string
  # UPDATED CONFIG
  default = "my_iam_user"
}

resource "azuread_user" "my_azuread_users" {
  count = 1
  user_principal_name  = "${var.iam_user_name_prefix}_${count.index}@emilesherrottdevops.onmicrosoft.com"
  display_name         = "${var.iam_user_name_prefix}_${count.index}"
  mail_nickname        = "${var.iam_user_name_prefix}_${count.index}"
  password              = "ChangeMe123!ChangeMe"
}
```

We can also **export** the environment variable like we did our **Azure credentials** <br>

* From VSCode terminal run: `export TF_VAR_iam_user_name_prefix=FROM_ENV_VARIABLE_IAM_PREFIX`
  * **Terraform env variables** need to start with: `TF_VAR_<regular-name>`
* Check with: `export`
* Run: `terraform plan -refresh=false`<br>

![intro-to-terraform-41](./resources/intro-to-terraform-41.png)

So we can see the **name change**, the value of the **environment variable** is overriding our default value. <br>

Let's delete that **environment variable**
- Mac: `unset TF_VAR_iam_user_name_prefix`
- Windows: `setx TF_VAR_iam_user_name_prefix`

If you're seeing potential changes from `terraform plan` and can't understand why maybe it's an environment variable. 

We can overwrite our default values another way too.<br>
We create a file called **terraform.tfvars**

* Run: `touch terraform.tfvars`

And we can define our values in here.

**terraform.tfvars**
```tf
# NEW CODE
iam_user_name_prefix = "VALUE_FROM_TERRAFORM_TFVARS"
```

* Run: `terraform plan -refresh=false` <br>
![intro-to-terraform-42](./resources/intro-to-terraform-42.png)

It'll overwrite the default value in **main.tf** and use the one from **terraform.tfvars**

In order of priority
1. **terraform.tfvars**
2. **environment varibales**
3. **default values**

We can also provide a variable name from the CLI, similar to how we passed **port** information when our running Docker containers. 

* Run:
* `terraform plan -refresh=false -var="iam_user_name_prefix"=VALUE_FROM_CLI` <br>

![intro-to-terraform-43](./resources/intro-to-terraform-43.png)

So it's used the value from the CLI. So it's got priority over our **var** file. 

We were looking at options before in relation to the`terraform apply`command

There's another called a **varfile**

* Syntax: `terraform apply -var-file="<file-name>.tfvars"`

That'll pick up variables defined in a specific file over others. 

#### Why are variables important?
The reason we're looking at variables so much is because the same terraform script will be used in multiple environments. 

In DevOps you'd create your:
* Development Resources
* Test Resources
* Production Resources

We could also use a variable for the location and quickly create resources in a new region. 


What you may see is people using a **variable** with the name of **environment** <br>

**main.tf**
```tf
[ . . . ]

# NEW CONFIG
variable "environment" {
  default = "dev"
  
}

variable "iam_user_name_prefix" {
  type = string
  default = "my_iam_user"
}

resource "azuread_user" "my_azuread_users" {
  count = 1
  # UPDATED CONFIG
  user_principal_name  = "${var.environment}_${var.iam_user_name_prefix}_${count.index}@emilesherrottdevops.onmicrosoft.com"
  display_name         = "${var.environment}_${var.iam_user_name_prefix}_${count.index}"
  mail_nickname        = "${var.environment}_${var.iam_user_name_prefix}_${count.index}"
  password              = "ChangeMe123!ChangeMe"
}
```

In the **environment** config we could define a default value of "dev" then we can use this with any resources we create. <br>

So in this case all we're doing is adding **dev** to the name of our Azure AD user. This basically helps us keep track of resources and if it's for a **dev environment**, testing, or anything else. 

More practically we may want to provision one server in a **test** environment and many more for **production**

Run: `terraform plan -refresh=false -var="iam_user_name_prefix"=VALUE_FROM_CLI`

Then when executing scripts, we have more control over what environments they're for. We'll go onto how we can configure environments with terraform more later. <br>

Just know that variables are important because they make our scripts dynamic. 

I'll undo these changes in **main.tf**

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

# DELETE CONFIG FOR VARIABLE: 'environment'

variable "iam_user_name_prefix" {
  type = string
  default = "my_iam_user"
}

resource "azuread_user" "my_azuread_users" {
  count = 1
  # UPDATED CONFIG
  user_principal_name  = "${var.iam_user_name_prefix}_${count.index}@emilesherrottdevops.onmicrosoft.com"
  display_name         = "${var.iam_user_name_prefix}_${count.index}"
  mail_nickname        = "${var.iam_user_name_prefix}_${count.index}"
  password              = "ChangeMe123!ChangeMe"
}
```

I'm just going to run: `terrrafrom validate` to make sure everything is fine. 

<br>
<br>

### Creating Terraform Project for understanding List and Map

Let's move onto some new things. We'll make a new folder
* **from parent of current repo** <br>
Run: `mkdir 03-lists-and-sets` <br>

So as it looks, we'll be taking a look at lists and sets. 

What we're going to want to do is create a list of **usernames**, and create **Azure AD users** with that list. So we can move away from the name **my_iam_user**. 

So I'm going to copy over the main config we need to get us off the ground. 

I'll have to create a new **main.tf** file: `touch main.tf`

**03-lists-and-sets / main.tf**
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
  count = 1
  user_principal_name  = "${var.iam_user_name_prefix}_${count.index}@emilesherrottdevops.onmicrosoft.com"
  display_name         = "${var.iam_user_name_prefix}_${count.index}"
  mail_nickname        = "${var.iam_user_name_prefix}_${count.index}"
  password              = "ChangeMe123!ChangeMe"
}
```

So we should have some errors <br>
**ASK** <br>
Why are we getting errors now? <br>
**ANSWER** <br>
Our variable isn't defined for **iam_user_name_prefix**

Let's ignore this for now and define another **variable** with our list of names <br>

**main.tf**
```tf
[ . . . ]

# NEW CONFIG
variable "names" {
  default = ["emile", "romeo", "sarah"]
}

resource "azuread_user" "my_azuread_users" {
  count = 1
  user_principal_name  = "${var.iam_user_name_prefix}_${count.index}@emilesherrottdevops.onmicrosoft.com"
  display_name         = "${var.iam_user_name_prefix}_${count.index}"
  mail_nickname        = "${var.iam_user_name_prefix}_${count.index}"
  password              = "ChangeMe123!ChangeMe"
}
```

So these are the names I want. <br>
The required provider config can remain the same that's fine. 

The count we'll need to change. <br>
In JavaScript if you're looping over an array we use a property of **length**. <br> 

**HCL** (HashiCorp Control Language) is similar but uses a different syntax. 

Let's also go ahead and change our **name fields** for the **azuread_user**, we want to pick them up from the **names** variable. 

**main.tf**
```tf
[ . . . ]

variable "names" {
  default = ["emile", "romeo", "sarah"]
}

resource "azuread_user" "my_azuread_users" {
  # NEW CONFIG
  count = length(var.names)
  # UPDATED CONFIG
  user_principal_name  = "${var.names[count.index]}@emilesherrottdevops.onmicrosoft.com"
  display_name         = var.names[count.index]
  mail_nickname        = var.names[count.index]
  password              = "ChangeMe123!ChangeMe"
}
```

So we know arrays and lists are indexed from 0. <br>
Also **count** will evaluate as 3, so the indexes will run from 0 to 2. Very similar to what we already know. 

**ASK** <br>
Remind me of the command to bring us the provider and its plugins? <br>
**ANSWER** <br>
`terrafrom init` <br>

* Run: `terraform init`
Once complete <br>
* Run: `terraform apply` <br>

*WHILST RUNNING SAY*


We're creating several projects to consolidate getting access to these providers. 

Typically when working with terraform, we might be working with several applications, each one of them might have different lifecycles. 

A **Storage Account** may have a different lifecycle than a **deployment**. <br>

Each can have their own independent management and update strategies. Typically a **Storage Account** may have a longer lifecycle compared to a deployment. As they can store:
* logs
* assets
* backups <br>

And they often persist over the lifetime of an application as deployments evolve. 

So we'll need to have separate configurations inside this wider project to manage these different terraform configurations. 

Anyway, it looks good. We'll create three new users and we can see the details for each of them matches one of the items in the list. 

Like arrays our list has positional values so we can see: **azuread_user.my_azuread_users[0].display_name** would evaluate to **emile**, the string in our list. <br>

![intro-to-terraform-44](./resources/intro-to-terraform-44.png)

* Run: `yes`

Let's go into the terraform console. <br>
* Run: `terraform console`
* `var.names` <br>

![intro-to-terraform-45](./resources/intro-to-terraform-45.png)

* `var.names[1]` <br>
![intro-to-terraform-46](./resources/intro-to-terraform-46.png)

There's a few more pieces of functionality we can use with our **collections**.

Collections are our **lists, sets, maps and tuples**

* `length(var.names)` <br>
![intro-to-terraform-47](./resources/intro-to-terraform-47.png)

* `reverse(var.names)`<br>
**ASK** <br>
What do you expect to happen? <br>
![intro-to-terraform-48](./resources/intro-to-terraform-48.png) <br>
**ANSWER** <br>
It reverse the list. <br>

* `distinct(var.names)`
* If we have duplicate values we can use **distinct** to remove them. <br>
![intro-to-terraform-49](./resources/intro-to-terraform-49.png)


We can see it has wrapped our **list** with the functionality **tolist** this is because we're performing a mutable command by only returning unique values.<br>

So it's ensuring the output is a list where it's not relevant to other collections. 

* `toset(var.names)` <br>
![intro-to-terraform-50](./resources/intro-to-terraform-50.png) <br>

Sets only contain unique values which is similar. <br>
Sets are immutable though so we can't change a set from this point. 

* `concat(var.names, ["tom", "astha"])` <br>
![intro-to-terraform-51](./resources/intro-to-terraform-51.png) <br>

Again, like JavaScript we can concatenate two lists into one.

* `contains(var.names, "simon")` <br>
![intro-to-terraform-52](./resources/intro-to-terraform-52.png)

* `sort(var.names)` <br>
![intro-to-terraform-53](./resources/intro-to-terraform-53.png)

We can't really see what's happening here. They were already in alphabetical order but if they weren't, they are now!

Some other things we can do with **collections** <br>
* `range(10)` <br>
![intro-to-terraform-54](./resources/intro-to-terraform-54.png) <br>
Outputs a list **10** items long

* `range(1,12)` <br>
![intro-to-terraform-55](./resources/intro-to-terraform-55.png) <br>
This gives us a list starting at **1** which runs up to but not inclusive of **12**

* `range(1,12,3)` <br>
![intro-to-terraform-56](./resources/intro-to-terraform-56.png) <br>
The third argument is a **step** so with each new item, we add 3. 

With all of these things it's not a bad idea to read documentation to see what else is available if there's a valid usecase. 

<br>
<br>

### Adding Elements: Problem with Terraform Lists

Let's do some more things with our lists. We've got our variable defined in **main.tf**, let's add an element to the beginning of the list and see what happens. 

**main.tf**
```tf
[ . . . ]

variable "names" {
  # UPDATED CONFIG
  default = ["simon", "emile", "romeo", "sarah"]
}

resource "azuread_user" "my_azuread_users" {
  count = length(var.names)
  user_principal_name  = "${var.names[count.index]}@emilesherrottdevops.onmicrosoft.com"
  display_name         = var.names[count.index]
  mail_nickname        = var.names[count.index]
  password              = "ChangeMe123!ChangeMe"
}
```

* Run: `terraform apply` <br>
![intro-to-terraform-57](./resources/intro-to-terraform-57.png)

So it's saying **1 to add... but 3 to change**, as far as we can see we've only added one name the the list so why are we changing 3?

From the output we can see, 
* **emile**: formerly indexed at 0 is now at 1
  * **simon** is now claiming the `azuread_user.my_azuread_users[0]` spot.
* everyone else is being pushed back.
* we didn't however have a resource in position 4 or `azuread_user.my_azuread_users[3]`
  * terraform thinks this is the new resource and it'll be created with the name **sarah**

When we create resources based off of **lists** the way they're stored is as a **list**. The problem with this is that each resource is indexed.  

If we look inside **terraform.tfstate** we can see each **resource** built from a **list** has a key of **index_key**

**terraform.tfstate**
```tf
{
  "version": 4,
  "terraform_version": "1.7.3",
  "serial": 4,
  "lineage": "54b7fa36-86a1-b04d-89da-da0a20a18ded",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      "type": "azuread_user",
      "name": "my_azuread_users",
      "provider": "provider[\"registry.terraform.io/hashicorp/azuread\"]",
      "instances": [
        {
          # HERE
          "index_key": 0,
          "schema_version": 0,
          "attributes": {
            "id": "5c3d1a2b-8f4e-4b6a-9c7d-1e2f3a4b5c6d",
            "user_principal_name": "emile@emilesherrottdevops.onmicrosoft.com",
            "display_name": "emile",
            "mail_nickname": "emile"
          },
          "sensitive_attributes": [],
          "private": "bnVsbA=="
        },
        {
          # HERE
          "index_key": 1,
          "schema_version": 0,
          "attributes": {
            "id": "7e2b4c6a-3d5e-4a6b-8c7d-1e2f3a4b5c6e",
            "user_principal_name": "romeo@emilesherrottdevops.onmicrosoft.com",
            "display_name": "romeo",
            "mail_nickname": "romeo"
          },
          "sensitive_attributes": [],
          "private": "bnVsbA=="
        },
        {
          # HERE
          "index_key": 2,
          "schema_version": 0,
          "attributes": {
            "id": "9a1c3e5f-7b2d-4a6c-8e9f-2d4f6a8b1c3e",
            "user_principal_name": "sarah@emilesherrottdevops.onmicrosoft.com",
            "display_name": "sarah",
            "mail_nickname": "sarah"
          },
          "sensitive_attributes": [],
          "private": "bnVsbA=="
        }
      ]
    }
  ],
  "check_results": null
}
```

When we modify the **list** and run `terraform apply` it compares the element at zero, with the resource indexed at 0. <br>

It doesn't recognise that all our previous names are still there and only **simon** has been added. 

To solve this we need to store these values in a different way. 

First let's destroy the resources we've made. 
* Run: `terraform destory`
* Run: `yes`

#### for each
We're going to solve this with a loop

**main.tf**
```tf
[ . . . ]

variable "names" {
  default = ["simon", "emile", "romeo", "sarah"]
}

resource "azuread_user" "my_azuread_users" {
  # UPDATED CODE
  # commented out
  #   count = length(var.names)
  #   user_principal_name = "${var.names[count.index]}@emilesherrottdevops.onmicrosoft.com"
  # NEW CONFIG
  for_each             = toset(var.names)
  user_principal_name  = "${each.value}@emilesherrottdevops.onmicrosoft.com"
  display_name         = each.value
  mail_nickname        = each.value
  password              = "ChangeMe123!ChangeMe"
}
```

So I've commented out the previous lines of code. <br>

Then we state **for_each** should equal **var.names**, we can only perform a **for_each** on unique sets of data and we know **lists** can store duplicates.<br>
 
 We resolve this by setting our `var.names` to a **set**. 

In a **for_each**, we don't specify each item using **i** but as you can see the key word **each** and we'll choose **each value** in our set. 

* Run: `terraform apply` <br>
![intro-to-terraform-58](./resources/intro-to-terraform-58.png)

So we can see we're creating 4 resources. 

We can also see the order doesn't match that of the **list** we originally defined. <br>

**sets** in Terraform and other programming languages are data structures that prioritise uniqueness, not order. They're implemented using something called a **hash table** which doesn't track order. <br>

**OPTIONAL SIDE TANGENT**
* **Hashing**: each element in a set is hashed, which converts the element into a unique numerical code or a fingerprint
* **Storage**: the hashed values serve as an address in a table, the actual elements of the set are stored at these addresses based on the hash value
* **Lookup**: when we check if an element exists, the hash table points to the corresponding address based on the elements fingerprint. 
This allows for fast searches and also limits orders being retained. <br>
**END SIDE TANGENT**

* Run: `yes`

In **terraform.tfstate** we can see that the **index_key** is no longer a number but the **string** of the **set**

**terraform.tfstate**
```tf
{
  "version": 4,
  "terraform_version": "1.7.3",
  "serial": 13,
  "lineage": "54b7fa36-86a1-b04d-89da-da0a20a18ded",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      "type": "azuread_user",
      "name": "my_azuread_users",
      "provider": "provider[\"registry.terraform.io/hashicorp/azuread\"]",
      "instances": [
        {
          # HERE
          "index_key": "emile",
          "schema_version": 0,
          "attributes": {
            "id": "5c3d1a2b-8f4e-4b6a-9c7d-1e2f3a4b5c6d",
            "user_principal_name": "emile@emilesherrottdevops.onmicrosoft.com",
            "display_name": "emile",
            "mail_nickname": "emile"
          },
          "sensitive_attributes": [],
          "private": "bnVsbA=="
        },
        {
          # HERE
          "index_key": "romeo",
          "schema_version": 0,
          "attributes": {
            "id": "7e2b4c6a-3d5e-4a6b-8c7d-1e2f3a4b5c6e",
            "user_principal_name": "romeo@emilesherrottdevops.onmicrosoft.com",
            "display_name": "romeo",
            "mail_nickname": "romeo"
          },
          "sensitive_attributes": [],
          "private": "bnVsbA=="
        },
        {
          # HERE
          "index_key": "sarah",
          "schema_version": 0,
          "attributes": {
            "id": "9a1c3e5f-7b2d-4a6c-8e9f-2d4f6a8b1c3e",
            "user_principal_name": "sarah@emilesherrottdevops.onmicrosoft.com",
            "display_name": "sarah",
            "mail_nickname": "sarah"
          },
          "sensitive_attributes": [],
          "private": "bnVsbA=="
        },
        {
          # HERE
          "index_key": "simon",
          "schema_version": 0,
          "attributes": {
            "id": "1b3d5f7a-9c2e-4b6a-8d1f-3e5f7a9b2c4d",
            "user_principal_name": "simon@emilesherrottdevops.onmicrosoft.com",
            "display_name": "simon",
            "mail_nickname": "simon"
          },
          "sensitive_attributes": [],
          "private": "bnVsbA=="
        }
      ]
    }
  ],
  "check_results": null
}
```


So if we make an addition to our original **collection** <br>

**main.tf**
```tf
[ . . . ]

variable "names" {
  # UPDATED CONFIG
  default = ["astha", "simon", "emile", "romeo", "sarah"]
}

resource "azuread_user" "my_azuread_users" {
  #   count = length(var.names)
  #   user_principal_name = "${var.names[count.index]}@emilesherrottdevops.onmicrosoft.com"
  for_each             = toset(var.names)
  user_principal_name  = "${each.value}@emilesherrottdevops.onmicrosoft.com"
  display_name         = each.value
  mail_nickname        = each.value
  password              = "ChangeMe123!ChangeMe"
}
```

We've added a new name at the beginning.

* Let's run: `terraform apply`<br>
![intro-to-terraform-59](./resources/intro-to-terraform-59.png)

It's not changing any resources, just creating one more. <br>
* Run: `yes`

If we delete a few values from our **collection** it should behave fairly well too. <br>

**main.tf**
```tf
[ . . . ]

variable "names" {
  # UPDATED CONFIG
  default = ["emile", "sarah"]
}

resource "azuread_user" "my_azuread_users" {
  #   count = length(var.names)
  #   user_principal_name = "${var.names[count.index]}@emilesherrottdevops.onmicrosoft.com"
  for_each             = toset(var.names)
  user_principal_name  = "${each.value}@emilesherrottdevops.onmicrosoft.com"
  display_name         = each.value
  mail_nickname        = each.value
  password              = "ChangeMe123!ChangeMe"
}
```

* Run: `terraform apply` <br>
![intro-to-terraform-60](./resources/intro-to-terraform-60.png)

Terraform is able to recognise the individual resources and successfully remove them.
* Run: `yes`

Using **sets** is advised, especially when we think the position of our data isn't important. 

<br>
<br>

### Creating Terraform project for learning Terraform Maps

So let's go and move onto another configuration with terraform. <br>

We'll switch on from **lists and sets** and move onto another **collection: maps**. 

Let's destroy our previous resources though
* Run: `terraform destroy`
* Run: `yes`

**Azure AD Users** don't really cost us anything but **Load Balancers** or **Virtual Machines** do so it's good to get into the habit of removing resources right away. 

**from /parent-directory**: run: `mkdir 04-maps` and **cd inside** 

* `touch main.tf`

Copy over config from **03-lits-and-sets / main.tf**

**04-maps / main.tf**
```tf
# NEW CONFIG
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
  default = ["emile", "sarah"]
}

resource "azuread_user" "my_azuread_users" {
  for_each             = toset(var.names)
  user_principal_name  = "${each.value}@emilesherrottdevops.onmicrosoft.com"
  display_name         = each.value
  mail_nickname        = each.value
  password              = "ChangeMe123!ChangeMe"
}
```

Get access to the provider: `terraform init`

This is our starting point, what we want to do is store some more information about our user names. Maybe their department, their country. Ultimately we want to see how we work with **maps**

**maps.tf**
```tf
[ . . . ]

variable "names" {
  # UPDATED CONFIG
  default = {"emile", "sarah"}
}

resource "azuread_user" "my_azuread_users" {
  for_each             = toset(var.names)
  user_principal_name  = "${each.value}@emilesherrottdevops.onmicrosoft.com"
  display_name         = each.value
  mail_nickname        = each.value
  password              = "ChangeMe123!ChangeMe"
}
```

So the first thing we do to switch from 
* square brackets **[ ]** -> to:
* curly braces **{ }**

**Maps** also follow a pattern of key value pairs, like our JavaScript objects. The key is going to be the name and we'll pass the country as the value. 

**maps.tf**
```tf
[ . . . ]

variable "names" {
  # UPDATED CONFIG
  default = {
    emile: "England",
    sarah: "France"
  }
}

resource "azuread_user" "my_azuread_users" {
  for_each             = toset(var.names)
  user_principal_name  = "${each.value}@emilesherrottdevops.onmicrosoft.com"
  display_name         = each.value
  mail_nickname        = each.value
  password              = "ChangeMe123!ChangeMe"
}
```

So we can see the key doesn't need to be a string but our values can be. 

Our **varibale names** is now a **map**, let's read from that variable.

* `terraform console`

* `var.names` <br>
![intro-to-terraform-61](./resources/intro-to-terraform-61.png)

**ASK** <br>
What will be responded if I run: `var.names.sarah` <br>
**ANSWER** <br>
"France" <br>

* `var.names.sarah` <br>
![intro-to-terraform-62](./resources/intro-to-terraform-62.png) <br>

* `var.names["sarah"]` <br>
Same thing using different syntax <br>
![intro-to-terraform-63](./resources/intro-to-terraform-63.png) <br>

* `keys(var.names)` <br>
If we want to find the keys of the **map** <br>
![intro-to-terraform-64](./resources/intro-to-terraform-64.png) <br>

* `values(var.names)` <br>
If we just want the values instead <br>
![intro-to-terraform-65](./resources/intro-to-terraform-65.png) <br>

* `lookup(var.names, "emile")` <br>
We can use **lookup**, pass a second argument of the key to try and find the corresponding value <br>
![intro-to-terraform-66](./resources/intro-to-terraform-66.png) <br>

Let's update the **resource** to use our **map**. 

**main.tf**
```tf
[ . . . ]

# UPDATED CONFIG
 variable "users" {
  default = {
    emile : "England",
    sarah : "France"
  }
}

resource "azuread_user" "my_azuread_users" {
  # UPDATED CONFIG
  for_each = var.users
  # Still we define a user for each key
  # We use the key of the map
  user_principal_name  = "${each.key}@emilesherrottdevops.onmicrosoft.com"
  display_name         = each.key
  mail_nickname        = each.key
  password              = "ChangeMe123!ChangeMe"
  # we can also define a country directly, as a native attribute
  # rather than a generic tag — Azure AD models this as a first-class field
  country               = each.value
}
```


So firstly, our **variable**, name was no longer accurate as we contain more information than just **"names"**. <br>
So I'll update it to **"users"** to be more accurate.

Let's do it:
* Run: `terraform apply`

Looks like it's doing what we asked it to. <br>

![intro-to-terraform-67](./resources/intro-to-terraform-67.png) <br>
* Type: `yes`

Let's head to **Azure Portal** and see if our users have been created. <br>
*THEY SHOULD BE*

Let's also look at our **Known State** in **terraform.tfstate** <br>
We can see the **index_key** is the name, so similar to **sets** in that respect. <br>
We should be fine to add or create resources with ease. 

What I want to do now is actually nest a **map** within each **user** in our **users** variable.

**main.tf**
```tf
[ . . . ]

variable "users" {
  default = {
    # UPDATED CONFIG
    emile : { country: "England"},
    sarah : { country: "France"}
  }
}

[ . . . ]
```

So we've got the same **key** but the value isn't a string any more, it's another map. 

Let's update our **resource** so we can try and still access this **string** <br>

**ASK** <br>
How can we do that? <br>
**ANSWER** <br>

**main.tf**
```tf
[ . . . ]

resource "azuread_user" "my_azuread_users" {
  for_each             = var.users
  user_principal_name  = "${each.key}@emilesherrottdevops.onmicrosoft.com"
  display_name         = each.key
  mail_nickname        = each.key
  password              = "ChangeMe123!ChangeMe"
  # UPDATED CODE
  country               = each.value.country
}

[ . . . ]
```

Let's apply

* Run: `terraform apply` <br>
![intro-to-terraform-68](./resources/intro-to-terraform-68.png) <br>

No changes, because the values all are the same even though the way we get to those values is different. 

Let's add some more info.
**main.tf**
```tf
[ . . . ]

variable "users" {
  default = {
    # UPDATED CONFIG
    emile : { country: "England", department: "Training"},
    sarah : { country: "France", department: "Training"}
  }
}

resource "azuread_user" "my_azuread_users" {
  for_each             = var.users
  user_principal_name  = "${each.key}@emilesherrottdevops.onmicrosoft.com"
  display_name         = each.key
  mail_nickname        = each.key
  password              = "ChangeMe123!ChangeMe"
  country               = each.value.country
  # UPDATED CONFIG
  department            = each.value.department
}

[ . . . ]
```

So we've just given each user, a **country & department**

Both **country** and **department** are native attributes on `azuread_user`, so unlike our AWS example — where these had to be stuffed into a generic **tags** map — here they're proper, typed fields on the resource itself.

* Run: `terraform apply` <br>
![intro-to-terraform-69](./resources/intro-to-terraform-69.png) <br>

As we can see, it's looking to change by adding in an additional **attribute**. <br>
* Run: `yes`

Alright, we've changed the **Azure AD users** but we're now done so lets:
* `terrafrom destroy`
* `yes`

Our next project we're going to move onto **Virtual Machines** in **Azure**: **Azure Compute**

<br>
<br>

### Quick Review of Terraform FAQ

Let's take a quick step back and look at a few of the FAQs around Terraform

#### Why do you need state?
https://developer.hashicorp.com/terraform/language/state


#### Why do we need Known State?

*CLICK ON LINK NAMED state purpose*: [State | Terraform | HashiCorp Developer](https://developer.hashicorp.com/terraform/language/state/purpose) <br>

It explains the need for **Known State**
##### Mapping to the Real World
1. SO we've got "To map the terraform objects, to the real world"
   * We've spoken about the **Object ID** and **id** which is useful but the identifier format is specific to one provider. 
   * Early prototypes of terraform didn't actually use state and used tags instead.
##### Metadata
2. Another reason terraform needs state is to track the dependencies between different resources.
   * Terraform needs to be able to create resources and delete resources in the right order. 
     * Maybe an application needs a **Virtual Machine** and a **Database**, we'd need the **DB** before we can load up an application whose config connects to it. 
     * Terraform meta data is set up to do this.
##### Performance
3. Also state helps performance
   * We talk to resources on the cloud, if there's 10 different resources and we need information on all of them. That's a big request.
   * State can be seen as cache where the details are stored and we can use **terraform.tfstate** to make quicker decisions. 
   * We've used `-refresh=false` when we've not wanted to refresh the actual state

Up until now we've been using **local state**, we'll transition to using **remote state** soon which helps us to share configuration. 

We've already discussed that using Github repositories is dangerous because our state can contain sensitive information. 

#### Why have we created so many different projects?
The reason why because it's part of terraform best practices. A single application can and often have multiple terraform projects. 
The application may need:
* Users
* Storage Accounts
* Virtual Machines

Each resource type might have different life cycle.

* Virtual Machines could change daily
* The users may change a few times a year as teams grow and shrink

So having different terraform projects to manage the different resources is common. 

<br>
<br>

### Exercise

Again there's a series of exercises for you to look through, these exercises are challenging and there's completed code if needed. 

Maybe have a look through the completed code and understand it. Try running it but familiarise yourself with **Terraform**.

### Conclusion

- **Inform** students this marks the end of the lecture for this morning
- **Tell** students we will be continuing to look at more complex resources and remote backends later.


---

[Back](./README.md)

---

