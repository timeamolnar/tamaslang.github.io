---
layout: post
title: Developer Tools for Spring based Rest applications
---

I have been developing Spring based applications for a while now and I realised that I have a bunch of common classes and solutions I keep reusing across multiple projects.
For these common things I created a library, named <strong>rest-devtools</strong>.
Find it on <a href="https://github.com/tamaslang/rest-devtools" target="_blank">github</a>

It proved to be the most beneficial when I developed small micro-service Spring boot modules where I didn't want to duplicated code and I also wanted some kind of consistency and standardisation regarding error handling, naming, logging, etc.

Without going into details in this blog-entry here is a list of the areas where the rest-devtools library offers solutions:<br/>

* Logging, converting object to it's String representation<br/>
* Logging, define logger for components by interface default method.<br/>
* <a href="http://www.talangsoft.org/2015/04/04/rest-devtools-interfaces/" target="_blank">
Interfaces, Indentifiable, CreatedTrackable, etc.<br/>
</a>
* Constants, ProfileConstants<br/>
* Domain entity parent classes<br/>
* Rest exception handling and resolving it to common ErrorDTO<br/>
* Gateway (external Rest calls) exception handling and resolve it to a common ErrorDTO<br/>

That's for introduction, details will come in separate posts. If you are keen to see what the rest-devtools provides, go to the github repo and clone the project.
In the test folder you'll find a full functioning test application using the library.

T.