---
layout: post
title:  "Season @ Fedora Modularity"
date:   2019-07-23
excerpt: "Progress report for my Outreachy project at Fedora Modularity"
tag:
- open source
- outreachy
- technology
- gsoc
- research
- python
- jekyll
- fedora
- modularity
thumbnail: https://www.elegantthemes.com/blog/wp-content/uploads/2015/10/Best-WordPress-Timeline-Plugins.png
categories: open-source
---

Hello all! It's **Open Source Summer** coding season at **The Fedora Project**. Translations are pouring heavily at **Modularity**, a strong gust of **Happiness Packets** are incoming in from the East, **Fedora pipelines** are being modified to handle the frequent Fedora releases.

I'm VERY excited to be writing this blog post. I have essentially only a month left for my Outreachy internsip with The Fedora Project to end. Surprisingly, this realization is of a bitter-sweet one. The amount of skills I've picked up during my 2 months of Outreachy are simply priceless. I not only tried to code within deadlines, but also learnt new concepts and technology, implemeted it to reflect into my code, networked with different people as part of fedora community bonding, made MAJOR mistakes, tried to redeem myself from them (still working on that 🙈), and I'm DEFINITELY not done just yet! 

## My Optimististic timelines 😂

Outreachy mentors and interns start the internship with a specific set of project goals. These timelines are ususally a very optimistic view of what could happen if everything goes exactly as planned. IT OFTEN DOESN'T, but people still make optimistic plans. This concept is called as [Planning Fallacy](https://en.wikipedia.org/wiki/Planning_fallacy). Projects always feel easy to work on initially. But delays to projects happen. Maybe your project turned out to be more complicated than you or your mentor anticipated. Maybe you needed to learn some concepts before you could tackle project tasks. Maybe the community documention wasn't up-to-date or was wrong.

Keeping all this in mind, this was my initial timeline:

> **May 20, 2019 - May 25, 2019** - Shall participate via community bonding. Will continuously be in touch with the mentor to break down the internship tasks into steps for the coming days. Will go through the code base thoroughly to look for vulnerabilities, potential bugs, and redundancies.

> **May 26, 2019 - June 10, 2019** - Implement any test that exists only in Python, in C, to ensure that they are getting properly tested by the static analysis and memory-leak checkers (valgrind). This would essentially involve copying already written python tests to C.

> **June 11, 2019 - July 10, 2019** - Write code for a set of new tests which will be provided by the mentor.

> **July 11, 2019 - Aug 11, 2019** - Scan the library for any leftover tests, or vulnerabilities. Write exhaustive unit tests for the leftover code blocks and extend some already written tests (if necessary). Try to make the tests modular by breaking them into smaller tests for maintainability.	

> **Aug. 12, 2019 - Aug. 20, 2019** - Update documentation for all the changes made during the internship.


And then my project changed completely. This time, my mentor made my new timeline: 

> **Phase 1**: Extract all translatable strings from the modules that have been built for each Fedora release and submit them to the translation tool, Zanata, for the translators to work on.

> **Phase 2**: Retrieve the finished translations and use the libmodulemd API to turn them into modulemd-translations documents.

> **Stretch Goal**: Include the code from Phase 2 into Fedora's repo creation automation so that it gets updated automatically every day.

![Translation Lifecycle](../assets/img/lifecycle_translation.png)

I must say this timeline seemed very relaxed and exciting to me. This project felt like I was a consumer AND a developer for libmodulemd both at the same time. 

## Project flow

Now that my project was broken down into tasks, I had to understand each of these problems thoroughly and break them into smaller tasks. That way, I could create a workflow and convert it into modular code. For any task I undertook, I was expected to:

1. Discuss the goal of that task and do theoritical study on the required concepts.
2. Write a requirements document stating:
	1. All the constraints.
	2. Expected input/output.
	3. Dependencies needed(if any).
	4. Activity flowchart described into words
3. Write unit tests first for the task validating our expectation of the workflow. This is called [Test-Driven Development](https://www.guru99.com/test-driven-development.html).
4. Finally write code and fix any intermediary bugs or add necessary features on the way.
5. Ask doubts throughout, if stuck.

Here is an extract of a Software Requirement Specification (SRS) document for my first task. You can check out the whole document [here](https://docs.google.com/document/d/1FANPQE1qwpIjgYGIDGxNH5pOcQ4VMq9O0Xy9Ie0t_ZM/edit?ts=5ce6e6f5).

![SRS Document](../assets/img/SRS.png)

## Task 1
 > Extract all translatable strings from the modules that have been built for each Fedora release and submit them to the translation tool, Zanata, for the translators to work on.

![Phase 1](../assets/img/phase1.png)

### Extraction

 1. Fedora modular metadata is divided into modules which are stored as `YAML` files. 
 2. Every `YAML` file is loaded into a `ModulemdIndex` object. 
 3. Each `ModulemdIndex` object contains `ModulemdModule` objects.
 4. Each `ModulemdModule` object contains `ModulemdModuleStream` objects.
 5. Each `ModulemdModuleStream` (identified by their [NSVCA](https://orionstar25.github.io/outreachy-week-2/)) object contains 3 translatable strings:
	1. Summary
	2. Description
	3. Profile description (There may be zero or more profiles. Each profile may have a description and an arbitrary profile name.)

We stored all the strings of a `module:stream` pair that were only of the **highest** version. It is acceptable to assume that if there are multiple matching `streams` of the same `module`, `stream` and `version` but different `context` and/or `architecture` that you do not need to care about `context` and `architecture`. The module build system ensures that all `context/arches` share the same summary, description and profiles across the `version`.

### Submit to Translation tool

[Zanata](https://fedora.zanata.org/?dswid=-2287) is a translation tool that Fedora relies on right now for its localization needs. It accepts [Babel Catalogs](http://babel.pocoo.org/en/latest/api/messages/catalog.html) (`gettext .pot` files) containing unique strings in one language as keys along with their locations of occurrences in the file. All these strings need to be unique in the catalog. This is important because `gettext .pot` files cannot handle having the same source string appear more than once.
These catalogs are further processed into `.po` (portable object) files and forwarded for translation. Locations are of the type:

```python
module_name;stream_name;string_type
```

 To put this in terms of code, there is a `ModulemdIndex` object as input, and we return a `Babel Catalog` as the output. 

 ```python
 def get_translation_catalog_from_index(index, project_name):
	...
	return catalog
 ```

 Next, to extract the translatable strings (or just strings, for brevity):

 ```python
	# Get all Modulemd.Module object names
    module_names = index.get_module_names()

    # Create a list containing highest version streams of a module
    final_streams = list()

    for module_name in module_names:
        module = index.get_module(module_name)
        stream_names = module.get_stream_names()

        for stream_name in stream_names:
            # The first item returned is guaranteed to be the highest version
            # of that stream in that module.
            stream = module.search_streams(stream_name, 0)[0]
            final_streams.append(stream)

    # A dictionary to store:
    # key: all translatable strings
    # value: their respective locations
    translation_dict = defaultdict(list)

    for stream in final_streams:
        # Process description
        description = stream.get_description("C")
        location = ("{};{};description").format(
            stream.props.module_name, stream.props.stream_name)
        translation_dict[description].append(location)

        # Process summary
        summary = stream.get_summary("C")
        location = ("{};{};summary").format(
            stream.props.module_name, stream.props.stream_name)
        translation_dict[summary].append(location)

        # Process profile descriptions(sometimes NULL)
        profile_names = stream.get_profile_names()
        if(profile_names):
            for pro_name in profile_names:
                profile = stream.get_profile(pro_name)

                profile_desc = profile.get_description("C")
                location = ("{};{};profile;{}").format(
                    stream.props.module_name, stream.props.stream_name, pro_name)
                translation_dict[profile_desc].append(location)

 ```

 Now that we have our strings, we simply put them in our babel catalog with their location of occurrences and return it.

 ```python
    catalog = Catalog(project=project_name)

    for translatable_string, locations in translation_dict.items():
        catalog.add(translatable_string, locations=locations)

    return catalog
 ```

## Task 2
> Retrieve the finished translations and use the libmodulemd API to turn them into modulemd-translations documents.

![Phase 2](../assets/img/phase2.png)

After Zanata translates our strings, they provide us with a `.po` file corresponding to one language. This file is turned back into a `babel catalog` so that we can parse the translations and put them into our original `ModulemdIndex` object. This file contains headers like this:

```
"Project-Id-Version: fedora-modularity-translations VERSION\n"
"POT-Creation-Date: 2018-10-16 18:39+0000\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Last-Translator: Geert Warrink <geert.warrink@onsnet.nu>\n"
"Language-Team: Dutch\n"
"Language: nl\n"
"X-Generator: Zanata 4.6.2\n"
```

Th input is a list of such `catalogs` and our original `ModulemdIndex`. We change this `ModulemdIndex` object inplace and later this change is reflected into the corresponding `YAML` file.

```python
def get_modulemd_translations_from_catalog(catalogs, index):
    ...
    return None
```

Creating `ModulemdTranslation` objects to be later stored in our `index`:

```python
	# Dictionary `data` contains information from catalog like:
	# Key: (module_name, stream_name)
	# Value: TranslationEntry object of a locale
	data = dict()

	for msg in catalog:
		for location, _ in msg.locations:
			(module_name, stream_name, string_type,
				profile_name) = split_location(location)

			try:
				entry = data[(module_name, stream_name)]
			except KeyError:
				entry = Modulemd.TranslationEntry.new(
					str(catalog.locale))

			if string_type == "summary":
				entry.set_summary(msg.string)
			elif string_type == "description":
				entry.set_description(msg.string)
			else:
				entry.set_profile_description(profile_name, msg.string)

			data[(module_name, stream_name)] = entry
```

And now store them in our `index`:

```python
for (module_name, stream_name), mmd_translation in translations.items():
	try:
		ret = index.add_translation(mmd_translation)
	except GLib.Error:
		print(
			"The translation for this %s:%s could not be added",
			module_name,
			stream_name)

```

### PR merged! 2 months, 2 tasks completed, we were right on time till now!

## Task 3
> Include the code from Phase 2 into Fedora's repo creation automation so that it gets updated automatically every day.

![Phase 3](../assets/img/phase3.png)

Till now everything were just functions being called on static files. Now we would pull actual `YAMLs` from fedora's repo creation tool, `Koji`, and then apply the process to obtain translations. I have created a CLI Tool so that individual release translations can be manually added to their respective `modulemd YAML` metadata. But to include these translations into the main release is still a work-in-progress.

As of now, I'm a maintainer for the Modulemd Translation Helpers repository for Fedora. You can check it out [here](https://github.com/fedora-modularity/ModulemdTranslationHelpers#modulemdtranslationhelpers).

## Wrapping up
I still have a long way to go, but my learning curve is only getting better. I will write thorough documentation for all of my work after the completion of my project. I also presented my Outreachy project at the annual Fedora developer conference, [Flock to Fedora](https://flocktofedora.org/) in 2019 at Budapest, Hungary. You can access the slides used for the presentation [here](https://docs.google.com/presentation/d/1-f8p3xIJwZBc73KphAZncSC4Ykz9Zpy4eKr76_qBNts/edit?ts=5d417262#slide=id.g5ee093febd_0_32).

 Hence proved, Outreachy is truly a rewarding experience! I hope you'd like to immerse yourself into this wonderful weather @ Fedora where the sun is about to shine soon 🌦️.


*Fin.*
