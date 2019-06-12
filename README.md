# terraform-guidelines

A style guide for writing Terraform.

This is a guide to writing Terraform to conform to Slalom London Style, it follows the Hashicorp guide to creating modules for the Terraform Registry <https://www.terraform.io/docs/registry/modules/publish.html> and their standard structure <https://www.terraform.io/docs/modules/index.html#standard-module-structure>.

There are many successful ways of writing your tf, this is tried and field tested.

## Templates

This is the Terraform code that is environment specific.  Templates should live with the code the that requires it, I usually create a root folder in the repository and all it **IAC**, something like this for the repository aws-lexbot-handlers:

```bash
23043-5510:/mnt/c/aws-lexbot-handler# ls -l
total 64
-rwxrwxrwx    1 root     root          1719 Mar  7 11:02 README.MD
-rwxrwxrwx    1 root     root           411 Mar  7 11:02 buildno.sh
-rwxrwxrwx    1 root     root          1136 Mar 18 15:43 buildspec.yml
-rwxrwxrwx    1 root     root           489 Mar 18 15:40 getlatest.ps1
-rwxrwxrwx    1 root     root           479 Mar  7 11:02 getlatest.sh
drwxrwxrwx    1 root     root           512 Feb 26 16:22 iac
drwxrwxrwx    1 1000     1000           512 Mar 18 15:41 node_modules
-rwxrwxrwx    1 root     root         49979 Mar 18 15:41 package-lock.json
-rwxrwxrwx    1 root     root          1579 Mar 18 15:41 package.json
drwxrwxrwx    1 root     root           512 Feb  5 11:51 powershell
-rwxrwxrwx    1 root     root           147 Mar  7 11:02 setlatest.sh
```

Inside the **iac** I breakdown the **templates** used:

```bash
total 0
drwxrwxrwx    1 root     root           512 May 28 11:22 codebuild
drwxrwxrwx    1 root     root           512 Apr  2 11:00 repository
```

This example just has an AWS CodeCommit repository (self describing) and an AWS Codebuild that has multiple environments:

```bash
total 0
drwxrwxrwx    1 root     root           512 Apr 24 23:28 dev
drwxrwxrwx    1 root     root           512 Apr 24 23:29 prod
```

Inside one of these and environmental specific template:

```bash
total 19
-rwxrwxrwx    1 root     root           800 May 28 11:21 Makefile
-rwxrwxrwx    1 root     root          1324 Mar  7 11:02 README.md
-rwxrwxrwx    1 root     root           709 Mar  7 11:02 aws_iam_policy.additionalneeds.tf
-rwxrwxrwx    1 root     root            40 Mar  7 11:02 data.aws_current.region.current.tf
-rwxrwxrwx    1 root     root           579 May 28 11:23 module.codebuild.tf
-rwxrwxrwx    1 root     root           239 Mar  7 11:02 outputs.tf
-rwxrwxrwx    1 root     root           208 Apr 24 23:30 provider.tf
-rwxrwxrwx    1 root     root           349 Mar  7 11:02 remote_state.tf
-rwxrwxrwx    1 root     root           148 Mar  7 11:02 setlatest.sh
-rwxrwxrwx    1 root     root          1531 May 28 11:25 terraform.tfvars
-rwxrwxrwx    1 root     root           618 May 28 11:20 variables.tf
```

There's a lot of files in here and some repitition that violates DRY principles, but with IAC, favour on being explict.
Each template is directly runnable using the Terraform CLI no wrapper script required.
Use a generator like tf-scaffold to automate template creation <https://github.com/JamesWoolfenden/tf-scaffold>

## Modules

You've written some TF and you about to duplicate its functionality, it's time to abstract a module. A module should be more than just one resource.  
Modules should be treated like applications services with a seperate code repository for each module.

## Files

- Name your files after their contents.
You have a security group called for elastic search the resource is aws_security_group.elastic, so the file is **aws_security_group.elastic.tf**. Be explicit.

- One resource per file.
**Exception**: By all means group resources where its really makes logical sense, security_group with rules, routes with route tables.

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

In your **module.tf** file set the version of the module. If you author modules make sure you tag successful module builds.
If your module comes from a registry, specify using the verision property, if its only standard git use a tag reference in your source statement.

### Fix the version of the providers you use

Using shiny things is great, what's not great is code that worked yesterday breaking because a plugin/provider changed. Specify the version in your **provider.tf** file.

## State

Using remote state is not optional, use a locking state bucket or use the free state management layer in Terraform Enterprise. The new free tier is worth a look.

## Layout

As your mandating use of the standard pre-commit, **Terraform fmt** is always run on git commit. End of problem.

## Protecting Secrets

Protect your secrets by installing using the pre-commit file and the hooks from the standard set:

- id: detect-aws-credentials
- id: detect-private-key

Other options include using git-secrets, Husky or using Talisman.  Use and mandate use of one, by all. Dont be that person.

## Configuration

Convention over configuration is preffered.
Use a data source over adding a configuration value.
Set default values for your modules variables.
Make resources optional with the count syntax.

## Tagging

Implement a tegging scheme from the start, use a map type for extensibility.
