[![Latest Release](https://img.shields.io/github/v/tag/slalom-consulting-ltd/terraform-guidelines.svg)](https://github.com/slalom-consulting-ltd/terraform-guidelines)

By [James Woolfenden](https://www.linkedin.com/in/jameswoolfenden/)

## A style guide for writing Terraform

This is a guide to writing Terraform to conform to Slalom London Style, it follows the Hashicorp guide to creating modules for the [Terraform Registry](https://www.terraform.io/docs/registry/modules/publish.html) and their [standard structure](https://www.terraform.io/docs/modules/index.html#standard-module-structure).

There are many successful ways of writing your tf, this one is tried and field tested.

## Naming

Use Lowercase names for resources.
Use "\_" as a separator for resource names.
Name must be self explanatory containing several lowercase words if needed separated by "\_".

Use descriptive and non environment specific names to identify resources.

Avoid Tautologies:

``` terraform
resource "aws_iam_policy" "ec2_policy"{
  ...
}
```

## Hardcoding

Don't hardcode values in resources. Add variables and set defaults.

You can avoid limiting yourself with policies and resources by making resources optional or overidable.

```Terraform
resource "aws_iam_role" "codebuild" {
  name  = "codebuildrole-${var.name}"
  count = "${var.role == "" ? 1 : 0}"

  assume_role_policy = <<HERE
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "codebuild.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
HERE

  tags = var.common_tags
}
```

And avoid heredocs like the one above, and use **data.aws_iam_policy_documents** objects, as practical.

## Templates

This is the Terraform code that is environment specific.  If your Templates are application specific the code should live with the code that requires it, create a folder in the root of the repository and call it **IAC**, similar to this for the repository aws-lexbot-handlers:

```bash
23043-5510:/mnt/c/aws-lexbot-handler# ls -l
total 64
-rwxrwxrwx    1 jimw     jimw          1719 Mar  7 11:02 README.MD
-rwxrwxrwx    1 jimw     jimw           411 Mar  7 11:02 buildno.sh
-rwxrwxrwx    1 jimw     jimw          1136 Mar 18 15:43 buildspec.yml
-rwxrwxrwx    1 jimw     jimw           489 Mar 18 15:40 getlatest.ps1
-rwxrwxrwx    1 jimw     jimw           479 Mar  7 11:02 getlatest.sh
drwxrwxrwx    1 jimw     jimw           512 Feb 26 16:22 iac
drwxrwxrwx    1 1000     1000           512 Mar 18 15:41 node_modules
-rwxrwxrwx    1 jimw     jimw         49979 Mar 18 15:41 package-lock.json
-rwxrwxrwx    1 jimw     jimw          1579 Mar 18 15:41 package.json
drwxrwxrwx    1 jimw     jimw           512 Feb  5 11:51 powershell
-rwxrwxrwx    1 jimw     jimw           147 Mar  7 11:02 setlatest.sh
```

Inside the **iac** I breakdown the **templates** used:

```bash
total 0
drwxrwxrwx    1 jimw     jimw           512 May 28 11:22 codebuild
drwxrwxrwx    1 jimw     jimw           512 Apr  2 11:00 repository
```

This example  has an AWS CodeCommit repository (self describing) and an AWS Codebuild, that has multiple environments:

```bash
total 0
drwxrwxrwx    1 jimw     jimw           512 Apr 24 23:28 dev
drwxrwxrwx    1 jimw     jimw           512 Apr 24 23:29 prod
```

Inside each of these folder is an environmental specific template:

```bash
total 19
-rwxrwxrwx    1 jimw     jimw           800 May 28 11:21 Makefile
-rwxrwxrwx    1 jimw     jimw          1324 Mar  7 11:02 README.md
-rwxrwxrwx    1 jimw     jimw           709 Mar  7 11:02 aws_iam_policy.additionalneeds.tf
-rwxrwxrwx    1 jimw     jimw            40 Mar  7 11:02 data.aws_current.region.current.tf
-rwxrwxrwx    1 jimw     jimw           579 May 28 11:23 module.codebuild.tf
-rwxrwxrwx    1 jimw     jimw           239 Mar  7 11:02 outputs.tf
-rwxrwxrwx    1 jimw     jimw           208 Apr 24 23:30 provider.tf
-rwxrwxrwx    1 jimw     jimw           349 Mar  7 11:02 remote_state.tf
-rwxrwxrwx    1 jimw     jimw          1531 May 28 11:25 terraform.tfvars
-rwxrwxrwx    1 jimw     jimw           618 May 28 11:20 variables.tf
```

There's a lot of files here and some repetition - that violates DRY principles, but with IAC, favour on being explicit.
Each template is directly runnable, using the Terraform CLI with no wrapper script required.
Use a generator like [tf-scaffold](https://github.com/JamesWoolfenden/tf-scaffold) to automate template creation.

Tf-Scaffold creates:

### .gitignore

Has good defaults for working with Terraform.

```yaml
terraform.tfplan
terraform.tfstate
.terraform/
./*.tfstate
*.backup
*.DS_Store
*.log
*.bak
*~
.*.swp
.project

```

### .pre-commit-config.yaml

Has a standard set of pre-commit hooks for working with Terraform and AWS. You'll need to install the pre-commit framework https://pre-commit.com/#install. And after that you need to add this file to your new repository **pre-commit-config.yaml**, in the root:

```yml
repos:
-   repo: git://github.com/pre-commit/pre-commit-hooks
    rev: v2.1.0
    hooks:
    -   id: trailing-whitespace
    -   id: end-of-file-fixer
    -   id: check-yaml
    -   id: check-added-large-files
- repo: git://github.com/Lucas-C/pre-commit-hooks
  rev: v1.1.6
  hooks:
  - id: forbid-tabs
    exclude_types: [python, javascript, dtd, markdown, makefile, xml]
    exclude: binary|\.bin$
- repo: git://github.com/kintoandar/pre-commit.git
  rev: v2.1.0
  hooks:
  - id: terraform_fmt
- repo: git://github.com/pre-commit/pre-commit-hooks
  rev: v2.1.0
  stages:
  - commit
  - push
  hooks:
  -   id: detect-aws-credentials
  -   id: detect-private-key
- repo: git://github.com/detailyang/pre-commit-shell
  rev: 1.0.4
  hooks:
  - id: shell-lint
- repo: git://github.com/igorshubovych/markdownlint-cli
  rev: v0.13.0
  hooks:
  - id: markdownlint
```

``` bash
pre-commit install
```
Hooks choices are a matter for each project. 

### main.tf

This is an expected file for Terraform modules. Remove it if this a template and add a **module.tf**.

### Makefile

A helper file. This is just to make like easier for you. Problematic if you are cross platform as make isn't very good/awful at that. If you use Windows update the PowerShell profile with equivalent helper functions instead.

```makefile
#Makefile
ifdef OS
   RM   = $(powershell  -noprofile rm .\.terraform\modules -force -recurse)
   BLAT = $(powershell  -noprofile rm .\.terraform\ -force -recurse)
else
   ifeq ($(shell uname), Linux)
      RM  = rm .terraform/modules/ -fr
      BLAT= rm .terraform/ -fr
   endif
endif

.PHONY: all

all: init plan build

init:
	$(RM)
	terraform init -reconfigure

plan: init
	terraform plan -refresh=true

p:
	terraform plan -refresh=true | landscape

build: init
	terraform apply -auto-approve

check: init
	terraform plan -detailed-exitcode

destroy: init
	terraform destroy -force

docs:
	terraform-docs md . > README.md

valid:
	tflint
	terraform fmt -check=true -diff=true

target:
	@read -p "Enter Module to target:" MODULE;
	terraform apply -target $$MODULE

purge:
	$(BLAT)
	terraform init -reconfigure
```

### outputs.tf

A standard place to return values, either to the screen or to pass back from a module.

### provider.aws.tf

Or Whatever you provider is or are.
Required. You are always going to be using these, included is this, the most basic provider for AWS.

### README.md

Where all the information goes.

### example.auto.tfvars

Files ending .auto.tfvars get picked by Terraform locally and in Terraform cloud. 
This is the standard file for setting your variables in, and is automatically picked up by Terraform.

### variables.tf

For defining your variables and setting default values. Each variable should define its type and have an adeqate description.
Also contains a map variable common_tags, which should be extended and used on every taggable object.

### .dependsabot/config.yml

Sets the repository to be automatically dependency scanned in Github.

## Modules

You've written some TF and your about to duplicate its' functionality, it's time to abstract to a module. A module should be more than just one resource, it should add something.  
Modules should be treated like applications services with a separate code repository for each module.

Each module should have a least one example included that demonstrates its usage. This example can be used as a test for that module, here its called **exampleA**.

```bash
examples/exampleA/
```

This is an example for AWS codecommit that conforms <https://github.com/JamesWoolfenden/terraform-aws-codecommit>

## Files

### Name your files after their contents

For a security group called "elastic", the resource is then aws_security_group.elastic, so the file is **aws_security_group.elastic.tf**. Be explicit.
It will save you time.

### Comments

Use Markdown for this, as many fmt and parsers break when you add comments into your TF with hashes and slash star comments.

### One resource per file

**Exception**: By all means group resources - where its really makes logical sense, security_group with rules, routes with route tables.

## Be Specific

You have 2 choices with dependencies. Live on the bleeding edge, or fix your versions. I recommend being in control.

### Fix the version of Terraform you use

The whole team needs to use the same version of the tool until you decide as a team to update.  
Create a file called **terraform.tf** in your template:

```terraform
terraform {
    required_version="0.12"
}
```

### Fix the version of the modules you consume

In your **module.tf** file, set the version of the module. If you author modules, make sure you tag successful module builds.
If your module comes from a registry, specify using the version property, if its only standard git source reference use a tag reference in your source statement.

If it's using modules from the registry like **modules.codebuild.tf**:

```terraform
module codebuild {
  source                 = "jameswoolfenden/codebuild/aws"
  version                = "0.1.41"
  root                   = var.root
  description            = var.description
}
```

### Fix the version of the providers you use

Using shiny things is great, what's not great is code that worked yesterday breaking because a plugin/provider changed. Specify the version in your **provider.tf** file.

```terraform
provider "aws" {
  region  = "eu-west-1"
  version = "2.15.0"
}
```

## State

Using remote state is not optional, use a [locking state bucket](https://registry.terraform.io/modules/JamesWoolfenden/statebucket/aws/0.0.15) or use the free state management layer in Terraform Enterprise. This new free tier is worth a look.

## Layout

Mandate the use of the standard pre-commit, using that the command **Terraform fmt** is always run on Git commit. End of problem.

## Protecting Secrets

Protect your secrets by installing using the pre-commit file and the hooks from the standard set:

```hooks
- id: detect-aws-credentials
- id: detect-private-key
```

Other options include using git-secrets, Husky or using Talisman.  Use and mandate use of one, by all. Dont be that person.

## Configuration

Convention over configuration is preferred.
Use a data source over adding a configuration value.
Set default values for your modules variables.
Make resources optional with the count syntax.

## Unit Testing

As yet to find a really satisfactory test approach or tool for testing Terraform other than:

 - Include a test implementation with your modules - from your examples root folder.
 - Run it for every change.
 - Tag the successful outcomes.
 - Destroy created resources

## Tagging

Implement a tagging scheme from the start, and use a map type for extensibility.

In **variables.tf**:

```terraform
variable "common_tags" {
  type       = "map"
  description= "Implements the common_tags scheme"
}
```

And in your **example.auto.tfvars**

```terraform
  common_tags={
    name      = "sap-proxy-layer"
    owner     = "James Woolfenden"
    costcentre= "development"
  }
```

and then have the common_tags used in your resources file:

```terraform
resource "aws_codebuild_project" "project" {
  name          = "${replace(var.name,".","-")}"
  description   = var.description
  service_role  = "${var.role == "" ? element(concat(aws_iam_role.codebuild.*.arn, list("")), 0) : element(concat(data.aws_iam_role.existing.*.arn, list("")), 0) }"
  build_timeout = "var.build_timeout

  artifacts {
    type                = var.type
    location            = local.bucketname
    name                = var.name
    namespace_type      = var.namespace_type
    packaging           = var.packaging
    encryption_disabled = var.encryption_disabled
  }

  environment = var.environment
  source      = var.sourcecode
  tags        = var.common_tags
}
```
So, use Readable Key value pairs.
DO NOT NAME OR TAG YOUR RESOURCES LIKE YOUR STILL ON PREM. 
So naming a security group  

 **DEV_TEAM_WILBUR4873_APP_SG**

is not helpful but
```
tags={
TEAM="Wilbur"
Purpose="App"
CostCode="4873}
```
Is fine. Names you can't update, tags you can. The longer you make the resource names the more bugs you will find/make.
Ok I get it some resources dont have tag attributes or you have some "Security" policy or other that mean you must have a naming regime.
If so I'd either use or copy the naming module from the Cloud Posse [https://github.com/cloudposse/terraform-null-label](https://github.com/cloudposse/terraform-null-label).

## Recommended Tools

[Terraform-docs](https://github.com/segmentio/terraform-docs)

Run to help make your readmes, included with the build-harness.

The [Pre-commit](https://pre-commit.com/) framework

So many different uses from linting to security, every git repo should have one.

[Beyond-Compare](https://www.scootersoftware.com/) or equivalent

A preference, for a comparision tool.

The Cli

Be it AWS, or whatever provider your using.

[VSCode](https://code.visualstudio.com/) and Extensions

Free and really quite good editor, with awesome extensions. Use the Extensions sync extension to maintain your [environment](https://gist.github.com/JamesWoolfenden/1a1ce363e6e6e5d2bcf321ca12ec3de2).

[AWS-Vault](https://github.com/99designs/aws-vault)

Helps with managing many AWS accounts at the CLI.

[SAML2AWS](https://github.com/Versent/saml2aws)

Generates temporary AWS credentials for AWS cmdline. Essential for running in Federated AD environment.

[build-harness](https://github.com/cloudposse/build-harness)

A DevOps related collection of automated build processes, customised [Slalom version](https://github.com/JamesWoolfenden/build-harness)

[Travis](https://travis-ci.com/) - or free for [open source projects](https://travis-ci.org/getting_started).

There are many other good SAS CI/CD tools including Circle, GitLab and a few shockers.

## Caches

Set a [plugin cache](https://www.terraform.io/docs/commands/cli-config.html). On a fat pipe you might not notice how quickly they download, but do setup your plugin-cache. It will save you time and stress.
