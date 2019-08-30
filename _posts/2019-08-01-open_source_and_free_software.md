---
layout: post
title: Open Source and Free Software
category: Technology
tags: ['Licenses', 'Open Source']
excerpt: Read about the differences between Open Source and Free Software, some commonly used licenses and factors involved in deciding the license.
---

### Money, Politics And Javascript
{:.no_toc}

I recently watched [The Economics of Open Source](https://www.youtube.com/watch?v=MO8hZlgK5zc), a JSConf EU 2019 talk by C J Silverio. C J Silverio is the former CTO of npm, Inc. and is currently involved with [Entropic](https://github.com/entropic-dev/entropic), a federated package manager for Javascript with Ronald Dahl. It has the potential to be the next **big thing** in Javascript and Web Development in general. I would recommend everything to check the first talk as well.

In her talk, Ceej talks about the history of javascript, node.js, and npm. A brief _(and oversimplified)_ summary is this: 

The early node community had many package managers. One of the managers eventually won and got official status, which continues to today. Joyent bought node but couldn't buy npm since it was still the intellectual property of its creator. Around 2013, node and npm became explosively successful. 

This success was expensive, with costs for managing centralized servers. Npm remains a centralized and privately controlled repository for modules. Private control is terrible because the community has no input in registry policies and features. Javascript community cannot hold npm inc accountable. C J and Ronald Dahl started _entropic_, a federated community-owned package manager as an alternative.

A central theme of the talk was **ownership of the software and the intellectual rigts** - Which inspired me to write this article on idealogies of intellectual rights in FOSS community.

{% include toc.html %}

### Open Source Software

Open source is a term denoting that a product includes permission to use its source code, design documents, or content. Open source development can bring in diverse perspectives beyond those of a single company. In recent years, there has been a significant shift in attitudes towards Open Source, with companies such as IBM, Google and Microsoft having a serious public stake.

### Free Software

Free Software is _the software that grants the user the freedom to share, study and modify it._ A common adage used to describe free software is **Think free speech, not free beer**. The emphasis of free software is on the freedom of the user, rather than the price of the software. Often, the term _libre software_ (liberty) is used instead of free software to make a clear distinction.

[Free Software Foundation](https://www.fsf.org/about/what-is-free-software) additionally outlines the [four pillars of freedoms](https://fsfe.org/about/basics/freesoftware.en.html) which a free software must follow:
- The freedom to run the program for any purpose.
- The freedom to study how the software works and modify it.
- The freedom to redistribute copies.
- The freedom to improve the program and release improvements to the public.

Both Open Source and Libre software can be protective (copyleft) or permissive. 

> Libre and Open Source Software are included together under a umbrella term, [Free And Open-Source Software](https://en.wikipedia.org/wiki/Free_and_open-source_software).

{% include divider %}

### Protective and Permissive Licenses

A **copyleft license** offers the right to modify and distribute copies of work provided the same rights are preserved. The derivates of copyleft software must also be copyleft and free software. Examples - [GNU General Public License](https://www.gnu.org/licenses/gpl-3.0.en.html)

A **permissive license**, on the other hand, offers the right to modify and distribute without any stipulations. Examples - [MIT License](https://opensource.org/licenses/MIT), [Apache License](https://www.apache.org/licenses/LICENSE-2.0)

### Choosing A License
 
 Having laid the groundwork, we can turn to the issue at hand - Choosing a license.

 While researching the article, I realized the issue is not Open Source Vs. Free Software. It is **Copyleft Vs. Permissive license**.

 A point Ceej brought up was _" Who makes the profits?"_.

 Javascript and npm are used and generate billions of dollars in revenue every year. The original contributors are not rewarded financially for their hard work.

 Venture capitalists such as those who funded npm inc usually are rewarded. Additionally, permissive software is used by companies without any obligation to give back to the community.

 > Capitalism loves permissive software because of its exploitable.

 Permissive software is easy to adapt and modify by an organization. There are under no restrictions to release their modified, better version. They can _(and do)_ profit off the derivatives without giving back to the community.

 Commercial organizations are wary of adopting a copyleft software due to its restrictiveness. They cannot profit from the derivatives.

 The ugly truth about FOSS is **there are never enough talented developers and resources**. 

 The growth of a project relies on its volunteers and the community.

 As a developer, I want to share my work and see other projects using it. I want to build and be a part of a community where other developers learn and share their ideas. However, I don't want people and organizations without any contributions to profit off my work.

 > I lean towards copyleft licenses like GPL.

{% include divider %}

### Too Long, Didn't Read

 - If widespread adoption is a priority, use _MIT license_ (or any other permissive license).
 - If freedom for the end-user and having same freedoms on derivatives is important, Use _GPL v3_ (or any other copyleft license).

I would recommend using [choosealicense](choosealicense.com), a website by Github built to guide content creators.

{% include divider %}

### References

 1. [Revolution OS (96 minutes)](https://www.youtube.com/watch?v=4vW62KqKJ5A) - A documentary that explores the story of programmers like Stallman and Torvalds and their journey of the GNU/Linux and the Open Source movement. This movie inspired my love for Linux and open source.
 2. [Economics of Open Source (37 minutes)](https://www.youtube.com/watch?v=MO8hZlgK5zc) - JSConf Talk by C J referenced throughout this article.
 3. [Free Software Vs. Open Source Vs. Freeware](https://dzone.com/articles/free-software-vs-open-source-vs-freeware-whats-the).
 4. [The Cathedral and the Bazaar](https://en.wikipedia.org/wiki/The_Cathedral_and_the_Bazaar) - A 1997  essay suggesting a model of developing OSS.
