---
authors: ["Pieter Vincken"]
title: "KubeCon | CloudNativeCon Europe 2026: Amsterdam edition"
description: ""
date: 2026-03-26T09:00:00+01:00
slug: ""
tags: ["cloud", "kubecon", "cloudnativecon", "open sovereign cloud", "opentofu", "testing", "sbom"]
categories: ["Cloud"]
series: []
cover:
    image: images/cover.png
---

> KubeCon | CloudNativeCon Europe, it always feels a bit like coming home.

# KubeCon | CloudNativeCon Europe 2026

## Co-located events aka day-zero

I attended Open Sovereign Cloud Day and OpenTofuDay.

### Open Sovereign Cloud Day

As sovereign cloud becomes more and more of a topic in our industry, I was interested in learning more about what's already available in the cloud native ecosystem. 

This first talk by Emiel Brok discussed the [EU framework for sovereign cloud](https://commission.europa.eu/document/download/09579818-64a6-4dd5-9577-446ab6219113_en). 
It's a good primer for anyone interested in sovereign cloud and how the EU framework can be used in practice.
If you're working in the EU and want to understand how the framework can be applied to your organisation, this talk is a good starting point.

[SUSE has a self-evaluation tool](https://www.suse.com/cloud-sovereignty-framework-assessment/) that can be used to evaluate a solution (or even your IT organisation, not recommended) against the EU framework.

The second talk was by Antoine Tielbeke, who explained that it's more important to start implementing sovereign cloud practices than to wait for the perfect solution.
He provided 3 recommendations to get your organisation started: 
- Know your crown jewels
- Close the tap
- Keep the lights on

The first recommendation is to identify the most critical data and applications in your organisation and prioritise the controls relevant to their sovereignty.
Aka start with what really matters. 

The second recommendation is to prevent new solutions from using non-compliant mechanisms or, at the very least, make them aware of the impact of their choices.
This idea is to stop doing the wrong thing before doing the right thing.

The final recommendation was one that I didn't expect, but makes a lot of sense: keep the lights on.
This means that your organisation should continue to operate and provide value to its customers while implementing sovereign cloud practices.
Antoine provided the example of the observability stack.
If your observability stack becomes unusable or you tell your teams they cannot use it anymore if they want to be compliant, this puts a lot of pressure on the teams and can lead to a lot of resistance.
Because now they have to both develop their new applications to be compliant, but also figure out how to observe and troubleshoot their applications without the tools they are used to.

### OpenTofuDay

I had the privilege of speaking at OpenTofuDay about how testing Infrastructure as code can be done using OpenTofu and Localstack. 
The gist of the talk is that you can use the OpenTofu testing framework to write tests as you'd do for traditional software development. 
You can create unit tests and composition tests for modules and end-to-end tests for orchestration repositories. 
If you want more info, check out one of my previous blog posts, which goes into more detail. 

Two other talks I'd like to highlight are the ones from James Humphries and Robert Glenn. 

James discussed how they (the OpenTofu maintainers) have integrated OpenTelemetry into OpenTofu to provide better observability and debugging capabilities for users.
His demo showcased how you can connect OpenTofu to Jaeger using OpenTelemetry and get detailed insights into the execution of your code. 
This will be a great tool to improve execution time as it clearly identifies bottlenecks in the OpenTofu execution, especially if provider support is added in the future. 

Robert discussed how excessive use of complex expressions can lead to very hard-to-read code.
His recommendations were very practical: split the expressions into different local variables and give them meaningful names.
This will make the code self-documenting and prevent complex expressions from being spread across the codebase.

## KubeCon | CloudNativeCon Europe 2026

### First day

IBM Research started the day off for me with a talk about [Kagenti](https://kagenti.github.io/.github/). 
Kagenti is a platform combining multiple existing open source tools to deploy, run, secure and govern AI agents in a Kubernetes cluster. 
For me, the talk was specifically interesting in that it explained how they are combining user and workload identities to allow AI agents to act on behalf of users without losing the connection with the user. 
This triggered an interesting thought with me as combining these identities could be extended to also include the signature of a workload to really guarantee that it's "on behalf of a user", "running in a specific environment" and "executed by a trusted binary".

The second talk of the day was another great discovery for me. 
Mercedes-Benz and Cloudbase solutions shared [GARM](https://github.com/cloudbase/garm): a tool that automates runners for GitHub Actions on many different platforms. 
As someone who has first-hand experience with operating self-hosted runners for many different CI/CD platforms, I can say that this is a very welcome addition to the ecosystem.
GARM supports multiple platforms (bare metal, VMs, Kubernetes) and multiple cloud providers (AWS, Azure, GCP, OpenStack) and can be used to easily deploy and manage runners for GitHub Actions. 
I attended the talk because Mercedes-Benz Tech Innovation always has interesting talks about how they are using cloud native technologies in their organisation, and this talk definitely stayed true to that reputation.

Next to AI, sovereign cloud is another hot topic at this KubeCon. 
One of the better talks about this was from Alexandra Hou Aldershaab (Eficode) and Thomas Vitale (Systematic):  The Developer’s Nightmare: How To Survive Compliance Checklists (and Still Ship Fast). 
They explained (through one of the better story tellings I've seen at a conference) how they suggested using SBOMS, signed commits and automation to prevent lengthy compliance approval processes while still following the necessary compliance requirements.
[Dependency Track](https://dependencytrack.org/) for SBOM management makes an appearance again, as well as [cosign](https://github.com/sigstore/cosign), as tools that can help in this process. 

The final talk of the day was about [Talos Linux](https://www.talos.dev/) and [Zarf](https://docs.zarf.dev/ref/packages/). 
Merijn from TrueFullStaq showcased the management of a three-node cluster live on stage. 
He bootstrapped a cluster and deployed an application using Zarf packages. 
If you'd like to know what application he deployed, definitely check out the talk. 
I'll give you a hint: of course it can run ....
While the cluster was performing actions in the background, Brandt explained how Talos, in combination with Zarf, can be used to manage clusters in air-gapped environments.
It was interesting to see that Talos itself can be managed using Kubernetes running on the Talos nodes. 

### Day two

This day, there was a clear theme in the talks I attended: platforms. 

There was a great talk by LY Corporation about how they built their internal Bare-Metal-as-a-Service platform using Kubernetes and OpenStack. 
They explained how they use OpenStack to manage the lifecycle of their bare metal servers and Kubernetes to provide a self-service platform for their internal teams to request and manage their own bare metal servers.

Two other talks I'd like to highlight are from NeoNephos and the Dutch Tax Authority.

The Dutch Tax Authority shared its journey of building a platform for its internal teams to deploy and manage their applications.
I especially liked that they didn't only focus on the technical capabilities but really sold their platform internally through all kinds of internal marketing activities, ranging from focusing on good documentation to hosting one-day workshops to get teams started on their platform. 
They even host internal user group sessions specific to the platform, where they share new developments and best practices.
Special shoutout to the fact that they open-sourced their ["project-as-a-service" operator](https://github.com/belastingdienst/opr-paas) that helps bootstrapping a project for teams. 

The talk from NeoNephos introduced me to their foundation and the fact that there is an open source project that combines multiple (CNCF landscape) tools that can work together to provide a sovereign cloud platform. 
The presentation only highlighted what their goal is and hinted at what components are already part of their platform, but this is definitely something to keep an eye on in the future. 

The rest of the day was filled with talking to industry friends and speaking to vendors in the sponsor showcase.
As this is my 6th KubeCon, this is probably the most underrated activity at the conference. 
Having conversations with industry peers and vendors can provide a lot of insights and help you stay up to date with the latest developments in the ecosystem.
For talking to vendors, it helps to have specific use cases or problems in mind that you want to discuss.
It helps to prevent having (long) conversations about things that are not relevant to you or your organisation.

### Day three

There were some interesting takeaways from the Keynote(s). 
First of all, the [reference architectures](https://architecture.cncf.io/architectures/?all=true) that the CNCF TAB is publishing. 
These seem to be great resources to use as blueprints or starting points for your own architecture.
They are community-provided and cover a wide range of use cases. 

The second takeaway from the keynote was the LFX Insights tool that the CNCF is using to analyse the health of projects in the ecosystem.
You can use this tool to get a quick insight into the health of a CNCF project based on some key metrics, such as contributor diversity (are there many different contributors or is it just one company contributing), contributor retention, merge lead time, security assessments, etc. 
It will definitely be an additional tool I'll use to assess the maturity of the projects, next to their maturity level and available documentation. 

There were again two talks discussing platforms, how they help accelerate development and their importance to accelerating a developer's activities. 
The first talk was by DigitalOcean.
They shared how they created a platform where teams can request development environments in minutes. 
They showed how they moved from a (slow) ticket-based process to a self-service platform using CNCF tools such as ArgoCD and vCluster.
Having been through multiple similar journeys at different organisations, this talk really resonated with me and highlighted again the importance of automation in accelerating development and providing a good developer experience.

The second talk was by Trivago, the hotel search engine company.
They showed how they use preview environments to allow their developers to test their changes in a production-like environment before merging their code.
It reminded me of the preview environments that we used at Agilians (longer ago than I would like to admit). 
Only replacing the updated service in the environment and allowing the environments to be tested by the developers and the QA teams within minutes of creating a pull request is, again, a great way to accelerate development and provide a good developer experience.

The final interesting talk of the day was about SBOMS. 
The talk discussed research done by Thales and CNAM on the quality of SBOMs and the way vulnerabilities are found using them. 
I was surprised to learn that there is a lot of variability in the quality of SBOMs, both in the detection of used packages and the lack of interoperability between different tools, all standardising on the same SBOM format (specifically the SPDX format).
If you're interested in security and want to learn more about the truths and myths of SBOMs, watch the VOD of this talk when it's available.

## Conclusion

As we're expecting by now, KubeCon is always a great conference to attend to stay up to date with the latest developments in the cloud native ecosystem.
This edition was (again) a bit hard to sift through the large amount of talks and find the interesting nuggets, especially with all the talks about AI. 
I tried to focus on talks related to security, platform engineer and developer experience, and they didn't disappoint.
If there's one takeaway that I want to highlight, it's that automation is still key in all aspects of cloud-native development, from building a sovereign cloud platform to providing a good developer experience to ensuring compliance with security requirements.

To me, this will remain a special edition as I had the opportunity to speak at OpenTofuDay and share my knowledge about testing Infrastructure as code with the community.

Finally, I'd like to end with a special shoutout to [ASC Audio Visual](https://acsaudiovisual.com/) for providing AV support at the conference. 
As a hobbyist sound engineer, conferences are always double fun as I get to nerd out not only over the content but also over the AV setup. 
I'm always really impressed by how the audio is clear, not too loud, and there is very little sound bleed between rooms, even when the "walls" between the rooms are not really walls but just curtains.
It probably didn't hurt that they used top-notch equipment such as Shure microphones, EV and even L'Acoustics speakers combined with a nice digital console from Yamaha, Midas and even some Behringer X32s (which I have a soft spot for as I used to own one myself). 
But from experience, I know that having good equipment is only part of the equation; the real magic happens in the setup and tuning of the system, which is something the AV team at the conference clearly nailed.
So if anyone of ASC Audio Visual is reading this: thank you for doing such a great job and making sure that the audio at the conference is top notch!
If someone in the KubeCon organisation reads this: please add a "nerd out over the production side" talk to the conference next year, I would love to attend that!

Oh, and really final though, coffee was great.
This isn't something you can take for granted at conferences. 
Good job, Lavazza and RAI! 