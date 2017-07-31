# infrastructure-static
> infrastructure for static sites on AWS

Provision and deploy static websites to AWS. Use [Terraform](https://www.terraform.io) to define reproducable infrastructure as code.

## Installing / Getting started

`infrastructure-static` is intended to be used as a [git-subtree](https://www.atlassian.com/blog/git/alternatives-to-git-submodule-git-subtree) in a static site's repository

### Setup the subtree

```shell
git remote add -f infrastructure https://github.com/mwpeterson/infrastructure-static.git
git subtree add --prefix infrastructure infrastructure master --squash
```

### Update the subtree

```shell
git fetch infrastructure master
git subtree pull --prefix infrastructure infrastructure master --squash
```

From time to time, update the subtree to pull in changes, bugfixes, and new features from upstream.

## Developing

### AWS
You'll need an AWS account and security credentials.

```shell
brew install awscli
aws configure
```

### Terraform

You'll need Terraform installed.

```shell
brew install terraform
```

And each Terraform environment needs to be initialized to store remote state

```shell
project='the unique name of this static project'
(cd infrastructure/terraform/global; terraform init -backend-config "key=global")
(cd infrastructure/terraform/stage; terraform init -backend-config "key=$project/stage")
(cd infrastructure/terraform/prod; terraform init -backend-config "key=$project/prod")
```

### ejson

[ejson](https://github.com/Shopify/ejson) is used to encrypt secrets stored in Terraform's tfvars files. 

```shell
brew install ejson
mkdir -p $HOME/.ejson/keys
export EJSON_KEYDIR=$HOME/.ejson/keys # add to your ~/.bash_profile too!
```


### Add .gitignore to the .gitignore for this static site

```shell
cat infrastructure/.gitignore >> .gitignore
```

## Configuring Terraform for this static site

You'll need a terraform.tfvars.json file for each environment. They needed to be encrypted with ejson. `infrastructure-static` expects to find `prod.terraform.tfvars.ejson` and `stage.terraform.tfvars.ejson` in root of this static site project. Example unencrypted files can be found in `infrastructure/terraform`.

```shell
for f in infrastructure/terraform/*-example ; do g=$(echo $f| sed -e s:-example:: -e s:.*terraform/::); cp $f $g; done
# edit the prod.terraform.tfvars.ejson and stage.terraform.tfvars.ejson files
for f in *.tfvars.json; do ejson encrypt $f; g=$(echo $f | sed 's:.json:.ejson:'); mv $f $g; done
```

## Building

To build `stage` or `prod` 

```shell
environment=stage # or environment=prod
cd infrastructure/terraform/$environment
ejson decrypt terraform.tfvars.ejson > terraform.tfvars.json
terraform plan -out plan
# review the plan terraform will execute
terraform apply plan
```

If terraform prompts for variables, abort the operation and review the settings in terraform.tfvars.json and set whichever variables are missing


## Deploying 

`infrastructure-static` configures AWS CodePipeline and CodeBuild to deploy changes pushed to `develop` to the `stage` environment and to deploy changes pushed to `master` to the `prod` environment. The CodePipeline will be triggered when the infrastructure is built, so that the latest version of the site will be deployed.


## Contributing

To contribute, please fork the repository and use a feature branch. Pull requests are warmly welcome.

## TODO
- Automate ejson steps with a [Makefile](https://mtpereira.com/terraform-ejson.html)

