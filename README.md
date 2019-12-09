# Microservice Websites

How can we develop websites where the different parts of the pages are developed by different teams? If you work in a large enough organization which has its content and services on the web, this is probably a question you have asked yourself several times.

There is no lack of technologies to choose from when building websites today. Three broad strategies can be identified: client-side rendering, server-side rendering, or both (isomorphic web applications).

To co-exist on the same page, the code for the different parts need to be integrated in some way. Apart from *where* we do rendering (client/server/isomorphic), we also need to decide *when* to integrate (static/dynamic) and *what* to integrate (data/code/content).

![A retail website with parts from different teams](assets/microservice-website.png)

When we build pages with parts from different teams, we're building a *network* of pages and parts. The properties of a network as a whole depends largely on the way we integrate the parts, which in turn depends on how we build the parts. But the way we build the parts depends in turn on the way we integrate the parts, which depends on what properties we want the network as a whole to have. How do we resolve this deadlock?

A rhetorical question: what do you think is most important, the network as a whole (the parts and their integrations) or the sum of all parts (without the integrations)? We think we shouldn't fall into the trap of letting the parts dictate how we integrate the whole. Instead, the whole should set the constraints on how we can build the parts.

So, what are good ways of building a network of microservice websites and components?

The meaning behind the word "good" depends on the current and future needs of the organisation responsible for the software and the users of the software. No architecture is good in-itself, it all depends on the context and the needs.

With this article we want to show that [*server-side rendered websites integrated on content*](https://gustafnk.github.io/microservice-websites/#integrating-on-content) (using [transclusion](https://en.wikipedia.org/wiki/Transclusion)) allow for high *long-term evolvability* compared to client-side rendering integrated with shared code. In other words, if you want a system with high long-term evolvability, you should not develop websites using only client-side JavaScript and integrate them using a shared components approach.

We also want to show that Client Side Includes is a good first choice for transclusion technology, since they are lightweight and allow for a faster initial release than Edge Side Includes (ESI). They also allow for keeping the option open for using ESI later, when beneficial. We describe and compare two related Client Side Include libraries: hinclude and h-include. We also give a suggestion for an approach to work with both global and service local JavaScript and CSS.

Throughout this article, we use an retail site as an example. In the end of this article, we give a brief description of an example architecture for this retail site.

The article begins with a short introdution to microservices. Before moving on to a comparision of integration techniques, we describe a strategy of how the effort of web design can be scaled effectively with *pattern labs*.

---

## Table of Contents

- [Why Microservices?](https://gustafnk.github.io/microservice-websites/#why-microservices)
- [Scaling web design: style guides and pattern labs](https://gustafnk.github.io/microservice-websites/#scaling-web-design)
- [Assumptions and constraints](https://gustafnk.github.io/microservice-websites/#assumptions-and-constraints)
- [Integration techniques](https://gustafnk.github.io/microservice-websites/#integration-techniques)
  - [Integrating on data](https://gustafnk.github.io/microservice-websites/#integrating-on-data)
  - [Integrating on code](https://gustafnk.github.io/microservice-websites/#integrating-on-code)
  - [Integrating on content](https://gustafnk.github.io/microservice-websites/#integrating-on-content)
    - [Edge Side Includes](https://gustafnk.github.io/microservice-websites/#edge-side-includes)
    - [Client Side Includes](https://gustafnk.github.io/microservice-websites/#client-side-includes)
    - [Using ESI and CSI together](https://gustafnk.github.io/microservice-websites/#using-esi-and-csi-together)
- [Client-Side Transclusion with &lt;h-include&gt;](https://gustafnk.github.io/microservice-websites/#client-side-transclusion-with-h-include)
  - [hinclude and &lt;h-include&gt;](https://gustafnk.github.io/microservice-websites/#hinclude-and-h-include)
  - [Local stylesheets and scripts](https://gustafnk.github.io/microservice-websites/#local-stylesheets-and-scripts)
  - [Server driven partial updates](https://gustafnk.github.io/microservice-websites/#server-driven-partial-updates)
- [Example architecture](https://gustafnk.github.io/microservice-websites/#example-architecture)
- [Conclusion](https://gustafnk.github.io/microservice-websites/#conclusion)
- [Appendix](appendix.html)
