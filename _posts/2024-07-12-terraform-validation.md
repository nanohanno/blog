---
layout: post
title: Terraform sanity checks
subtitle: When do they happen?
date: 2024-07-12
background: /images/yosef-futsum-ZAvhxLTcSok-unsplash.jpg
---

Terraform is a declarative configuration language for infrastructure. It is often used to configure and create cloud resources. Its declarative nature means that the state and configuration is defined in which the resources should exist or at least get created in. It is agnostic about the way how those resources are created or changed.

## Validation

One nice aspect is that the configuration can be checked before actually making any changes. The command `terraform validate` checks if the syntax is correct. Are the resources referenced correctly, are parameters in modules referenced correctly. A quick static sanity check without taking into account variables or remote state.

## Plan

The next stage is to call `terraform plan` where the configuration variables are imported and the configuration is compared against the remote state to see how it would change. While creating that execution plan, the configuration checks in the terraform provider code are also executed. These are modules that connect terraform to a specific API of a resource provider like AWS for example. The [terraform aws provider](https://github.com/hashicorp/terraform-provider-aws) offers resource definitions for all aws resources and implements their creation by integrating the go client for the AWS API. Here, validation checks for configuration parameters can be implemented that are called during plan execution, like required parameters, type or number of parameters.

## Apply

Unfortunately, it is a matter of the owner of the provider which validation checks are integrated. Finally, the API of the resource provider also does some validation checks on the given parameters for a resource. This write API is just called during the `terraform apply` stage when the resources are up for creation. That last step is necessary but sometimes a little late. We already have checked it beforehand and are creating resources but for some the creation might still fail.

In my case, a resource needed to be updated and therefore destroyed and re-created. After destruction of the old resource there was a validation error from the API that one parameter was missing which was apparently mandatory for the updated configuration. The creation of the new resource failed and eventually no resource existed. This result would be disastrous in a production setting where a service gets updated but the result is that it is just removed without creation of a new service.
