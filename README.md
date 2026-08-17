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
