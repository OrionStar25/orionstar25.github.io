---
layout: post
title:  "Language is a bridge, not a barrier"
date:   2019-06-05
excerpt: "My Outreachy project, and a rough timeline of how I'm going to implement it"
tag:
- open source
- outreachy
- technology
- gsoc
- research
- python
- jekyll
- modularity
- fedora
thumbnail: https://docs.pagure.org/modularity/assets/logo.png
categories: open-source
giscus_comments: true
---

This post explains my Outreachy project with Fedora Modularity in greater depth.

I'm going to start with the basics. This flow of information in this blog post, also happens to be the order in which I learnt about Fedora and ultimately my project.

### What is Fedora?

[Fedora](https://getfedora.org/) is a Linux distribution developed by the community-supported [Fedora Project](https://start.fedoraproject.org/) and sponsored by [Red Hat](https://www.redhat.com/en). Currently, three different editions of Fedora are currently available:
1. **Workstation**, focused on the personal computer,
2. **Server** for servers,
3. **Atomic** focused on cloud computing.

> As of February 2016, Fedora has an estimated 1.2 million users, including Linus Torvalds, creator of the Linux kernel.


### What is Fedora Modularity?

The Fedora Project has a set of technologies called Fedora Modularity whose purpose is to allow you to be able to "swap out" portions of the available package set in the Fedora Linux Distribution. Modularity introduces a new optional repository to Fedora called Modular that ships additional versions of software on independent life cycles. **This enables users to keep their operating system up-to-date while having the right version of an application for their use case, even when the default version in the distribution changes.**

![modularity](https://user-images.githubusercontent.com/28835849/58980334-b4b0df00-87ed-11e9-99dd-58729bd733be.png)

#### Too fast vs. too slow

Modularity solves the *"Too Fast / Too Slow Problem"*. Different users have different needs. Everyone wants their OS to be stable and change nothing except for 5 things that need to always be the latest version: 
1. Module (N)
2. Stream (S) 
3. Version (V)
4. Context (C)
5. Architechure (A)

This is the metadata of a module. An example module would be:
```
name: nodejs
stream: 8
version: 20180816123422
context: 6c81f848
arch: x86_64
```
This NSVCA is never the same between different users. The module metadata is written and stored as a `YAML` document, which `libmodulemd`, a helper library for Modularity, exists to read, parse, modify and output.

|Too fast| Too slow|
|:-------|-------:|
System administrators often want stability for long periods of time.|Developers often want the latest version of softwares.|
|----
Fedora generally ships the latest stable versions of its component packages when it is released **twice per year**. That is convenient for desktop users and developers.| CentOS targets long-term stability and releases a new version **once every few years**. This is convenient for server administrators as there are fewer changes over longer periods of time.|
|=====
Some Fedora upstreams release their software faster than twice a year. This can be an issue for Fedora servers because it is sometimes necessary to have a stable version of certain packages for a longer period, mostly because of third-party applications.|CentOS's issue is that some of the software gets too old for modern applications, and newer versions might be needed. |

{: rules="groups"}

In other words, it would be convenient to be able to choose some parts of the system to update infrequently, while other parts update at a faster pace.


### What are translation tools?

Another of Fedora'a project called the [**Fedora Localization Project (FLP)**](https://fedoraproject.org/wiki/L10N) has the goal to bring everything around Fedora (the Software, Documentation, Websites, and culture) closer to local communities (countries, languages and in general cultural groups). 

> Also called the **L10N** (an abbreviation of the term "Localization". replace the middle letters of a word (in case 'ocalizatio') by naming the number of letters between the first and last letter of that word (in case 10)

Fedora uses a translation tools like Zanata where we can push the English strings and  translators from around the world can produce translated versions of them.


### What is my project?

Initially, my Outreachy project was titled: `Extend unit tests for libmodulemd`. 
Well, that seems pretty straightforward. But then, 

*On IRC one day, having a chat with my mentor*

> sgallagh: So I was wondering how you would feel if I offered you a different, slightly harder (but more well-defined) task.

**BAMM!**

So here's what I'm doing now! :O

1. On Fedora, we have `dnf` for package management. `dnf` has many subcommands, including `dnf module`, `dnf module list`, `dnf module install module:stream`, etc.
In particular is `dnf module info`, which gives detailed information and description about each module stream.

2. `libmodulemd` has the ability to manage translations for these description fields, so that DNF (and other tools like it) can present the descriptions in the user's preferred languages (English, Swahili, Mandarin, etc.)

3. We do that by including another YAML document type in the module metadata (`document: modulemd-translations`). This is essentially a lookup table of strings with their translations in various languages.

4. What we don't have right now is this: 

> **An automatic way to extract these strings(summaries, descriptions) and push them to Zanata, and a way to retrieve the translations from Zanata, convert them to modulemd-translation documents and include them in the module metadata in the package repository that DNF reads.**

5. There's two parts to the task:
	1. **Phase 1**: Extract all translatable strings from the modules that have been built for each Fedora release and submit them to the translation tool, Zanata, for the translators to work on.

	2. **Phase 2**: Retrieve the finished translations and use the libmodulemd API to turn them into modulemd-translations documents.

6. **Stretch Goal**: Include the code from Phase 2 into Fedora's repo creation automation so that it gets updated automatically every day.


I'm creating a Python library called `ModulemdTranslationHelpers` that will achieve the task mentioned above. I've been greatly enthused by the whole concept of modularity and this project is super interesting!
