# reddit Terraform style guide

## Other resources
This document captures our agreed practices for:
* Code style
* Code structure
* Repo structure

See the [Terraform Service Manual](https://reddit.atlassian.net/wiki/spaces/IO/pages/71139524/Terraform) for all other Terraform process and policy.<br>
See [AWS Accounts](https://reddit.atlassian.net/wiki/spaces/IO/pages/120940398/AWS+Accounts) for account shortnames.


## Terraform version
We all use the same major version of Terraform, knowing we will need to update to the next major release in sync with each other, and with the Drone CI that checks Terraform code.
* The current major release is `0.11.0`
* Terraform developers should be running at least 0.11.0, and from time to time will be required to update to a newer minor version when a template deployed with a newer version is encountered.

## Code style and structure
We require running `terraform fmt` to control general structure of terraform templates. [(1)](#1).

Additionally, we observe these local conventions:

* **Every blueprint _or_ module has a `main.tf`**
    * See [Blueprint and module patterns](#Blueprint-and-module-patterns) below for the location of `main.tf`, which varies by use case.
    * Required items in `main.tf` (examples below)
        * include the constants module, if it is needed in the blueprint/module
        * pin a terraform required version
        * pin an AWS provider required version
        * set one region and one allowed account ID [(2)](#2)
    * If the blueprint or module is simple
        * `main.tf` may contain all the required items listed above, along with all the resource definitions that comprise the blueprint or module.
    * If the blueprint or module is not simple
        * `main.tf` should contain only the required items, along with common variables needed in other files. Resources should be defined by one or more additional .tf files located alongside `main.tf`.

* **Most directories have a `README.md`**
    * `README.md` should concisely explain what's in the directory
    * Where applicable, include a link to more information about the thing defined in the directory, such as a link to a service manual for the service defined by the blueprints in the directory
    * For shared modules under [/shared_modules](https://github.com/reddit/reddit-terraform/tree/master/shared_modules), each module's `README.md` should both explain the module's purpose and usage, and provide concrete usage examples

* **Define and use constants** for widely-used, rarely-varying values such as AWS account numbers and VPC IDs.
    * Constants allow the use of human-friendly symbols such as `account_search` in templates, rather than obscure and hard-to-remember literals like "106382545705" (the actual account number of the search account).
    * When a value is defined in a constant, always use the constant rather than hard-coding the literal value.
    * Talk with the Terraform working group in the [#terraform](https://reddit.slack.com/messages/terraform/) slack channel before introducing new types of constants, as there are limited use cases. Good candidates for new constants are any values that are invariant or that are usually pinned and manually maintained, that are useful in many Terraform blueprints and/or modules.
    * Constants are defined at: [reddit-terraform/shared_modules/constants/CONSTANT_SET_NAME.tf](https://github.com/reddit/reddit-terraform/tree/master/shared_modules/constants)
    * To include constants in blueprints and modules _(relative path to constants from blueprint will vary)_:
    ```
    module constants {
      source = "../../../shared_modules/constants"
    }
    ```

* **Pin the current major Terraform release** in main.tf using [pessimistic constraints](https://www.terraform.io/docs/configuration/terraform.html#specifying-a-required-terraform-version)
    ```
    terraform {
      required_version = "~> 0.11"
    }
    ```
    * If you need a more recent minor version than .0, use that instead (e.g. replace 0.11.0 with 0.11.1).
    * If you validated and deployed the code using a more recent minor version, pin that code to the version you are actually running. [(3)](#3)
    * Our current major version is 0.11. Blueprints should utilize ~> 0.11.0, or a more recent minor version (0.11.x) as appropriate.

* **Use account, region, and provider version** in main.tf using [pessimistic constraints](https://www.terraform.io/docs/configuration/terraform.html#specifying-a-required-terraform-version) for the version [(4)](#4)
    ```
    provider aws {
      region = "us-east-1"
      allowed_account_ids = ["${module.constants.ACCOUNT_SHORTNAME}"]
      version = "~> 1.1"
    }
    ```

* **Resource types and names are unquoted** [(5)](#5)
    * This:<br>
    ```
    resource aws_instance foo {
     …
    }
    ```
    * not this:<br>
    ```
    resource  “aws_instance” “foo” {
    …
    }
    ```

* **Variable and resource names use underscores**, not dashes
    * These:<br>
    ```
    variable my_variable {
        ...
    }

    resource aws_security_group_rule someservice_egress {
        ...
    }
    ```
    * Not these:<br>
    ```
    variable my-variable {
        ...
    }

    resource aws_security_group_rule someservice-egress {
        ...
    }
    ```
    
* **All templated files must end with their formatting type and a `.tmpl` suffix**
    * This:<br>
    ```
    data template_file graphite_hostname {
      # File has a .yaml.tmpl suffix since its a yaml templated file 
      template = "${file("${path.module}/cloud-config/hostname.yaml.tmpl")}"

      vars {
        hosttype = "graphite"
      }
    }
    ```
    * Not this:<br>
    ```
    data template_file graphite_hostname {
      template = "${file("${path.module}/cloud-config/hostname.yaml")}"

      vars {
        hosttype = "graphite"
      }
    }
    ```

* **Where it makes sense, the terraform statefile `.tfstate` should live in S3**
    *  This allows us to avoid worrying about committing  statefile changes. There have been several incidents where un-committed statefile changes resulted in problems for the next person interacting with that blueprint.
    * The exception to this rule are blueprints that often require manual statefile maintenance using methods such as `terraform state rm`, `terraform state mv`, and `terraform import`. These blueprints may opt to keep their statefile committed to this repo.

    However, once a service's statefile is stable and no longer requires routine manual maintenance, it should be moved to S3 when convenient.

* **AWS IAM / access policies are written in Terraform code**, not inline JSON
    * We prefer Terraform code so that we can use comments inline, and so that we write will be rendered in up-to-date AWS JSON.


## Repo layout

### Per-account-and-region directories
* Because we do not yet have tooling to enforce the application of terraform plans to specific accounts, we separate terraform blueprints by directory, under `blueprints/ACCOUNT_SHORTNAME-SHORT_REGION/`, using the [short-name of the account](https://reddit.atlassian.net/wiki/spaces/IO/pages/120940398/AWS+Accounts) and the region, like this:
    ```
    blueprints/
        data-ue1/
        search-ue1/
        prod-ue1/
    ```
* Files in `blueprints/` that are not under a directory named for an account are presumed to belong to the production account. These will be relocated in the future to:
    `blueprints/prod-ue1/`

### Blueprint and module patterns
#### 1. A complex blueprint, with or without local modules
* This pattern is the most common. It is the correct pattern for defining application services.
* For most services, there is no benefit to using modules, and no basis for using shared modules or abstracting privately-defined resources into private modules.
* The resources that comprise the service are defined in one or more blueprints.
* Parts of the blueprint _may_ be abstracted into private modules as appropriate for clean design, when the total system includes a case like #2 below.
* Location of `main.tf`:
    * The blueprint is the common element, so place `main.tf` with the blueprint files:<br>
    example: `blueprints/prod-ue1/sendbird/main.tf`
* Example:<br>
  In this example, there are no private modules. [main.tf](https://github.com/jyoullreddit/reddit-terraform/tree/master/blueprints/prod-ue1/sendbird/main.tf) defines the basics and security groups. Other complex resources are defined in separate .tf files in the [blueprints/prod-ue1/sendbird/](https://github.com/jyoullreddit/reddit-terraform/tree/master/blueprints/prod-ue1/sendbird) directory.
    ```
    blueprints/prod-ue1/sendbird/
        cassandra.tf
        elasticache.tf
        main.tf
        ...
    ```

#### 2. Many simple, nearly-identical blueprints with one shared private module
* This pattern is suited to generating many nearly-identical, probably simple systems, such as cache clusters. Our twelve cache clusters vary by name and cluster size, but are otherwise mostly identical
* All resources comprising a system in a single module, probably one .tf file
* Each of the blueprints defines the variables for one instance of the system (e.g one cache cluster)
* Location of `main.tf`:
    * The _module_ is the common element, so place `main.tf` with the module:<br>
    example: `r2-caches/modules/r2-cache-asg/main.tf`
* Example:<br>
  In this example, module  [blueprints/r2-caches/modules/r2-cache-asg/](https://github.com/jyoullreddit/reddit-terraform/tree/master/blueprints/r2-caches/modules/r2-cache-asg) is called by individual cache-making blueprints in [blueprints/r2-caches/](https://github.com/jyoullreddit/reddit-terraform/tree/master/blueprints/r2-caches)
    ```
    blueprints/prod-ue1/r2-caches/
        cache-gutter.tf
        cache-hard.tf
        cache-incr.tf
        ...
        modules/r2-cache-asg/
            main.tf
            variables.tf
            files/
                userdata.sh
    ```

## Shared modules
Shared modules are an exception to the per-account-and-region directory structure. A few modules are universally shareable by all blueprints/modules, such as the `constants` module.
* Shared modules are found at `/shared_modules/MODULE_NAME`.
* Example:<br>
    ```
    modules_shared/
        README.md    <-- required
        constants/   <-- the module "constants"
            ami.tf
            vpc.tf
            ...
    ```

## S3 buckets
* Terraform files for S3 buckets that are cleanly owned and more or less exclusively used by one service (that is defined in Terraform) can be co-located with other templates for the service.
* Terraform files for S3 buckets that are shared between services or that do not have one clear owner that is defined in terraform, should be collected in the `blueprints/ACCOUNT_SHORTNAME-SHORT_REGION/s3/` of the account that owns the bucket. Work in progress to capture existing S3 assets in Terraform will begin by storing the buckets' .tf files in `blueprints/ACCOUNT_SHORTNAME-SHORT_REGION/s3/`.

## Rationale
### 1
We don't always like what it produces, but given the unique structure of the Terraform language, it's the cheapest/easiest way to maintain a consistent style/structure baseline for our code.

### 2
Terraform allows setting more than one "allowed account" in this parameter. However, because our templates are presently segregated by account, there are no use cases _at present_ of templates that work across accounts. Accordingly, we limit allowed accounts to one per template.

### 3
Because it's impossible to know without testing whether a prior minor release might not work correctly with your code due to a different implementation of a feature, or a bug that was fixed.

### 4
_account_ and _region_ constraints help assure that templates are not mis-applied across accounts or regions

Setting a provider _version_ in main.tf using [pessimistic constraints](https://www.terraform.io/docs/configuration/terraform.html#specifying-a-required-terraform-version) assures the oldest provider version with which the template is known to work will be used in the future:

### 5
This style looks more like code and is easier to comprehend as code than the alternative "quoted" form. Terraform documentation tends to show the "quoted" form, but everything works perfectly well without the extra quotes.
