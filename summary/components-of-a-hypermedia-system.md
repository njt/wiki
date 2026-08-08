---
url: https://hypermedia.systems/components-of-a-hypermedia-system/
title: Components of a Hypermedia System
author: Carson Gross, Adam Stepinski, Deniz Akşimşek
date: 2023
---

A chapter from the book *Hypermedia Systems* that lays out the theoretical
foundation of the web as a hypermedia system. It decomposes a hypermedia system
into four components — hypermedia (HTML), network protocol (HTTP), hypermedia
server, and hypermedia client — then dives deep into Roy Fielding's REST
architectural constraints as formalized in his doctoral dissertation.

The chapter walks through HTTP methods and response codes, the uniform
interface constraint and its four sub-constraints (resource identification,
representation manipulation, self-descriptive messages, HATEOAS), and the
practical consequences for web application architecture. A key contribution is
the HTML-vs-JSON comparison for the same endpoint, demonstrating concretely why
self-describing hypermedia messages eliminate API versioning and client-server
coordination overhead.

The authors argue that most JSON APIs are not RESTful in Fielding's original
sense — and structurally cannot be — because they lack hypermedia controls. The
chapter closes by positioning scripting (Code-On-Demand) as an optional but
native part of REST, provided it augments rather than replaces the hypermedia
model. An appendix on HTML5 semantic elements rounds out the practical advice.
