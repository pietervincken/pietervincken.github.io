---
authors: ["Pieter Vincken"]
title: "How to structure infrastructure as code?"
description: ""
date: 2026-02-06T09:00:00+01:00
slug: ""
tags: ["cloud", "iac"]
categories: ["Cloud"]
series: []
cover:
    image: images/cover.png
---

> Quote: Everyting worth doing twice is worth automating. 

## Problem 

Enterprise cloud environments often consist of 1000s if not 100000s of resources that need to be managed somehow to have a working application for a customer of the enterprise. 
Operating this many resources is challenging, especially when these environments tend to be subject to constant change, both from the cloud provider side and from the enterprise requirements side. 

For some time now, Infrastructure as Code (IaC) has been the go-to solution for enterprises to improve their quality of life in operating these large cloud environments. 
In essense, using automation to improve the throughput a single team can deliver is nothing new. 
Our datacenter colleagues have been using automation since the early 90s to prevent having to go to the racks to install new software or to repeat that same installation over and over again over a fleet of 100s of servers in the datacenter. 
Using automation or infrastructure as code is a no-brainer today for managing even smaller environments.
As the famous saying goes: everything worth doing twice is worth automating. 

In the last ten years, Terraform has become the de-facto standard Infrastructure as Code framework of choice.
With OpenTofu as a runtime, it might be the best solution on the market today for operating cloud environments at scale.
Terraform solved the problem of how to easily, consistently, confidently and safely rollout changes to 1000s of resources in a cloud environment. 

However, probably the most commonly asked question in the community and in enterprises is: how do we structure our IaC codebase?
Creating the right structure for an enterprise IaC setup is a difficult balancing act between at least the following concerns: 
- Team autonomy
- Codebase ownership
- Timely feedback loops
- Quality guarantees

In this blogpost a framework will be discussed based on the best practices combined from experience with multiple enterprise IaC setups. 
A special shoutout to the [Terraform Best Practices](https://www.terraform-best-practices.com/) by Anton Babenko that were a great starting point. 

## One-size fits no-one

Before we start, a quick disclaimer. 
This a framework that worked for our projects. 
As with most of frameworks or structures, this is not a one-size fits all. 
If your organizations is similar in size and framework, this should be a great jumping off point. 

This framework is aimed at organizations with the following characteristics. 

- A central Cloud Center of Excellence operates as an Enabling team (in the definition of the [Team Topologies](https://teamtopologies.com/)) for all cloud related cross-functional requirements. 
- You have clear ownership of applications, components and capabilities defined. 
- (Optional, but it helps) The development teams are responsible for operating their workloads, including production.
- (Optional, but it helps) There are requirements (regulatory or internal) w.r.t. testing infrastructure changes and providing a guarantee on their execution in production. 

If you don't fit these requirements, don't worry, your likely to find good guidelines in this blogpost. 

## Definitions

This framework relies on using Git repositories to split the different component into separate manageble blocks. 
It has 2 definitions for these repositories: 
- Module repositories
- Orchestration repositories

Let's start with module repositories as this definition is quite common. 
These are repositories where one or more reusable modules are defined in a single repository. 
For the scope of this framework, these module repositories should house a single capability that can be versioned as one component for reuse. 

The second definition is an orchestration repository. 
This a repository that houses code that can be deployed, as-is, to an environment. 
All components and information to deploy the resources should be available to or in this repository. 
Another good way to define it is that one orchestration repository houses one Terraform state. 

Now there is a third definition, which is not a separate repository, but still an important concept: an embedded module. 
An embedded module is exactly what is says on the box: a module embedded into either another module or an orchestration repository. 
It's important to note that it's versioned with the parent context and should only be used as a way to improve the codebase structure within a repository. 

## The rules

This next section will discuss the different rules and guidelines that apply to each of the components. 

### Module repository

The goal of a module repository is to provide a common way to deploy, operate and managed a specific capability within your organization.
The focus should be to abstract complexity away from teams by providing sane defaults, integrate with shared infrastructure, provide a minimal interface and is compliant by design with your organizational policies. 
A module should provide an organizational interpretation of the capability in an oppinioated way. 

Therefore the following rules apply to a module repository. 

#### A module repository must have single version

A module repository must be automatically versioned and should only have a single version associated with it. 
This makes sure that a user of the module set the version of the module and can use that version of the module with any embedded modules in that module repository with the same version. 
This prevent a hard to maintain setup with different tags and versions in a single repository and guarantees version consistency when using the module. 

The versioning of the module should follow the standard semantic versioning schema. 
This is important for automations like renovatebot or dependabot to easily identify upgrades and suggest changes. 
Using tools like [release-it](https://github.com/release-it/release-it) can be used with conventional commits to make this a breeze to do. 
Out of the box changelogs are an added bonus when using conventional commits. 

#### A module should have an automated release process

This framework can be used with git flows: trunk based development, git flow or Gitlab flow. 
Technically, the only requirement is that a given commit is tagged with a specific version tag that follows [semantic versioning](https://semver.org/).

When a release is created, either automated or developer initiated, a new version number should be automatically calculated and attached to the commit.
Releasing a version of a module should only be possible after the necessary tests (discussed later) have passed. 

While strictly not required, it's recommended to include documentation and changelog generation in the automated release process.
This guarantees that these are up to date with the associated tag. 

#### A module should use composition

To support reusability of modules, modules should be written to be using the [composition design pattern](https://developer.hashicorp.com/terraform/language/modules/develop/composition#module-composition).
Composition makes sure that modules can be used to augment other modules.
It prevents that modules become overly complex or violate one of our core principles: do one thing and do it well. 

This also implies that modules should prevent embedding other remote modules. 
Using a remote module implies that the module needs to be updated at least every time the remote module updates. 
Therefore careful consideration needs to be taken that the usage of the remote module provides enough benefit. 

Another consideration when using public modules is having a proxy registry for them. 
By using public modules from public sources, a direct dependency is created between the rollout of an orchestration repository using the dependency and the availability of the remote source. 
Although rare, registries can go down and not having access to the source will prevent rollouts from being performed. 
Therefor it's highly recommended to use a proxy or pull-through registry for Terraform components. 
An added benefit of a proxy or pull-through registry is that modules with unwanted licences can be blocked before they are used in your organizations codebase. 

A simple example of this is a virtual machine module and the monitoring of that virtual machine. 
By using the composition pattern to provide these two capabilities the modules can be used separately and their functionality isn't combinited into one large module which reduces the complexity of maintaining the module. 
The interface between these modules should be only the reference of the virtual machine. 
If a virtual machine is deployed through other means, the monitoring module should still be usable.

#### A module should have a minimal interface (input and output)

This rule is one based on experience and against what is commonly done in open source Terraform modules. 
The idea is to limit the amout of inputs and outputs to only the ones that a user of the module needs.
A discussed earlier, the focus of a module should be to provide a compliant way to provision infrastructure in accordance to the organizational requirements. 
By providing only the necessary input variables to make changes to the modules behavior in a way that is compliant with the organization policies, it makes the life of both the user and the maintainer of the module a lot easier. 
A good rule of thumb is to only expose the inputs and outputs of a module for the use cases that it's supporting. 

When creating a new module, start with an MVP implementation and only expose a handful of input variables. 
If a user of the module needs an additional input variable, this either means they cannot (or should not) use this module as they are trying to serve a different use case or they simply create a PR to add the variable as adding a variable with a sane default is a non-breaking change. 

Removing a input or output variable on the contrary is often a breaking change as all it's usage needs to be removed from all places where the module is used before the variable can be removed.

Output variables should support linked components together in an orchestation repository.
Again, limiting the amount of output variables makes maintaininig modules over time a lot easier.

Finally, a module should not rely on environment information to function. 
All information it needs to operate should be provided through input variables. 
This means that hardcoded environment information or the usage of data blocks should be prevented. 

#### A module should have documentation

Having good documentation for modules is crucial to promote their usage across your organization. 
Module documentation should have at least: one minimal example of the module being used, one with all the possible input variables being used, a clear description of the use case it's solving and documentation of all input and output variables. 

Tooling like [terraform-docs](https://github.com/terraform-docs/terraform-docs) can help in partially automating this process. 
If all input and output variables have a description, type and optional default value, this tool will automatically generate Terraform Registry like documentation for the module in the README in the module repository. 

This README should also have the minimal and full example of the module being used.
When creating the examples it can be helpful to include any other resources or modules in the example. 
This way the user of the module has a full example to get them started.

#### A module should have tests

Modules should have at least one of each of the following tests: 
- A unit test for every input variable
- A unit test for every output variable
- A unit test for every conditional scenario in the module
- An integration test for the minimal example
- An integration test for the full example

If you're not aware of how to test Terraform codebases, no worries, there is a future blogpost in which this will be discussed in detail. 

### Orchestration repository

As orchestration repositories are intended to be out as is. 
This means that they require all information to be available in the repository. 

#### An orchestration repository must be deployable at all time from the default branch

As orchestration repositories are part of the operations of an application or workload, it's helpful to have the default branch always in a deployable state. 
This helps in incident or debugging scenarios by being able to go to a known good state. 
It can also help to identify manual changes outside of the IaC setup and should trigger an investigation if this occurs in production environments. 

#### Automation for the orchestration repository should include automated organizational security and policies testing

This is a typical shift-left requirement. 
When automation is setup to deploy orchestration repositories to an environment, including all security and policy checks up front, prevents costly remediation actions down the line. 
Tools like [Trivy](https://trivy.dev/) or [OPA](https://www.openpolicyagent.org/) can help in performing standardized policy tests as well as organization specific checks that your organization requires. 
When these tests are embedded into the rollout process, especially of non-production environments, the workload teams will be more aware from the start about what is expected from them. 
As a nice bonus, if your modules are compliant by design, it's a huge selling point to the workload teams to use your modules instead of creating their own. 

#### An orchestration repository should not have input variables

Input variables need to be passed to a Terraform run, either through environment variables or configuration files. 
Generally, this is a bad practise as this makes the Terraform run less portable and more dependent on the system performing the Terraform run. 
It also typically creates unneccesary hassle with having configuration managed in different systems. 
As an orchestration repository is only aimed at deploying a single deployable context (workload, application, ...) to a single environment, it's perfectly fine to hardcode any input variables that might be needed: environment name, name of the workload, ...
To adhere to the [DRY principles](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself), these hardcoded values can be stored in local variables to make using them across different module instances easier. 

#### An orchestration repository must not have complex resource structures

Orchestration repositories should prevent having complex resource structures. 
If these are needed and a module doesn't exist for it's use case (or the use case is specific to the environment) embedded modules can be used to house that complexity. 
The root of an orchestration repository should only house modules and simple data blocks.
This makes sure that the repository is easy to read and understand. 

#### An orchestration repository should have timely feedback cycles

This one is a simple one. 
As a rule of thumb, if a single plan takes more than 10 minutes, look into splitting the repository into multiple orchestration repositories. 
In a CI/CD setup where a feature branch does a plan, the default branch does a plan and apply, a single plan of 10 minutes means that a single rollout would require at least 30 minutes to complete end-to-end. 
It's not always possible to split the runs into multiple runs, but the 10 minutes threshold should be a trigger to consider a split. 

#### An orchestration repository should have automated post-deployment validation checks

These checks should validate that the deployment behaved as expected and that all expected modules have been deployed. 
In a future blogpost these tests will be discussed and demonstrated. 
For now it's just important to note that having these automated post-deployment validation tests should improve the rollout confidence. 

#### An orchestration repository should prevent direct dependencies with other orchestration repositories

Orchestration repositories often need information about the environment they're deployed into. 
Typically, there are two methods that are used for this purpose: data blocks and remote state reads. 
Both create direct dependencies on other infrastructure deployments to be succesful.
The data blocks are a bit of a looser requirement as they represent actual infrastructure being present. 
The remote state reads introduce a direct coupling between the remote state and the orchestration repository. 

This couple typically introduces hard to debug issues and requires multiple successful deployments before a change can be executed. 

Too prevent this couple, removing the dependency outright is of course the easiest option. 
This is however not always possible. 
The next best thing is to either hardcode the reference or using data blocks to discover the necessary components. 

While using hardcoded references is the easiest, it comes with the significant drawback that there is no link with the actual infrastructure being present.
This could result in successful plans, but failed applies. 

When using data blocks, the rollout will actually check for the existance and access to the resource. 
While this prevents rollouts from occuring without the infrastructure being present, it does require that the automation has access to the actual underlying infrastructure to fetch the data through the data block. 

Finally, there is the [remote state data block](https://developer.hashicorp.com/terraform/language/state/remote-state-data).
While this seemingly allows to have both the referential integrity without needing access to the actual underlying infrastructure, it introduces a direct dependency between both automations.

All of this results in the following recommendation when references are needed in an orchestration repository: 
- Hardcode the reference, but use it sparsely
- Use data blocks if there is very little risk of the underlying infrastructure not being available and there is no risk in having access to the resource (E.g. availability zone data block or subnet data block based on a VPC/VNET ID)
- Prevent the usage of remote state data block at all cost. They introduce hard dependencies between runs which are hard to debug and often cross team boundaries which makes coordinating rollouts even harder. 

### Licensing checks

When using public modules or 3rd party provided modules it's important to check the licenses against your organizational policies. Most open source modules are licensed under MIT, GPL or the Apache 2.0 license. 
Depending on your industry, usage pattern and organization policy w.r.t. permitted licences. 

Next to checking the licences of public modules, it's also important to add a license to the internally build modules. 
Again, depending on your organization a specific license should be added to the code. 
This prevents the code from being used or shared outside of your organization.

Automation can be used to perform the necessary checks and make sure you're compliant with your organizations policies. 

## Conclusion

This framework with it's rules are a suggestion on how to structure your Terraform setups.
It's by no means a one size fits all and it definately doesn't accomodate all setups. 
It tries to allow for flexibility, especially w.r.t. the interactions with other teams. 

If you're part of a Cloud Center of Excellence or are part of a platform team please consider the following tips in order of importance if you want to improve your Infrastructure as Code usage in your organization: 

- Ask your workload teams what problems you could solve for them.
- Automate your IaC rollouts. This includes generating documentation, automated versioning ,and security and policy testing. 
- Offer an IaC as a service to your workload teams. Allow them to provision their own infrastrucure.
- Offer versioned modules for common scenarios in your organization. Make them compliant by design.
- Implement automated testing across both your module and orchestration repositories.