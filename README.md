# Terraform Beginner Bootcamp 2023

## Semantic Versioning :mage:

This project is going to utilize semantic versioning for its tagging.
[semver.org](https://semver.org/)


The general format:

**MAJOR.MINOR.PATCH**, eg. `1.0.1`:

- **MAJOR** version when you make incompatible API changes
- **MINOR** version when you add functionality in a backward compatible manner
- **PATCH** version when you make backward compatible bug fixes

## Install the Terraform CLI

### Considerations with the Terraform CLI changes
The Terraform CLI installation has changed due to gpg keyring changes. So we needed to refer to the latest CLI instructions via Terraform documentation and change the scripting for install.

[Install Terraform CLI](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)

### Considerations for Linux Distributions

This project is built against Ubuntu.
Please consider checking your linux distribution and change accoridngly to distribution needs.

Example of checking OS version:

```
$ cat /etc/os-release
PRETTY_NAME="Ubuntu 22.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.4 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy
```

[How to check Linux Distribution Version](https://www.cyberciti.biz/faq/find-linux-distribution-name-version-number/)


### Refactoring into Bash Scripts

While fixing the Terrform CLI gpg deprecated issues we noticed that the bash scipts steps were a considerable amount more code. So we decided to create a bash script to install the Terraform CLI. 

This bash script is located here: [./bin/install_terraform_cli](./bin/install_terraform_cli)

- This will keep the Gitpod Task File tidy. ([.gitpod.yml](.gitpod.yml))
- This allows us an easier to debug and execute manually Terraform CLI install
- This will allow better protability for other projets that need to install Terraform CLI.

#### Shebang Considerations

A Shebang (pronounced Sha-bang) tells the bash script what program that will interpret the script. `#!/bin/bash`

ChatGPT recommended this format for bash: `#!/usr/bin/env bash`

- for portability for different OS distributions
- will search the users PATH for the bash executable

[Shebang Wiki](https://en.wikipedia.org/wiki/Shebang_(Unix))

#### Execution Considerations
When executing the bash script we can use th `./` shorthand notation to execute the bash script. 

eg. `./bin/install_terraform_cli`

If we are using a script in .gitpod.yml we need to point the script to a program to interpret it.

eg. `source ./bin/install_terraform_cli`

#### Linux Permissions Considerations

In order to make our bash scripts executable we need to change linux permissions for the fix to be executable to the user mode.

```sh
chmod u+x ./bin/install_terraform_cli
```

alternatively:

```sh
chmod 744 ./bin/install_terraform_cli
```
https://en.wikipedia.org/wiki/Chmod

### Github Lifecycle (Before, Init, Command)

We need to be careful when using the Init because it will not rerun if we restart an existing workspace.

https://www.gitpod.io/docs/configure/workspaces/tasks

### Working with Env Vars

#### env command

We can list out all Environment Variables (Env Vars) using the `env` command

We can filter specific env vars using grep eg. `env | grep AWS_`

#### Setting and Unsetting Env Vars

In the terminal we can set using `export HELLO= `world`

In the terminal we unset using `unset HELLO`

We can set an env var temporarily when just running a command

```sh
HELLO=`world` ./bin/print_mesage
```

Within a bash script we can set env without writing export eg.

```sh
#!/usr/bin/env bash

HELLO=`world`

echo $HELLO
```

#### Printing Vars

We can print an env var using echo eg. `echo $HELLO`

#### Scoping of Env Vars

When you open up new bash terminals in VSCode it will not be aware of Env Vars that you have set in another window. 

If you want Env Vars to exist across all bash terminals that are open you need to set env vars in your bash profile. eg. `.bash_profile`

#### Persisting Env Vars in GitPod

We can persist env vars into gitpod by storing them in Gitpod Secrets Storage. 

```
gp env HELLO=`world`
```

All future workspace launched will set the env vars for all bash terminals opened in those workspaces

You can also set env vars in the `.gitpod.yml` but this can only contain non-sensitive env vars. 

### AWS CLI Installation

AWS CLI is installed for this project via the bash script [`./bin/install_aws_cli`](./bin/install_aws_cli)

[Getting Started Install (AWS CLI)](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

[AWS CLI Env Vars](https://docs.aws.amazon.com/cli/v1/userguide/cli-configure-envvars.html)

We can check if our AWS credentials are configured correctly, but running the following AWS CLI command:
```sh
aws sts get-caller-identity
```

If it is successful you should see a json payload return that looks liek this:

```json
{
    "UserId": "AIDAYIR1991EKSMMJ19S6",
    "Account": "561236547584",
    "Arn": "arn:aws:iam::56813254884:user/terraform-beginner-bootcamp"
}
```

We'll need to generate AWS CLI credentials from IAM user in order to use the AWS CLI.


## Terraform Basics

### Terraform Registry

Terraform sources their providers and modules from the Teraform registry which is located at [registry.terraform.io](https://registry.terraform.io/)

- **Providers** are an interface to APIs that will allow you to create resources in Terraform.
- **Modules** are a way to make large amount of terraform code modular, portable and sharable. 

[Random Terraform Provider](https://registry.terraform.io/providers/hashicorp/random/latest)

### Terraform Console

We can see a list of all the Terraform commands by. simply typing `terraform`

#### Terraform Init

At the start of a new Terraform project we will run `terraform init` to download the plugins for the terraform providers being used in the project.

#### Terraform Plan
`terraform plan`
This will generate out a changeset about the state of our infrastructure and what will be changed.

We can output this changeset ie. "plan" to be passed to an "apply", but often you can ignore outputting.

#### Terraform Apply
`terraform apply`
This will run a plan and pass the changeset to be executed by terraform, apply should prompt us to type "yes" to approve changes.

If we want to automatically approve an apply we can provide the auto approve flag eg. `terraform apply -auto-approve`

#### Terraform Destroy
`terraform destroy`
This will destroy the previously executed by terraform configuration on AWS, destroy should prompt us to type "yes" to destroy changes.

If we want to automatically approve an destroy we can provide the auto approve flag eg. `terraform destroy -auto-approve`

### Terraform Lock Files

`.terraform.lock.hcl` contains the locked versioning for the providers or modules that should be used with this project.

The terraform lock file **should be committed** to your version control system (VSC) eg. Github

### Terraform State Files

`.terraform.tfstate` contains information about the current state of your infrastructure.

This file **should not be committed** to your VCS.

This file can contain sensitive data

If you lose this file, you lose knowing the state of your infrastructure.

`.terraform.tfstate.backup` is the previous state of file state.

### Terraform Directory

`.terraform` directory containces binaries of terraform providers.


