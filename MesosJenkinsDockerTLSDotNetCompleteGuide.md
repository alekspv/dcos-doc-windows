# Abstract

Purpose of this guide walk reader threw installation process of Jenkins case study with dynamic windows slaves.
Key feature here is configuring Docker on Docker on Windows nano server DCOS nodes.
Guide will walk you threw configuration steps and tehnical aspects of this case.

# Setup DCOS with Windows nodes
First follow [ Universal instsller Cloud Installation](https://docs.d2iq.com/mesosphere/dcos/1.13/installing/evaluation/) guide for you  clod and platform.
Curently  AWS and Azure are supported.
Please take a note that universal installer is tested under Mac or linux and not fully supports windows to run terraform.

For windows support use [Example main.tf](https://raw.githubusercontent.com/dcos-terraform/examples/feature/windows-support/aws/windows-agent/main.tf)

change module version to:
```
module "winagent" {
  source  = "dcos-terraform/windows-instance/aws"
  version = "~> 0.0.1"
```
run terraform as mentioned , for example in [AWS installation guide](https://docs.d2iq.com/mesosphere/dcos/1.13/installing/evaluation/aws/)
dont forget adding ssh key!!

```
terraform init
terraform plan -out=plan.out
terraform apply

```
terraform outputs windows password for administrator and cluster url
Foloow the url and login to  cluster with  prefered way.


# Setup software Prerequisites

Now install prerequisites for CI CD pipeline

## MarathonLB
install from catalog

## Nexus

- install from catalog
- login with default credentials
- create raw(hosted) repository called `dotnet-sample` and uncheck box "Validate format"

## Docker hub

- Log in
- Create `testworkload` repository.

## Example Workflow:

At DC/OS cluster setup following services:
- Jenkins from [Marathon-templates/jenkins.json](https://github.com/alekspv/TestWorkload/blob/master/Marathon-templates/jenkins.json)
- Nexus from DC/OS catalog, just specify the static port `27092` as it has been used in the pipeline
- Marathon-lb from DC/OS catalog

## Jenkins Framework Setup

use [Dominic's jenkins templete](https://github.com/DominikDary/TestWorkload/blob/master/Marathon-templates/jenkins.json)


### Jenkins configuration

[Early  refference](https://github.com/alekspv/TestWorkload/blob/master/README.md)
TODO

# Build and deploy .Net applocation

## Build .Net App

## Publish the App to Nexus

## DockerTLS Setup

## Build Docker image with the App and publish to Docker registry

# Deploy To DCOS cluster
