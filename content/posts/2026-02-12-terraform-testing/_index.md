---
authors: ["Pieter Vincken"]
title: "If infrastructure as code is code, why don't we test it that way?"
description: ""
date: 2026-02-12T09:00:00+01:00
slug: ""
tags: ["cloud", "iac"]
categories: ["Cloud"]
series: []
cover:
    image: images/cover.png
---

> Quote: If it hurts, do it more often. 

## Problem 

When writing Infrastructure as Code (IaC) the typical the cycle goes more or less like this: 

1. Write some code
2. Execute the code
3. Validate that it has the desired result by manually checking the result in the target system

The result of these actions often looks something like this: 

1. You've made a silly syntax error on line one
2. The code tries to execute but the backend system returns an error
3. The code executes successfully, but doesn't configure the target as you expected
4. Everything works as you intended

Unfortunately, like any programming, scenario 1-3 occurs vastly more often than scenario 4.

In application development, this is mostly a solved problem
You write all kinds of tests to increase your confidence that the code performs as you expected. 
For decades now, application developers have written unit test, integration tests, end-to-end tests, frontend test, smoke tests, automated regression tests, production acceptance tests ...
There is even the Test-Driven-Development (TDD) methodology where you write tests before writing any application code.

So, what can we learn from application development best practices to improve our Infrastructure as Code rollouts?

## Goals and ambitions

In this blog post, we'll discuss an approach you can take when writing tests for Infrastructure as Code.
The post will use HashiCorp Configuration Language (HCL) and CNCF OpenTofu to demonstrate an approach you can take.

The framework will focus on solving four issues with writing IaC: 
- Long feedback loops
- Low confidence when execution changes
- Hard to prevent regressions in code over time
- Scale testing parallel changes in the same environment

We'll use the same definitions for different parts of the IaC as described in the previous blog post.
If you haven't read that, make sure to at least read the section about the [definition of the module and orchestration repository](/posts/2026-01-21-terraform-structure/#definitions).

## The solution

As with the previous blog post about IaC code structuring, this post is also a suggestion on how to do it and might be a good jumping off point for you to expand on yourself. 
The framework (which might be too much credit...) defines 4 main tests: the unit test, the integration test, end-to-end tests and the deployment validation.

### The unit test

First, let's start with a look at the most fundamental type of test used in application development: unit tests. 
These tests are only relevant for module repositories.
They validate that our configuration performs the actions we expected it to do based on the input and context we provide.

A unit test comes in many flavours, but we'll use the well-known [given-when-then](https://martinfowler.com/bliki/GivenWhenThen.html) structure, although the lesser known 4-phase test would be even more fitting.

Given: a context and input variables
When: an OpenTofu Plan or OpenTofu Apply
Then: assert provisioned infrastructure and output variables
(Teardown): delete any provisioned infrastructure

Let's look at an [example](https://github.com/pietervincken/iac-testing/tree/main/demo/1-simple-module-repo/standalone-module).
Consider a simple blob storage module with just 2 resources: an S3 bucket and a bucket policy that prevents public access.
Our module takes two inputs, an env(ironment) and a name and it produces a bucket name and a bucket ARN.

If we follow our given-when-then-teardown (GWTT) structure, we get the following test scenario: 

```hcl
run "minimal" {

  variables {
    env="tst"
    name="standalone-module-test"
  }

  command = plan

  assert {
      condition     = strcontains(aws_s3_bucket.this.bucket, "standalone-module-test")
      error_message = "S3 bucket name doesn't contain the standalone module name"
  }
}
```

**Given** an env named `tst` and a name `standalone-module-test` {{< line_break >}}
**When** we perform a `plan` {{< line_break >}}
**Then** we assert that the name of the bucket contains `standalone-module-test` {{< line_break >}}
**Teardown** is implicit. Because we only performed a plan, no infrastructure was provisioned so there is nothing to cleanup. 

This is a very basic test and doesn't really validate our module.
It only tests two things: 
1. That the module can be used in a plan, which means that it passes a syntax validation
2. That the input we provide is mapped correct to be part of the bucket name. When using `plan`, we can only test inputs, locals and resource arguments. 

Let's look at a more extensive example. 
```hcl
run "minimal" {

  variables {
    env="tst"
    name="standalone-module-test"
  }

  command = apply

  assert {
      condition     = strcontains(aws_s3_bucket.this.id, "standalone-module-test")
      error_message = "S3 bucket name doesn't contain the standalone module name"
  }

  assert {
      condition     = contains(keys(aws_s3_bucket.this.tags_all), "environment")
      error_message = "S3 bucket environment tag should be set"
  }

  assert {
      condition     = aws_s3_bucket.this.tags_all["environment"] == "tst"
      error_message = "S3 bucket environment tag is not set to 'tst'"
  }

  
  assert {
      condition     = output.name == "s3-tst-standalone-module-test"
      error_message = "S3 bucket name must follow the naming convention"
  }
    
  assert {
      condition     = strcontains(output.arn, "s3-tst-standalone-module-test")
      error_message = "S3 bucket ARN doesn't contain the naming convention"
  }
}
```

We follow the same GWTT structure: 

**Given** an env named `tst` and a name `standalone-module-test` {{< line_break >}}
**When** we perform an `apply` {{< line_break >}}
**Then** we assert that the `name` of the bucket contains `standalone-module-test` {{< line_break >}}
**Then** we assert that the `environment` tag exists and is set to `tst` {{< line_break >}}
**Then** we assert that the output variable `name` is `s3-tst-standalone-module-test` {{< line_break >}}
**Then** we assert that the output variable `arn` contains `s3-tst-standalone-module-test` {{< line_break >}}
**Teardown** is implicit here. OpenTofu will delete all resources created in the test scenario automatically after the test has completed.

There are two differences. 
The `when` is an `apply`. 
This means that we provision infrastructure and execute the assertions after the provisioning succeeds. 
It also causes the `teardown` to destroy the infrastructure provisioned by the test.
The second difference is that we assert the attributes of resources and the outputs of the module.

As we provision infrastructure, we can extend the test scope significantly.
It makes our tests much broader and increases the confidence gained from a successful test.
However, there always is a but isn't there. 
Additional care needs to be taken to prevent side-effects from causing undesired randomness and flakiness in our tests.

Unit tests should validate that our IaC is working as intended, it should not validate any side-effect generated by the provider.
This is the reason that the ARN test only validates that the name is present and doesn't match the full ARN.
The structure of the ARN is a side-effect created by AWS, not by us, so we shouldn't validate it.

As a final, personal, request, please write error messages and make them meaningful.
Together with the filename, the expected value, the actual value and line number already provided by OpenTofu, the error message should accelerate your debugging process. 
Describing why you expect the value to be the expected value is a good starting point for the error message. 

```hcl
assert {
  condition     = output.name == "s3-tst-standalone-module-test"
  error_message = "S3 bucket name must follow the naming convention"
}
```

### The composition tests

The next type of tests are integration, component or contract tests. 
These have many names in traditional application development.
The main goal of the tests in our framework is to validate that the module under test can be used together with other (remote) modules.
They should provide you with the confidence to regularly update your modules and upgrade incoming dependencies as they validate that you or a change in a remote module, haven't broken any existing scenarios you rely on. 

Where it's important to try to validate as much as possible in unit tests, you want to be more cautions to what scenarios you test in composition tests. 
They should increase confidence in modules working together.
They should not test incoming dependencies, nor should they test inner workings of the module under test. 

Let's look at an example.
The module under test will be a default alarms module for virtual machines.
This module has a dependency on a virtual machine and a virtual network being available before it can be deployed.
We'll show how we use the VPC and EC2 module to deploy the necessary resources for the module to be tested. 

```hcl
run "setup_vpc" {
  module {
    source = "../vpc-module"
  }
  variables {
    env="tst"
    name="test-fixture-vpc"
  }
}

run "setup_ec2" {
  module {
    source = "../ec2-module"
  }
  variables {
    env="tst"
    name="test-fixture-ec2"
    subnet_id = run.setup_vpc.subnet_ids[1]
  }
}

run "minimal" {
  variables {
    env="tst"
    name="test-fixture"
    instance_id=run.setup_ec2.instance_id
  }

  command = apply

  assert {
      condition     = try(length(output.ids) == 2 ,false)
      error_message = "Expected 2 CloudWatch alarm IDs to be returned"
  }
}
```

The different run blocks in a test script are executed in order. 
This means that we first deploy the VPC module, then the EC2 module and finally the alarms.
We can pass information between the different runs by referencing the output variables of modules deployed in a previous run in the input variables for the next test. 
This is how we pass the `subnet_id` to the EC2 module and the `instance_id` to the alarms module. 

The assertions we execute in this test should be limited to validating that the module deployed as expected. 
In the example, we simple check if there are 2 IDs returned by the alarms module.
In more complex examples you could for example validate that the alarms are connected to the virtual machine. 

### End-to-end tests

The first two types of tests focussed on module repositories.
The next two types are used to test orchestration repositories.

Just like with end-to-end tests in application development, here our tests will also have a different lifecycle compared to the actual provisioned infrastructure. 
The tests are short-lived while the provisioned infrastructure has a long(er) lifecycle.
We typically don't want to immediately destroy the infrastructure we've provisioned.

To achieve this, we split our tests from our infrastructure provisioning.
By using the same tool(stack) for both provisioning and testing, we only need to learn and operate a single tool.

Let's dive into an example orchestration repository setup. 
We have an orchestration repository that provisions all the modules we've discussed so far, our VPC module, EC2 module, EC2 alarm module and the S3 bucket module.

We perform the provisioning of the resources just like you're used to. 
When the infrastructure is provisioned, the tests can be executed in a separate OpenTofu run. 
In the test run, we only have data blocks in our `main.tf` to fetch information.
We do this to make our test fixture(s) (aka specific testing modules) reusable across orchestration repositories. 

Example main.tf
```hcl
data "aws_s3_bucket" "this" {
    bucket = "s3-prd-orchestration-example"
}

output "bucket_name" {
  value = data.aws_s3_bucket.this.id
}
```

Now, let's have a look at the tests.

Example main.tftest.hcl
```hcl
run "data" {
  
}

run "create_s3_object" {
  module {
    source = "./create-s3-object"
  }

  variables {
    bucket = run.data.bucket_name
  }

  command = apply
}

run "validate_s3_object" {

  module {
    source = "./validate-s3-object"
  }

  variables {
    bucket = run.data.bucket_name
  }

  command = apply

  assert {
      condition     = data.aws_s3_object.this.server_side_encryption == "AES256"
      error_message = "S3 object is not encrypted with AES256"
  }
}
```

The first run block just provisions the data block that we've shown in the `main.tf` file.
The second run block is used to validate that we can use the S3 bucket we provisioned. 

It uses a module to create an object in the bucket.
By using a module in the test itself instead of in the tests `main.tf` the module can be re-used across multiple scenarios.
When writing these scenarios follow the same principle as with regular IaC, if it's just used once, embed it. 
If it's used across multiple tests or repositories, turn it into a reusable module. 

A second module is used to fetch all information of the S3 object. 
The information is used to validate that the correct default encryption is set for the S3 object. 

Finally, because all these modules are part of the test scenario, all provisioned infrastructure will be deleted again after the tests have completed (or failed). 
This makes sure that the S3 object just in our tests is deleted again. 

As the actual provisioned infrastructure is not part of the test setup it's left as is, which is exactly what we want.

### Bonus: Deployment validation tests

As a bonus, let's have a look at deployment validation tests. 
These types of tests are mostly relevant in (highly) regulated environments where you need proof that a change you intended to execute executed properly.
These scenarios are relevant in for example financial services (BaFin, DORA regulations) or the field of medicine (GxP).

In most cases, the required evidence is captured either through audit reports that are checked at regular intervals, by audit following along when operations execute a pre-defined runbook or even operators being required to record their actions through screen capture and providing evidence in the form of screenshots that the desired state was reached. 

These approaches of course don't align with one our idea that everything worth doing twice is worth automating. 
So let's have a look at how OpenTofu can help in automating these tests as well. 

The testing setup is very similar to what we do in the end-to-end tests. 
The main (or even only) difference is that we explicitly don't provision **any** additional infrastructure to validate that our deployment worked.
This means that all tests should only **fetch** information through data blocks and perform assertions on those. 
As with the end-to-end tests, the deployment validation tests are separate from the actual infrastructure provisioning. 
The tests should validate that the environment is what you expect it to be after your deployment. 

There are two main reasons to write your validation tests this way: 
- There is no additional framework or language to learn if you're already provisioning your infrastructure using OpenTofu. 
- They don't interfere (or even directly depend) on your infrastructure provisioning process. 

These validation tests could even be used in a setup where the infrastructure provisioning isn't even done using OpenTofu or Terraform at all. 
This can be helpful in scenarios where the provisioning is performed by another team or even by a third party. 

## Conclusion
In an ideal world all of these tests would integrate directly with our existing testing frameworks and tools. 
There is an [open issue on OpenTofu's GitHub](https://github.com/opentofu/opentofu/issues/2501) to add support for outputting the test results in a well-known format like JUnit.
Unfotunately, at the time of writing (February 2026), the feature is not implemented yet.

The framework we discussed hopefully provides you with a good foundation or at least a jumping off point to start implementing your own testing setup.
Like in regular application development there is extra effort involved in setting up and maintaining tests over time. 
And the line between what to test and under what conditions can be hard. 
But investing in it will pay off as your confidence in rollouts will increase over time. 
And when a bug eventually occurs, you'll have a way easier time fixing it reliably without having the fear of breaking existing usages. 

Testing infrastructure as code should also fit into your larger infrastructure provisioning process.
It's crucial to make sure that the different components work together in your landscape. 
You can (and should) still use policy checks (like OPA) to prevent deploying non-compliant resources. 
Your modules should provide a compliant-by-design implementation of your resources. 
This should accelerate teams in being compliant by leveraging already complaint components. 
In a typical shift left approach, you don't want to have your deployment fail on a policy check, but you want your unit tests to fail when creating a new version of your module. 

Furthermore, the methods discussed in this blog post are not a one-to-one copy of their application development counterparts. 
Unit tests should typically prevent relying on external dependencies when executing, but in our use case there is no way to avoid it. 

Finally, let me know what you think about the framework.
Does it work in your context? 
What would you solve differently and are the caveats you ran into?