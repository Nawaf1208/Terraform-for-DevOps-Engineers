# Terraform-for-DevOps-Engineers

![Terraform](https://img.shields.io/badge/terraform-%235835CC.svg?style=for-the-badge&logo=terraform&logoColor=white)

![](Terraform.png)  

## Exercises

## Terraform 101

<details>
<summary><b><i>1.Learn how to use and run Terraform basic commands

- Create a directory called "my_first_run"
- Inside the directory create a file called "main.tf" with the following content
  ```Terraform
  resource "local_file" "mario_local_file" {
    content  = "It's a me, Mario!"
    filename = "/tmp/who_is_it.txt"
  }
  ```
- Run `terraform init`. What did it do?
- Run `terraform plan`. What Terraform is going to perform?
- Finally, run `terraform apply` and verify the file was created

</i></b></summary>

$\color{green}{\text{Answer}}$

```Terraform
# Create a directory
mkdir my_first_run && cd my_first_run

# Create the file 'main.tf'
cat << EOT >>  main.tf
resource "local_file" "mario_local_file" {
    content  = "It's a me, Mario!"
    filename = "/tmp/who_is_it.txt"
}
EOT

# Run 'terraform init'
terraform init
# Running 'ls -la' you'll it created '.terraform' and '.terraform.lock.hcl'
# In addition, it initialized (downloaded and installed) the relevant provider plugins. In this case, the "hashicorp/local"

# Run 'terraform plan'
terraform plan
# It shows what Terraform is going to perform once you'll run 'terraform apply'

<< terraform_plan_output
Terraform will perform the following actions:

  # local_file.mario_local_file will be created
  + resource "local_file" "mario_local_file" {
      + content              = "It's a me, Mario!"
      + directory_permission = "0777"
      + file_permission      = "0777"
      + filename             = "/tmp/who_is_it.txt"
      + id                   = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
terraform_plan_output

# Apply main.tf (it's better to run without -auto-approve if you are new to Terraform)
terraform apply -auto-approve

ls /tmp/who_is_it.txt
# /tmp/who_is_it.txt
```

</details>

## AWS

<details>
<summary><b><i>2.Launch EC2 instance

- Write Terraform configuration for launching an EC2 instance
- Run the commands to apply the configuration and create the EC2 instance
- What happens if you run again `terraform apply`?
- Destroy the instance you've created with Terraform

</i></b></summary>

$\color{green}{\text{Answer}}$

```Terraform
mkdir exercise

cat << EOT >> main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }

  required_version = ">= 1.2.0"
}

provider "aws" {
  region  = "us-west-2"
}

resource "aws_instance" "app_server" {
  ami           = "ami-830c94e3"
  instance_type = "t2.micro"

  tags = {
    Name = "ExampleAppServerInstance"
  }
}
EOT

terraform init
terraform validate
terraform plan

# You should see this line at the end: Plan: 1 to add, 0 to change, 0 to destroy

terraform apply -auto-approve

# You should see the following output:
# aws_instance.app_server: Creation complete after 49s [id=i-004651a9d4427d236

# Running 'terraform apply' again won't change anything as
# Terraform will compare actual infrastructure to your
# configuration and won't find any difference. You should see the following line:
# Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

# Remove instance
terraform destroy -auto-approve

# Destroy complete! Resources: 1 destroyed.
```

</details>

<details>
<summary><b><i>3.Rename S3 Bucket

- An existing S3 bucket tracked by Terraform. If you don't have it, you can use the following block and run `terraform apply`:
  ```Terraform
  resource "aws_s3_bucket" "some_bucket" {
      bucket = "some-old-bucket"
  }
  ```
- Rename an existing S3 bucket and make sure it's still tracked by Terraform

</i></b></summary>

$\color{green}{\text{Answer}}$

```Terraform
# A bucket name is immutable in AWS so we'll have to create a new bucket
aws s3 mb s3://some-new-bucket-123

# Sync old bucket to new bucket
aws s3 sync s3://some-old-bucket s3://some-new-bucket-123

# Option 1 (remove and import)

## Remove the old bucket from Terraform's state
terraform state rm aws_s3_bucket.some_bucket

## Import new bucket to Terraform's state
terraform import aws_s3_bucket.some_bucket some-new-bucket-123

: '
aws_s3_bucket.some_bucket: Refreshing state... [id=some-new-bucket-123]

Import successful!
The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
'

# Option 2 (move)

## Move the old bucket from Terraform's state to the new one
terraform state mv aws_s3_bucket.some_bucket some-new-bucket-123

: '
Move "aws_s3_bucket.some_bucket" to "aws_s3_bucket.some-new-bucket-123"
Successfully moved 1 object(s).
'

# Modify Terraform file

# Modify the Terraform definition to include the new name
# resource "aws_s3_bucket" "some_bucket" {
#    bucket = "some-new-bucket-123"
# }

# Remove old bucket
aws s3 rm s3://some-old-bucket --recursive
aws s3 rb s3://some-old-bucket
```

</details>

<details>
<summary><b><i>4.Creating Custom VPC and Subnets with Terraform

- Create a custom VPC with a specified CIDR block. For example, you can use `10.0.0.0/16`.
- Create two subnets within the VPC, each with a different CIDR block. For example, you can use `10.0.0.0/20` for the first subnet and `10.0.16.0/20` for the second subnet.
- Both subnets should be in different availability zones to ensure high availability.
- Ensure that the VPC and subnets are tracked by Terraform.

</i></b></summary>

$\color{green}{\text{Answer}}$

```Terraform
# Create a directory for the Terraform configuration
mkdir vpc_subnet_creation && cd vpc_subnet_creation
```

```Terraform
# Create the main.tf file with the VPC and subnets configuration
touch main.tf
```

```Terraform
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "your-region" # e.g., ap-south-1
}

resource "aws_vpc" "my_custom_vpc" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "my_custom_vpc_made_with_terraform"
  }
}

resource "aws_subnet" "Subnet_A" {
  cidr_block       = "10.0.0.0/20"
  vpc_id            = aws_vpc.my_custom_vpc.id
  availability_zone = "your-availability-zone-a" # e.g., ap-south-1a
  tags = {
    "Name" = "Subnet A"
  }
}
resource "aws_subnet" "Subnet_B" {
  cidr_block       = "10.0.16.0/20"
  vpc_id            = aws_vpc.my_custom_vpc.id
  availability_zone = "your-availability-zone-b" # e.g., ap-south-1b
  tags = {
    "Name" = "Subnet B"
  }
}
```

```Terraform
# Initialize Terraform to download the AWS provider
terraform init
```

```Terraform
# Apply the Terraform configuration to create the VPC and subnets
terraform apply -auto-approve
```

</details>

## Questions

## Terraform 101

<details>
<summary><b><i>5.What is Terraform?</i></b></summary>

$\color{green}{\text{Answer}}$

HashiCorp Terraform is an infrastructure as code tool that lets you define both cloud and on-prem resources in human-readable configuration files that you can version, reuse, and share. You can then use a consistent workflow to provision and manage all of your infrastructure throughout its lifecycle. 

Terraform can manage low-level components like compute, storage, and networking resources, as well as high-level components like DNS entries and SaaS features.

</details>

<details>
<summary><b><i>6.What are the advantages in using Terraform or IaC in general?</i></b></summary>

$\color{green}{\text{Answer}}$

- Full automation: In the past, resource creation, modification and removal were handled manually or by using a set of tooling. With Terraform or other IaC technologies, you manage the full lifecycle in an automated fashion.

- Modular and Reusable: Code that you write for certain purposes can be used and assembled in different ways. You can write code to create resources on a public cloud and it can be shared with other teams who can also use it in their account on the same (or different) cloud>

- Improved testing: Concepts like CI can be easily applied on IaC based projects and code snippets. This allow you to test and verify operations beforehand.

</details>

<details>
<summary><b><i>7.What is one reason why manual processes can be helpful?</i></b></summary>

$\color{green}{\text{Answer}}$

For learning a platform when first starting out

</details>

<details>
<summary><b><i>8.Why is it advisable to avoid using manual processes when creating infrastructure at scale?</i></b></summary>

$\color{green}{\text{Answer}}$

Manual processes for creating infrastructure are slow because they require human intervention for each step, which delays deployment. They are error-prone since manual configuration increases the risk of mistakes and inconsistencies. Additionally, these processes are not easily repeatable, making it difficult to ensure the same infrastructure setup across different environments—unlike Infrastructure as Code (IaC), which automates and standardizes deployments.

</details>

<details>
<summary><b><i>9.What are some of Terraform features?</i></b></summary>

$\color{green}{\text{Answer}}$

- Declarative: Terraform uses the declarative approach (rather than the procedural one) in order to define end-status of the resources

- No agents: as opposed to other technologies (e.g. Puppet) where you use a model of agent and server, with Terraform you use the different APIs (of clouds, services, etc.) to perform the operations

- Community: Terraform has strong community who constantly publishes modules and fixes when needed. This ensures there is good modules maintenance and users can get support quite quickly at any point

</details>

<details>
<summary><b><i>10.What language does Terraform uses?</i></b></summary>

$\color{green}{\text{Answer}}$

A DSL called "HCL" (Hashicorp Configuration Language). A declarative language for defining infrastructure.

</details>

<details>
<summary><b><i>11.What's a typical Terraform workflow?</i></b></summary>

$\color{green}{\text{Answer}}$

- Write Terraform definitions: `.tf` files written in HCL that described the desired infrastructure state (and run `terraform init` at the very beginning)

- Review: With command such as `terraform plan` you can get a glance at what Terraform will perform with the written definitions

- Apply definitions: With the command `terraform apply` Terraform will apply the given definitions, by adding, modifying or removing the resources

This is a manual process. Most of the time this is automated so user submits a PR/MR to propose terraform changes, there is a process to test these changes and once merged they are applied (`terraform apply`).

</details>

<details>
<summary><b><i>12.What are some use cases for using Terraform?</i></b></summary>

$\color{green}{\text{Answer}}$

- Infra provisioning and management: You need to automated or code your infra so you are able to test it easily, apply it and make any changes necessary.

- Multi-cloud environment: You manage infrastructure on different clouds, but looking for a consistent way to do it across the clouds

- Consistent environments: You manage environments such as test, production, staging and looking for a way to have them consistent so any modification in one of them, applies to other environments as well

</details>

<details>
<summary><b><i>13.What's the difference between Terraform and technologies such as Ansible, Puppet, Chef, etc.</i></b></summary>

$\color{green}{\text{Answer}}$

Terraform is considered to be an IaC technology. It's used for provisioning resources, for managing infrastructure on different platforms.

Ansible, Puppet and Chef are Configuration Management technologies. They are used once there is an instance running and you would like to apply some configuration on it like installing an application, applying security policy, etc.

To be clear, CM tools can be used to provision resources so in the end goal of having infrastructure, both Terraform and something like Ansible, can achieve the same result. The difference is in the how. Ansible doesn't saves the state of resources, it doesn't know how many instances there are in your environment as opposed to Terraform. At the same time while Terraform can perform configuration management tasks, it has less modules support for that specific goal and it doesn't track the task execution state as Ansible. The differences are there and it's most of the time recommended to mix the technologies, so Terraform used for managing infrastructure and CM technologies used for configuration on top of that infrastructure.

</details>

## Terraform Hands-On Basics

<details>
<summary><b><i>14.Explain the following block of Terraform code

```Terraform
resource "aws_instance" "some-instance" {
  ami           = "ami-201720221991yay"
  instance_type = "t2.micro
}
```

</i></b></summary>

$\color{green}{\text{Answer}}$

It's a resource of type `aws_instance` used to provision an instance. The name of the resource (NOT INSTANCE) is `some-instance`.

The instance itself will be provisioned with type `t2.micro` and using an image of the AMI `ami-201720221991yay`.

</details>

<details>
<summary><b><i>15.What do you do next after writing the following in main.tf file?

```Terraform
resource "aws_instance" "some-instance" {
  ami           = "ami-201720221991yay"
  instance_type = "t2.micro
}
```

</i></b></summary>

$\color{green}{\text{Answer}}$

Run `terraform init`. This will scan the code in the directory to figure out which providers are used (in this case AWS provider) and will download them.

</details>

<details>
<summary><b><i>16.You've executed `terraform init` and now you would like to move forward to creating the resources but you have concerns and would like to make be 100% sure on what you are going to execute. What should you be doing?</i></b></summary>

$\color{green}{\text{Answer}}$

Execute `terraform plan`. That will provide a detailed information on what Terraform will do once you apply the changes.

</details>

<details>
<summary><b><i>17.You've downloaded the providers, seen the what Terraform will do (with terraform plan) and you are ready to actually apply the changes. What should you do next?</i></b></summary>

$\color{green}{\text{Answer}}$

Run `terraform apply`. That will apply the changes described in your `.tf` files.

</details>

<details>
<summary><b><i>18.Explain the meaning of the following strings that seen at the beginning of each line When you run `terraform apply`

- `+`
- `-`
- `-/+`

</i></b></summary>

$\color{green}{\text{Answer}}$

- `+` - The resource or attribute is going to be added
- `-` - the resource or attribute is going to be removed
- `-/+` - the resource or attribute is going to be replaced

</details>

<details>
<summary><b><i>19.How to cleanup Terraform resources? Why the user should be careful doing so?</i></b></summary>

$\color{green}{\text{Answer}}$

`terraform destroy` will cleanup all the resources tracked by Terraform.

A user should be careful with this command because there is no way to revert it. Sure, you can always run again "apply" but that can take time, generates completely new resources, etc.

</details>
