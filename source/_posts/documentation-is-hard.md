---
title: Documentation is Hard
date: 2017-07-18 14:19:13
tags:
- documentation
---

Producing effective documentation can be very difficult. The labor attendant with maintaining it tends to grow exponentially with the complexity and maturity of your project. Once tiny guides explode over time into sprawling works, while APIs you described in meticulous, comprehensive detail quickly come to lack critical examples. And still, even when you've done the best you can do, users show up in your IRC channel asking basic questions because they didn't read the manual. After a while, you wonder if you can blame them.

I wouldn't know how to program if it weren't for documentation. Poring over readmes and [Sphinx sites](https://docs.python.org/3/index.html) and [annotated source code](http://backbonejs.org/docs/backbone.html), and eventually [writing my own](https://github.com/garbados/prisoners-dilemma-tournament#prisoners-dilemma-tournament) has given me a deep appreciation for the language we use to make our works accessible. There are some things I've come to expect from documentation, so I figured I'd share them:

## Readmes

For projects of a certain size or nature, the first thing your potential users will encounter might be your readme. I can't tell you how many readmes I've encountered that only included the name of the project and a single sentence vaguely referencing the intended functionality. Even if you haven't got users, what happens when you need to put the project down? Do you expect your future self, upon returning to it, to painstakingly explore the source code all the while wondering, "What was I thinking?"

Save yourself the trouble. Include these basic items:

- Project name: How should I call your project?
- Project description: What does it do? Who is it for?
- Project status: Is the project active? Alpha? Beta? Ready for production? Badges (ex: Travis, Coveralls, Greenkeeper, etc.) can help communicate the liveliness of the project. If the project is dead or you have no plans to support it, say so.
- Demo link: Is there a demo? If it's a static site, is it deployed somewhere I can visit?
- Install instructions: How do I install the project? What does it depend on?
- Usage instructions: How do I use the project? If it has an executable, what flags and arguments does it use? If I need to modify source files to make it work for me, what do I need to modify? How? Why?
- Testing instructions: If there's a test suite (There better be!) how do I run it?
- Contributor instructions: How should I file a bug report? A feature request? How do I submit patches or PRs? Include links, even examples if you have complex requirements. If it starts to span more than a paragraph, consider writing a CONTRIBUTING file ([example](https://github.com/beakerbrowser/hashbase/blob/master/CONTRIBUTING.md)) and referencing it from your readme.
- License information: By using your project, what legalese must I oblige? If I need permission to use it, whose permission do I need? How can I contact them?

As your project grows, more and more of these sections will become links to more comprehensive documents. Still, keeping these sections in your readme even only as links establishes canonical locations for relevant docs.

## API docs

If you've got an API, whether a library exposing methods or an HTTP interface, it needs two essentials:

- Practical examples: How do I accomplish the things the project is actually for? Example by example, describe best practices for common tasks.
- Reference docs: List every public method and describe its parameters. Options, headers, understanding possible errors, etc. CouchDB does [a good job](https://couchdb.readthedocs.io/en/2.0.0/api/index.html) on this.

It frustrates me to no end when I find libraries with hidden methods that only the maintainers are aware of. Document everything!

## Audience Considerations

As your project grows, you might find people evaluating it who will not be using it directly. You might have seen a [convince your manager](https://github.com/Netflix/unleash#convince-your-manager-why-use-unleash) section in a readme or two. Often, the manager themself will be reading your documentation, even if they don't know a `for` from a `while`. Think hard about who is using your project for what, and how their needs for information differ. Some folks care about testimonials or how and whether anyone else is using the project. Some folks want guides for use in production, details about performance and scaling considerations, comparisons to similar tools, etc. Some folks wanna know who they can pay for a number to call at 3am in a panic about their failing code. As your project matures, rifts in your userbase will emerge, and their needs will grow more distinct. Think hard about how to accomodate those differences.

## Accessibility Considerations

Not everyone can read, not everyone can read English, and not everyone can read long bodies of text. Video tutorials can help those with dyslexia, and keeping guides short and to the point can help folks who struggle with focus. Segment your documentation into focused sections instead of aglomerating them into 100k+ word manuals. For the folks who want that mega manual to be as mega as possible, many documentation frameworks today provide utilities for building a PDF from your docs which you can distribute to users.

Consider also internationalization as a matter of accessibility. Baking i18n into your documentation build pipeline will save you work as the project matures, even if you only start with one language. Welcome translations but do your best to double-check them like you would any PR.

## Conclusion

Docs matter! Do your best to make them great. Use what you make, and write the docs you want to see in the world. This guide is only a primer for the broader world of documentation best practices, so look around for advice that resonates with you, and think about the documentation that especially helped you.

Good luck! I believe in you 💖
