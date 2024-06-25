---
layout: post
title:  "ReSpec- My Interpretation"
date:   2019-02-19
description: "My project repo for GSOC'18 selections"
tag:
- W3C 
- ReSpec
- GSOC
- open soruce
- jekyll
thumbnai: https://utilu.com/UtiluMFC/img/mozilla_firefox_logo.png
categories: open-source
giscus_comments: true
---


This is going to serve as a handy guide for me in the coming days. I'm going to log into this my understanding of what ReSpec is, how it works, code base structure, workflow, developments needed to be made, etc. This is going to make my understanding of it more solid as I shall try to be as clear and accurate as possible and hence (kind of ) learn by teaching myself. Also, all this so that I bag GSOC with ReSpec. Right now it looks kinda tough, but lets hope for the best!

Getting started :-

1. ReSpec is a tool (JS library) that makes writing specifications easier. ReSpec handles things like styling, referential integrity, bibliographical data, and other mundane tasks.

2. Uses Babel ( a JS compiler to compile the code in src directory).

3. `respecConfig` object used to specify configuration in ReSpec. Can  configure options by changing its element values.

4. Specifications typically require having a "short name", which is the name used (amongst other places) in the canonical "[http://w3.org/TR/short-name/](http://w3.org/TR/short-name/)" URLs. This is specified using the shortName option.

5.  Most (but not all) W3C documents are produced by groups of some sort: Working Groups (WG), Interest Groups (IG), Incubator Groups (XG, now defunct), Coordination Groups, the TAG, and Community or Business Groups (CG, BG). For simplicity, we will be referring to all of the above as "Working Groups".

6. The result of changing these configuration options can be seen in the "Status of this Document" section. Here is an example with different values. **Incubator Groups require some additional information in addition to the above, but since this group type no longer exists they are deprecated: xgrGroupShortName, xgrDocShortName. Specification Status**

7. Pointing to previous editor version of specs (drafts) is possible (version control). ** You can look at an example. ( but provided no example, in many cases).**

8. **Migrate a lot of documentation to the specified file.**

9. Inclusion of external content in ReSpec is done using the data-include attribute. It is expected to point to a resource, using its relative path from the including document. The content will get included as a child of the element on which the inclusion is performed (unless data-include-replace is used, in which case it replaces the element), replacing its existing content.

10. By default the inclusion is asynchronous, which means that the included content may or may not be further processed by ReSpec after it gets added to the DOM (the result won't be deterministic and so is only useful if the content is not meant to be further processed). If you wish to ensure that ReSpec processing is applied to the content, use data-include-sync. Be warned that there is a steep performance penalty in doing so, since all ReSpec processing will effectively pause until the content has been fully fetched and inserted. **Can change this somehow for performance issues?**

11. HTML supports functionality to mark abbreviations and acronyms (using &lt;abbr&gt;), using the title attribute to provide the expanded version. This is something that's nice to do once, but tedious to repeat every time that a given term is used. What ReSpec does is that it allows you to do it just once, and it will detect all other uses of the same in the text and will automatically mark them up in the same way.

12. The href-less a element is not limited to linking to definitions but also knows how to link to other items such as WebIDL interfaces.

13. Best Practice Documents Best practices may be shown, numbered, and formatted using a div with class practice containing a p &gt; span.practicelab with the practice's title and a p.practicedesc with description of the practice. **This feature is rarely used, and likely needs to be updated. If you wish to use it in anger, please contact me and we can improve support for it.**
