---
layout: default
---

# Microservice Web UIs

<em class="sub-heading">Building consumer facing websites with multiple teams</em>

By Gustaf Nilsson Kotte ([@gustaf_nk](https://twitter.com/gustaf_nk/)).<br/>
Please give feedback and report issues on the [GitHub repository](https://github.com/gustafnk/microservice-web-uis/).

---

How can we develop web UIs where the different parts of the pages are developed by different teams? If you work in a large enough organization which has its product(s) on the web, this is probably a question you have asked yourself several times.

There are no lack of technologies to choose from when building web sites today. Three broad strategies can be identified: client-side rendering, server-side rendering, or both (isomorphic web applications).

To co-exist on the same page, the code for the different parts need to be integrated in some way. Apart from *where* we do rendering (client/server/isomorphic), we also need to decide *when* to integrate (static/dynamic) and *what* to integrate (data/code/content).

When we build pages with parts from different teams, we’re building a *network* of pages and parts. The properties of a network as a whole depends largely on the way we integrate the parts, which in turn depends on how we build the parts. But the way we build the parts depends in turn on the way we integrate the parts, which depends on what properties we want the network as a whole to have. How do we resolve this deadlock?

A rhetorical question: what do you think is most important, the network as a whole (the parts and their integrations) or the sum of all parts (without the integrations)? we think we shouldn't fall into the trap of letting the parts dictate how we integrate the whole. Instead, the whole should set the constraints on how we can build the parts.

So, what are good ways of building a network of microservice web UIs?

The meaning behind the word “good" depends on the current and future needs of the organisation responsible for the software and the users of the software. No architecture is good in-itself, it all depends on the context and the needs.

With this article we want to show that [*server-side rendered web UIs integrated on content*](#content) (using [transclusion](https://en.wikipedia.org/wiki/Transclusion)) allow for high *long-term evolvability* compared to client-side rendering integrated with shared code. In other words, if you want a system with high long-term evolvability, you should not develop web UIs using only client-side JavaScript and integrate them using a shared components approach.

We also want to show that Client Side Includes is a good first choice for transclusion technology, since they are lightweight and allow for a faster initial release than Edge-Side Includes (ESI). They also allow for keeping the option open for using ESI later, when beneficial. We describe and compare two related Client Side Include libraries: hinclude and h-include. We also give a suggestion for an approach to work with both global and service local JavaScript and CSS.

Throughout this article, we use an retail site as an example. In the end of this article, we give a brief description of an example architecture for this retail site.

The article begins with a short introdution to microservices. Before moving on to a comparision of integration techniques, we describe a strategy of how the effort of web design can be scaled effectively with *pattern labs*.


<a href="javascript:document.querySelector('#table-of-contents').classList.toggle('show'); document.querySelector('.toc-ellipsis').classList.toggle('hide')">[Show/hide Table of Contents]</a>

## Table of Contents

<!-- REMOVE THIS TEXT TO ENABLE MarkdownTOC depth=5 autolink="true" autoanchor="true" bracket="round" -->

- [Why Microservices?](#why-microservices)
- [Scaling web design: style guides and pattern labs](#scaling-web-design-style-guides-and-pattern-labs)
- [Assumptions and constraints](#assumptions-and-constraints)
- [Integration techniques](#integration-techniques)
  - [Integrating on data](#data)
  - [Integrating on code](#code)
  - [Integrating on content](#content)
    - [Edge-Side Includes](#edge-side-includes)
    - [Client Side Includes](#client-side-includes)
    - [Using ESI and CSI together](#using-esi-and-csi-together)
- [Client-Side Transclusion with &lt;h-include&gt;](#client-side-transclusion-with-h-include)
  - [hinclude and &lt;h-include&gt;](#hinclude-and-h-include)
  - [Local stylesheets and scripts](#local-stylesheets-and-scripts)
  - [Server driven partial updates](#server-driven-partial-updates)
- [Example architecture](#example-architecture)
- [Conclusion](#conclusion)
- [Appendix](appendix.html)

<!-- /MarkdownTOC -->

<p class="toc-ellipsis">[...]</p>

---

## Why Microservices? <a name="why-microservices"></a>

A good place to start to learn about microservices is Martin Fowler and James Lewis’ [article](http://martinfowler.com/articles/microservices.html). For me, the key benefits are:

- Less organisational/architectural friction by following [Conway’s Law](https://en.wikipedia.org/wiki/Conway%27s_law)
- Separate deploys for separate services
- Allows for a heterogenous system, which in turn leads to long-term evolvability

The approach of [Self-Contained Systems](http://scs-architecture.org) (SCS) is a specialisation of microservices: a SCS is a “team sized" autonomous web application.

### Example: Retail site <a name="example-retail-site"></a>

I’ll use a retail site as an example in this article. The retail site has two teams – Products and Orders – where the Products team is responsible for showing the products to the users and the Orders team is responsible for the shopping cart and the checkout flow. The number of items in the shopping cart should be visible on each product page and it should be possible to add products to the shopping cart with a single click.

Further, when a product is added to the shopping cart, the entire page shouldn't be reloaded (only a partial reload of the shopping cart should be necessary). And the teams want to avoid unnecessary knowledge of each other’s domain, i.e. the Product team should not need to know that the shopping cart should be updated when adding a product to the shopping cart (only the Orders team should know this).

The future plan is to create a Recommendations teams for product recommendations and a Social team for customer ratings and reviews. As with the shopping cart, these teams will expose pages from their own services, as well as web UI components that are visible on each product page.

In this article, we will think of our services as Self-Contained Systems, since each system will mostly correspond to a team. 

---

## <a name="scaling-web-design"></a>Scaling web design: style guides and pattern labs <a name="scaling-web-design-style-guides-and-pattern-labs"></a>

Web design across multiple teams is always a challenge, regardless of the software architecture. Even if the microservice architecture allows for mostly autonomous teams, they still need to agree on (or comply to) a common web design.

One way to create and communicate around a common web design is to use a *style guide*. A style guide is a toolbox of graphical elements, where some elements in the style guide can consist of other elements from the style guide. 

Using a style guide is not risk free. One risk is that a design team authors a style guide that the teams start to use but over time deviates from. It’s hard for a design team to see all the future design needs of the teams. Another risk is that a policy is created that force the teams to use the style guide, either by creating a “style guide CSS library" or by creating a “QA step" before releases. Regardless, forcing a style guide on the teams creates a bottleneck for delivery, which runs opposite to the distributed development style of the microservice architecture.

A promising improvement on the style guide is the *living style guide* or the *[pattern lab](http://www.bigeng.io/the-living-style-guide-pattern-lab/)*. Here, mutations of the existing style guide elements are fed back into the style guide itself, creating a catalog of existing mutations of some or all of the style guide elements. A subset of the style guide can then be published as a shared resources, directly available for the teams, while still allowing the teams to develop local mutations.

If one service needs to take a dependency on another service with local mutations, style information need to be integrated as well. In the section [Local stylesheets and scripts](#local-stylesheets-and-scripts), we cover how this could be implemented.

### Different design means different resources <a name="different-design-means-different-resources"></a>

In theory, if the same information is displayed in two different places with different styling, the same HTML markup but different CSS rules could be used, given that there is some way of detecting the different contexts. This would probably be the most Don’t-Repeat-Yourself (DRY) approach. However, being too DRY also introduces coupling. For us, it means that we will have trouble changing our markup without introducing regressions – it’s hard to find out in what way our markup is used and styled by our consumers of the service.

A better approach is to use distinctive names for our pattern lab elements and use these names in the stylesheets. This way, we have a one-to-one relationship between the HTML markup for a pattern lab element and the CSS for it, which makes it easier to get feedback on what impact our changes have.

### Responsive/adaptive web design <a name="responsiveadaptive-web-design"></a>

It’s beyond the scope of this article to go into the topic of responsive/adaptive web design (RWD/AWD). In general, RWD/AWD means that we create a flexibility in design and capabilities for different device dimensions, network profiles, etc – the user experience should be good regardless of what device the user has. From a high-level perspective, this means that designers need to think deeply on how the design changes depending on the differences in device capabilities.

In our case, one dimension of RWD/AWD is how the different pattern lab elements adapt to different device capabilities. Another dimension is how the page layout work for different device capabilities (especially screen size).

---

## Assumptions and constraints <a name="assumptions-and-constraints"></a>

Before we continue, we list our assumptions and the constraints that we get today's web browsers.

### Assumptions <a name="assumptions"></a>

These are the assumptions that we are base our reasoning on:

#### Consumer facing website <a name="consumer-facing-websites"></a>

We assume that you are building a consumer facing website. You can still benefit from reading this article if you're building a private web application or similar, but the constraints are quite different.

#### Not only building for desktop web browsers

Quite related to the above assumption is that if you're building a consumer facing website, you think that it's important that users that browse your website with something else than a desktop web browser should have a good experience.

#### Time to interaction is important <a name="time-to-interaction-is-important"></a>

For a consumer facing website, the metric "Time To Interact" is at least as important as the metric "Time to First Meaningful Render". Again, if you're not building a consumer facing website, this might not be true.


#### Long-term evolvability comes from heterogeneity <a name="long-term-evolvability-comes-from-heterogeneity"></a>

In order to evolve a system over time, the system needs to support that parts are built in different technologies, as long as the parts follow an agreed upon protocol. The protocol should be as generic as possible, thus not being based on a particular programming language or framework.

### Client-side constraints <a name="client-side-constraints"></a>

The environment on the server and the client (the different browsers) are not alike. On the client-side, you are constrained by the user’s device, network quality, and runtime. Also, compared to the server-side, there’s a high degree of *diversity* of devices, network quality, and runtime. The web is messy – which is a good argument for using [Progressive Enhancement](https://en.wikipedia.org/wiki/Progressive_enhancement).

#### Some mobile devices have a slow CPU <a name="some-mobile-devices-have-a-slow-cpu"></a>

Historically, the limiting factor has been the network. However, today the limiting factor more and more also tends to be the CPU, since we increasingly use mobile devices when browsing the web. Many websites also rely on a lot of JavaScript being executed before they can show any meaningful content.

Parsing of JavaScript is a CPU bound operation and the CPUs in many mobile devices are not fast. Therefore, you should limit the amount of JavaScript being used on pages capable of being viewed on mobile browsers.

####  Less room for change of framework <a name="less-room-for-change-of-framework"></a>

On the server-side, we can partition our systems basically however we want and use different languages and frameworks for the different parts. There is certainly a cost associated with using multiple languages/frameworks at the same time, but this cost is mostly “cognitive" for the organisation. The big upside with allowing for multiple languages/frameworks is that the system can be migrated from using tech A to using tech B over a quite long period of time, without the user being aware of it. 

On the client-side however, the cost of doing the same move would be much higher, since the end-user needs to download/parse/execute a double amount of code for the length of the migration. This means that the amount of time that the migration is active is a cost factor as well. And we suspect that this could be a reason behind why teams often want to rewrite their client-side web applications instead of migrating them.

Having two frameworks (or ecosystems) on the same page simultaneously is costly, which in turn leads to low evolvability of the system.

#### High rate of change on the client-side <a name="high-rate-of-change-on-the-client-side"></a>

The number of technologies for building client-side web applications has formally exploded during the last ten years, which has led to a high rate of change in how we build these applications. However, when the common idea of how we build good client-side web application changes, the view of our current code bases also change: we increasingly get the feeling that our code base is written in a legacy technology. This can lead to that trend-sensitive developers are leaving the organisation for more modern code bases, or that the pressure for rewriting the code increases.

On the server-side there is much less change, in terms of frameworks and libraries. This is probably due to the fact that HTTP has been used as the delivery mechanism for server-side rendered web since the early days of the web.

#### Isomorphic web applications do not resolve the constraints <a name="isomorphic-web-applications-do-not-resolve-the-constraints"></a>

For the purposes of this article, the same constraints that apply to client-side web applications apply to isomorphic web applications as well. Therefore, we can think of the two strategies as the same thing, namely using a large amount of templating code on the client-side.

As a side note, we think the [drawbacks of isomorphic web applications](https://www.jayway.com/2016/05/23/6-reasons-isomorphic-web-apps-not-silver-bullet-youre-looking/) are too large, compared with the benefits. However, *isomorphic parts* could be a good choice sometimes.

---

## Integration techniques <a name="integration-techniques"></a>

Going back to our retail example, we see that we need a way to integrate the product pages with the shopping cart. We need to decide where to integrate (client/server), when to integrate (static/dynamic), and what to integrate (data/code/content). All in all, this gives twelve different combinations. I’ll go through the most important ones, organised on “what" to integrate.


### Integrating on data <a name="data"></a>

Integrating on data in our example means that the Orders team will expose an API endpoint containing the shopping cart information for a logged in user. The Products team can then build their own component that use that API to display a shopping cart.

This approach means that we will have as many shopping cart components as we have consumers of the shopping cart API (the Products team is one consumer here). This is both good and bad: we don’t have a lot of coupling, but there is a lot of duplicated effort as well. Integrating on data puts the cost on the consumers, which doesn't scale well if the consumers are all members of the same organisation.

Also, if we’re not using [hypermedia controls](http://amundsen.com/hypermedia/hfactor/), we will have a duplication of business logic in the components. Say that we have a rule that says that a user should not be able to proceed to checkout if the user has zero products in their basket. In this case, it means that the rule will be implemented by all shopping cart API consumers.

### Integrating on code <a name="code"></a>

In our retail example, integrating on code means that the Orders team will develop and publish code that the Product team will take a dependency on. The teams need to agree on the mechanisms necessary to render the component. Now, if the Product team need to integrate with a Recommendations team, they too need to agree on the mechanisms necessary to render the component.

On the server-side, one could agree to use the JVM and to only use JVM based languages. Or, maybe to use some other form of interoperability technique between two separate run-times. 

But on the client-side web, the situation is much worse, since the client needs to download/read from cache, parse, and execute all the libraries necessary to render all components on the page. For a desktop computer on a fiber connection, this might not be too bad, but on the mobile web, with a limited CPU and network, this is very bad for performance.

This means that the system as a whole can no longer be heterogeneous – not without a large performance penalty – which hurts long-term evolvability.

Another drawback of integrating on code is release management. In our example above, if all three teams have two versions of their software each, NEW and OLD, we get eight possible combinations of possible release artifacts. The solution to this problem is to introduce [release trains](http://www.scaledagileframework.com/agile-release-train/) – you have to be in time to in order for your software to be a part of the Product team release.

For the site as a whole, you can either have separate release trains for the separate teams, which will cause inconsistencies on for example how the shopping cart works. Or you can have a big coordinated release train for the whole site, which reduces the benefits of microservice quite a bit.

### Integrating on content <a name="content"></a>

Integrating on content in our example means means that the Orders team will expose an API endpoint containing the HTML representation for the shopping cart information for a logged in user. This is similar to when we integrate on data, except that nothing more needs to be done to render the information, other than the web browser software itself.

Another name for including content from another resource is to *[transclude](https://en.wikipedia.org/wiki/Transclusion)* the content from another service (inlining a document in the current document), like '<img>’ elements in HTML pages.

There are several ways to transclude content. One could for example run imperative code to perform HTTP requests on either the server or client and include the responses at the proper places. However, we think that a declarative approach is better, since it mimics the design of other transcluded content in HTML, like images.

Transclusion can be done either on the server or the client. Transclusion on the client is called Client Side Includes (CSI) and transclusion on the server is called Server Side Includes (SSI). However, SSI is also a specific (and old) language for including files or executing cgi-bin scripts in HTML files (or HTTP responses, in general), so the terms are a bit confusing. We’ll try to avoid using the term Server Side Includes for this reason.

*Use root relative URLs in transcluded content*

One additional thing that we need to think about when transcluding content is that our references to external resources (i.e. links and images) needs to be valid after transclusion. Relative URLs (i.e. 'resource/’ or './resource’) are not likely to work, since the location of the transcluded content and the transcluding content are not likely the same. Absolute URLs (i.e. 'https://example.com/resource’) contain the hostname of the resource, which introduce unneccesary coupling between the content and the environment (i.e. development/testing/production).

Using root relative URLs (i.e.’ /path/to/resource’) to external resources is a better way than relative or absolute URLs, since it only relies on a base path in order to resolve the URL. And if we use the default base path (i.e. the current hostname), we keep things as simple as possible.

#### Edge-Side Includes <a name="edge-side-includes"></a>

[Edge-Side Include](https://en.wikipedia.org/wiki/Edge_Side_Includes) (ESI) is a technology that provides a declarative way to include content on the server-side, like this:

```
<esi:include src="/shopping-cart" />
```

Edge-Side Includes is today the most popular way of transcluding content on the server-side.

<div class="fact-box">
<dl class="typl8-lining">
  <dt><b>Edge</b></dt>
  <dd>The term <i>edge</i> in Edge-Side Includes should be seen as the last physical node in our control that is passed by a HTTP response <i>before</i> it reaches the user's computer. If we're using a CDN with ESI support, like Akamai or Fastly, we call that layer our edge. Otherwise, if we use the caching HTTP reverse proxy Varnish (which has ESI support) in our infrastructure, that layer can be considered to be our edge. Finally, if we not using either of the above options, we can consider the web server(s) our edge.</dd>
</dl>
</div>

##### Performance <a name="performance"></a>

Since transclusion is made on the server-side, it's not possible to inspect the returned HTML response and draw a conclusion that ESI was used. However, this property of ESI also introduces risk for degraded performance, since we need to rely on the included services’ performance in order to create a complete page. If the transcluded content is cacheable, we can cache the fragments and the performance risk is removed. But for dynamic content, this option is not available.

In our example, if we want to transclude the shopping cart (which contains dynamic content) early in the HTML and it at that time is having problems with performance, ESI will block the rest of the response until the service has responded or the ESI request times out.

Also, one can argue that web UIs integrated with ESI is a violation of [Self-Contained Systems](http://scs-architecture.org/), since the page is no longer autonomous: if the shopping cart request is hanging, the whole page is hanging.

To be fair though, ESI is really performant when it comes to transcluding static HTML files or cacheable content.


##### Headers <a name="headers"></a>

Another challenge with ESI is headers:

“When an ESI template is processed, a separate request will need to be made for each include encountered. Implementations may use the original request's headers (e.g., Cookie, User-Agent, etc.) when doing so. Additionally, response headers from fragments (e.g., Set-Cookie, Server, Cache-Control, Last-Modified) may be ignored, and should not influence the assembled page." – [ESI Language Specification 1.0](https://www.w3.org/TR/esi-lang)

So, when considering different solutions for ESI, we need know if the solution forwards the client’s headers or not. And, if not, is there any way to enable forwarding of headers, by means of configuration? Since the most common (all?) web authentication mechanisms rely on headers with session tokens, it’s crucial that these headers are forwarded to the other services.


##### Development perspective <a name="development-perspective"></a>

From a development perspective, ESI is quite problematic. How can we see complete pages on a developer machine? 

If we use a cache like Varnish [VARNISH] to get ESI, the development environment either needs to be enhanced with a virtual machine (i.e. VirtualBox) or a container (i.e. Docker), to run the cache on the developer machines. This approach increases the upfront infrastructure investment needed.

If we use a CDN provider, we need another ESI implementation on the developer machines. And that ESI implementation need to match the CDN provider’s implementation, at least for the parts of the ESI specification used. Then, we can either choose to include a cache on the developer machines, or to use a library for your platform, like nodesi [https://github.com/Schibsted-Tech-Polska/nodesi] for node.js. 

##### Summary <a name="summary"></a>

Where Edge-Side Includes really shines is the transclusion of static resources like menus and footers, but for integration with dynamic content it introduces performance risks. With ESI you need to think about how headers are forwarded when doing HTTP integrations. The development perspective is a bit problematic, at least initially. If you use ESI in a CDN and still want the site to be visible on a developer machine, you’ll need to find and use only the “lowest common denominator" features between the CDN and the local ESI implementation.

#### Client Side Includes <a name="client-side-includes"></a>

Client Side Includes (CSI) is a bit broader concept than ESI, since ESI is a standard and CSI is a technique. CSI make one or more AJAX requests to server-side resources and includes the document(s) somewhere in the DOM.

In our example, the shopping cart will be included with an AJAX request to the shopping cart resource, which returns HTML for the browser to render.

It’s not a requirement for CSI to be declarative, but we think it’s a good practice. Some declarative CSI techniques/technologies are:

- Traversing the DOM for a 'data-’ attribute on an `<a href>` and include the `href`
- Traversing the DOM for an XML element and include its `src` attribute inside the XML element [hinclude.js](https://github.com/mnot/hinclude)
- Registering a custom element [CE] and include its `src` attribute, may or may not keep the existing element [&lt;h-include&gt;](https://github.com/gustafnk/h-include), [`<include-fragment-element>`](https://github.com/github/include-fragment-element), [`<html-include>`](https://github.com/chris-l/html-include/)

One downside with CSI is that it doesn’t play well with links to JavaScript included dynamically. This means that it’s a bit more cumbersome to integrate a microservice that has dependencies to its own JavaScript. For an example on how this issue can be resolved (and why integrating CSS is not a problem), see the section [Local stylesheets and scripts](#local-stylesheets-and-scripts).

Another downside with CSI is that it’s best if we know the height and width of the transcluded content in advance, in order to avoid a flickering UI. This is very similar to how we need to treat image elements in HTML. However, it’s sometimes cumbersome to know exactly how much space is needed for content with variable length. Over time, it’s better if we transclude that kind of content that come early in the DOM with ESI, while content with variable length that comes later in the DOM can be included with CSI. However, we then also need to evaluate the possible performance risks with this change.   

With CSI, we also need to follow the cross-origin policy for AJAX requests in the browser, or use CORS (if supported by the browser and enabled in the server). If CORS is not enabled, we need a reverse proxy or load balancer in front of the endpoint, so that they share the same host (both in development mode and production).

Even though iFrames should be considered a CSI technology, it's not a good solution for our purposes. For more details, see the [appendix section on iFrames](appendix.html#iframes-dont-scale).

##### Performance <a name="performance-1"></a>

The performance of CSI is quite the opposite of ESI. It allows the transcluding page to load and render without waiting for the transcluded resources to load. Again, very much like an '<img>’ tag. The downside of this, as with images, is that the page can “jump up and down" if we don’t specify fixed dimensions of the transcluded content.

Before HTTP/2, the browser would create one TCP request for each transclusion, but today more and more browsers ([http://caniuse.com/#feat=http2](http://caniuse.com/#feat=http2), [http://caniuse.com/#feat=spdy](http://caniuse.com/#feat=spdy)) and servers are supporting HTTP/2.

And browsers with HTTP/2 are using HTTP/2 for xhr requests as well. So if both the server and the current browser supports HTTP/2, all requests made with h-include will go through the same TCP connection, given that they have the same origin.

##### Headers <a name="headers-1"></a>

Contrary to ESI, you don’t have to think about header forwarding when using CSI, since the resources are transcluded by the browser itself.

##### Development perspective <a name="development-perspective-1"></a>

Again, contrary to ESI, the development perspective of using CSI is very low-friction, since we’re just using the browser to transclude the content.

##### Summary <a name="summary-1"></a>

Client-Side Includes is a lightweight alternative to Edge-Side Includes. It removes the performance problem of ESI, and adds the 'cross-origin constraint’ and the 'fixed dimensions of transcluded resources’ constraint. CSI is faster with HTTP/2, since AJAX requests to the same origin are made on the same TCP connection.

#### Using ESI and CSI together <a name="using-esi-and-csi-together"></a>

ESI and CSI complement each other: ESI can be quite heavyweight, but work really well with including static resources. CSI is lightweight, but for static and/or cacheable content it would be better for performance to use ESI, especially if the content contains references to JavaScript or CSS. However, the two techniques could very well be used together – use ESI for static content and CSI for dynamic content. A nice side-effect of this combination of techniques is that it’s easier to simulate ESI on a local developer machine if the only thing we use ESI for is to include static resources.

One possible strategy could be to use only CSI in the beginning of a project and when the project matures move some of the CSI elements to ESI instead, where appropriate. This way, we defer the investment in the heavier ESI infrastructure until later, allowing us to release earlier and extract value earlier. With regards to CSS, there would be a performance penalty to include a `<link rel="stylesheet">` in a CSI transcluded response (but with HTTP/2 the performance penalty would be limited). Later we could – again, if appropriate – use ESI instead of CSI. However, we can’t use this strategy with JavaScript, due to the way how browsers load JavaScript, all JavaScript files either needs to be available as shared resources or be included with ESI. For more details on this, see the section [Local stylesheets and scripts](#local-stylesheets-and-scripts)..

It's also possible to combine ESI and CSI in related parts of the page. For example, a header menu could be transcluded with ESI and contain a client side included shopping cart. The opposite relationship also works – to include something with CSI that in turn uses ESI to construct a full response.

---

## Client-Side Transclusion with &lt;h-include&gt; <a name="client-side-transclusion-with-h-include"></a>

In this section, we’ll look at how to transclude content on the client-side using the declarative libraries hinclude and &lt;h-include&gt;. We’ll also give some general advice when using transclusion. For transparency, we want to point out that the author of this article is the creator and core contributor of the &lt;h-include&gt; library.

### hinclude and &lt;h-include&gt; <a name="hinclude-and-h-include"></a>

Let’s look at two libraries that provides declarative ways to include content on the client-side: `hinclude` and &lt;h-include&gt;.

#### hinclude <a name="hinclude"></a>

In January 2006, Mark Nottingham sent an email to www-archive list at W3C, claiming authorship for hinclude. Mark uploaded it to GitHub in 2011 and it has had a steady stream of commits since then.

'hinclude’ uses a custom xml element name in a separate XML namespace to declaratively include resources, like this:

```
<hx:include src="/shopping-cart/component">
  <a href="/shopping-cart/component">Shopping cart</a>
</hx:include>
```

When the DOM has loaded, `hinclude` will find all the 'hx:include’ elements in the DOM, fetch the associated resources of each element and replace the innerHTML of the `hx:element`s with the responses. If JavaScript is not enabled in the browser or if the request fail, the fallback content (in this example, the link) is still shown.

The result after transclusion will look something like this:

```
<hx:include src="/shopping-cart/component">
  <div>You have 0 items in your <a href="/shopping-cart">shopping cart</a></div>
</hx:include>
```

If a link is used as the fallback content, search engines and other crawlers will be able to crawl the site without executing JavaScript, so we would consider it a good practice. However, this would mean that the link is shown briefly during initial load. In the section on &lt;h-include&gt; below, we’ll show how to avoid this brief flash of fallback content.

##### Timing <a name="timing"></a>

The timing on *when* to replace the innerHTML is quite important. We can’t control how fast each service will respond, so if we’re transcluding in a lot of different places the UI will initially change a lot, which is not a great user experience. From this perspective, we’d like to wait for all the requests to have been completed before changing the UI. On the other hand, if one service has a performance issue, all the other transclusions are blocked by that request.

In `hinclude`’s synchronous mode, it waits 2.5 seconds (the default value, which can be configured) for all the requests to return before including the finished requests at that time. The remaining requests are included when each return. In `hinclude`’s asynchronous mode, it includes responses as they arrive.

##### Refresh resources <a name="refresh-resources"></a>

Since `hinclude` only replaces the innerHTML and keeps the surrounding 'hinclude’ element, we are able to refresh the included resource when needed. For example, if we show a list of products where each product has a form with a button that adds that product to the shopping cart, we can detect those form submissions and refresh the shopping cart after a product was added to the shopping cart.

##### Conditional transclusion for small screens <a name="conditional-transclusion-for-small-screens"></a>

For small screens, one might want to skip the transclusion of some resources to save bandwidth. If the brower supports [`matchMedia`](https://developer.mozilla.org/en/docs/Web/API/Window/matchMedia), hinclude looks for a `media` attribute on the `hinclude` element and only tries to transclude the resource if the media expression matches.

##### Transitive transclusion not supported <a name="transitive-transclusion-not-supported"></a>

One drawback of `hinclude` is that transcluded responses containing *other* `hinclude` elements are not automatically processed. However, this is solved in &lt;h-include&gt;.

#### <a name="h-include">&lt;h-include&gt;</a> <a name="h-include"></a>

&lt;h-include&gt; is a port of hinclude using the Web Components standard [Custom Elements](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Custom_Elements) for detecting `hinclude` elements. Among other things, Custom Elements provides us with events when a custom element is created or attached to the DOM. This means that we get transitive transclusion “for free" when using &lt;h-include&gt;.

##### Easy to extend <a name="easy-to-extend"></a>

It’s easy to extend &lt;h-include&gt;, since it’s is a custom element and exposes its prototype. The simplest extension is to disable automatic transclusion, so that the `refresh` method needs to be called in order to load the content. This is how we would create such an extension, called `h-include-manual-loading`:

```
var proto = Object.create(HIncludeElement.prototype);
proto.attachedCallback = function() {};
document.registerElement('h-include-manual-loading', {
  prototype: proto
});
```

At first, this seems like a strange thing to disable automatic loading, but there are at least two scenarios where this would be beneficial: wrapping content included with ESI with a `<h-include-manual-loading>` element to make the content updatable, and lazy loading of content depending if the element is about to enter the viewport.

Another extension to &lt;h-include&gt; could be to be trigger anchor scrolling after successful transclusion.

##### Listens to the `@src` attribute <a name="listens-to-the-src-attribute"></a>

&lt;h-include&gt; listens to changes to the `@src` attribute and transcludes the new URL. This can be valuable in a master/detail scenario, where you can think of &lt;h-include&gt; as a lightweight iFrame.

##### Fragment extraction <a name="fragment-extraction"></a>

&lt;h-include&gt; supports *fragment extraction*, which allows the developer to specify a CSS selector to transclude a part of the response. Instead of having two separate resources (i.e. one with header and footer, and one without), one can expose a single resource and let the consumers use fragment extraction. This would work best for transcluding larger resources (like articles) and not smaller components (like a shopping cart).

There are a few more features in &lt;h-include&gt; that you can read about its [GitHub page](https://github.com/gustafnk/h-include).

##### How to avoid a brief flash of fallback content <a name="how-to-avoid-a-brief-flash-of-fallback-content"></a>

Even if our web servers usually responds fast, we’d like to avoid to show a brief flash of fallback content for our &lt;h-include&gt;s. Here’s an example of how to do it:

```
<!-- Put this code before the first h-include or in the <head> element -->
<script>
  <!-- https://gist.github.com/egeorjon/6755681 -->
  document.documentElement.className =
    document.documentElement.className.replace( /(?:^|\s)no-script(?!\S)/g , '' );
</script>

<style>
  h-include:not(.included) {
    visibility: hidden;
  }
  .no-script h-include, h-include.included {
    visibility: visible;
  }
</style>

```

The first line of code is to detect if JavaScript is enabled in the browser at all, otherwise we’ll always show the fallback content. The first CSS rule then hides all the &lt;h-include&gt;s that are not included (&lt;h-include&gt; adds an `included` class after the AJAX request returns). The second CSS rule shows all included &lt;h-include&gt;s.

##### Drawbacks <a name="drawbacks"></a>

In order to use &lt;h-include&gt;, we need to conditionally load a polyfill for Custom Elements, for those browsers that don’t support Custom Elements. There are a few polyfills to choose from but one property that unites them all is that they drop support for Internet Explorer around version 9 or 10.

### <a name="local-stylesheets-and-scripts"></a>Local stylesheets and scripts <a name="local-stylesheets-and-scripts"></a>

When integrating microservice web UIs, we need to think more carefully on how to transclude content that depends on specific stylesheets and/or scripts. Regardless of if we use a pattern lab or not (see [Scaling web design: style guides and pattern labs](#scaling-web-design)), we probably end up in a scenario where we have some JavaScript and CSS exposed in shared resources (globally available), while some resources are only used within a microservice (locally available). How to transclude content that has a dependency locally available JavaScript and/or CSS then needs to be decided.

#### Local stylesheets <a name="local-stylesheets"></a>

If we optimize for browsers supporting HTTP/2 (and SPDY) it’s not necessary to load stylesheets in the head ([https://jakearchibald.com/2016/link-in-body/](https://jakearchibald.com/2016/link-in-body/)). Early in the product lifecycle, we can include references to stylesheets in the transcluded responses, like this:

```
<h-include src="/shopping-cart/component">
  <link rel="stylesheet" href="/shopping-cart/component/style.css"> <!-- after transclusion -->
  <!-- shopping cart content here -->
</h-include>
```

This approach would cause two HTTP requests in series, since the browser wouldn't see the stylesheet reference until the CSI request has completed. This could be good enough to start with, but we would want to optimize performance later. This optimization can be done in two ways.

If we want to avoid having series of HTTP request we can use ESI instead, which would look like this:

```
<esi:include src="/shopping-cart/component">
```

...which after inclusion becomes:

```
<link rel="stylesheet" href="/shopping-cart/component/style-[hash].css">
<!-- shopping cart content here -->
```

Note that going from CSI to ESI, we now have introduced a performance risk for the transcluding page. Sometimes the benefit is worth the risk and sometimes not. However, the shopping cart's internal architecture matters here as well, so maybe the performance risk is actually quite low.

#### Local scripts <a name="local-scripts"></a>

When importing scripts for transcluded content, we are more constrained than when importing CSS, due to how the browsers load JavaScript.

If we don’t want to use ESI at the initial phase of the product development, we need to do a bit of thinking (if we have a limited amount of scripts, one approach could of course be to let all scripts be exposed as shared resources). One approach could be to use [HTML Imports](https://www.html5rocks.com/en/tutorials/webcomponents/imports/) to include the scripts, like this:

```
<h-include src="/shopping-cart/component">
  <!-- shopping cart content here -->
</h-include>
<link rel="import" href="/shopping-cart/component/scripts">
```

One downside with this approach is that the polyfills for HTML Imports use `eval` to run referenced JavaScript files, which is a violation of the [Content Security Policy](https://en.wikipedia.org/wiki/Content_Security_Policy) Level 2 (CSP). Using the CSP removes many common web security threats, so it’s a good practice to enable it, which in turn means that we today should think twice before use HTML Imports.

Instead, we can use a script loader, i.e. [little-loader](https://github.com/walmartlabs/little-loader), to load cache busted script files:

```
<!-- global resource in head -->
<script src="/shared/vendor/little-loader.js"></script>

<!-- shopping cart component -->
<h-include src="/shopping-cart/component">
  <!-- shopping cart content here -->
</h-include>
<-- more content here -->
<!-- use the good practice of loading scripts at the bottom of the page -->
<script src="/shopping-cart/component/scripts.js"></script>
</body></html>
```

Where `/shopping-cart/component/script.js` would look like this:

```
window._lload('/shopping-cart/component/the-script-[hash].js";
```

This way, we can release new versions of local scripts without forcing consumers to update their code. However, this approach means that we make HTTP requests in series for loading local scripts, since we need to download `/shopping-cart/component/scripts.js` in order to download the actual scripts the component need.

#### Local scripts with ESI enabled <a name="local-scripts-with-esi-enabled"></a>

With ESI enabled, we can use the same approach as for loading CSS, i.e. inlining the script reference in the transcluded content, like this:

```
<esi:include src="/shopping-cart/component">
```

...which after inclusion becomes:

```
<link rel="stylesheet" href="/shopping-cart/component/style-[hash].css">
<script src="/shopping-cart/component/script-[hash].js"></script>
<!-- shopping cart content here -->
```

Note that the page rendering now blocks and this point in the code until the script downloaded and executed. If you only include one script reference in the transcluded content, you can

- use the `async` attribute to download and execute the script in an asynchronous way, if the script has no dependencies
- use the `defer` attribute to download the script in an asynchronous way, but execute in order just before `DOMContentLoaded`

The `async` and `defer` attributes are relatively well supported by the browsers. For more details, see [Deep dive into the murky waters of script loading](http://www.html5rocks.com/en/tutorials/speed/script-loading/).

#### Summary <a name="summary-2"></a>

In the beginning of the product development, include stylesheets references in the CSI responses. The services should expose a JavaScript file that in turns calls a script loader. The consumers should reference that JavaScript file at the bottom of each page.

When ESI is part of the infrastructure, remove the references to the JavaScript script loader and inline the scripts with the transcluded content. Note that this operation crosses two service boundaries, so it either needs to be coordinated or the scripts need to be able to detect if they have already been loaded (and then do nothing).

### Server driven partial updates <a name="server-driven-partial-updates"></a>

Going back to our retail example, we now know how to expose a shopping cart component service, possibly with CSS and JavaScript references. We also know how to use this shopping cart component service from the consumer side. But what we haven’t covered yet is how to do a partial update of the shopping cart when the user adds products to it.

First, let’s make sure users without JavaScript get an updated shopping cart when they add products to it. Each product item in the list of products contains a form with a button:

```
<form method="POST" action="/shopping-cart/add" class="hijack">
  <input type="hidden" name="product-id" value="123"> 
  <button type="submit">Add to shopping cart</button>
</form>
```

In the `shopping-cart/add` resource, we then redirect to the value of the `Referer` header, or the shopping cart itself if the header value is not set. In node/express, this could look like this:

```
res.redirect(req.header('Referer') || '/shopping-cart');
```

For users with JavaScript enabled, we want to “hijack" the form submit and replace the default browser behavior with custom behavior, namely to refresh the shopping cart on a successful AJAX form submission. We do this with jQuery:

```
$(document).on('submit', 'form.hijack', function(event) {
    event.preventDefault();

    var $form = $(event.target);

    $.ajax({
      url: $form.attr('action'),
      type: $form.attr('method'),
      data: $form.serialize(),
      success: function(response) {
         var shoppingCart = $('shopping-cart')[0];
         shoppingCart && shoppingCart.refresh(); // Refresh the h-include element
      },
    });
  });
```

For small sites, this can be enough. But for larger sites, this type of code will introduce coupling, since we now have JavaScript code that introduces coupling between the 'add to shopping cart’ forms and the shopping cart itself. How can we remove this coupling?

If the server detects that AJAX was used to submit the form, it can return something else than a redirect response. Instead, it can return a list of *events* that some client-side infrastructure should process. In node/express, like this:

```
if (req.xhr) { // req.xhr is true if the header 'X-Requested-With' is 'XMLHttpRequest'
  res.send({ events: ['shopping-cart-item-added'] });
} else {
  res.redirect(req.header('Referer') || '/');
}
```

(Note that if we wanted to, we could submit the form to another resource instead of relying on a header. Then, we would have needed some form of URL convention to differ between the scenarios of enabled/disabled JavaScript.)

The transcluded content needs to be decorated with a `data-` attribute, to indicate that it should be refreshed on one or several events:

```
<div data-refresh-on="shopping-cart-item-added">
  <!-- shopping cart here -->
</div>
```

Finally, we need a piece of client-side infrastructure code to glue this together. The following example could be the seed of such code:

```
var IncludesRefresher = {
  refresh: function(eventNames){

    function check(element, eventName){
      var parent = element.parentElement;

      var attribute = element.getAttribute('data-refresh-on');

      if (attribute) {
        var tokens = attribute.split(',');

        return tokens.some(function(token){
          var topic = token.trim();
          return topic === eventName;
        });
      }
      
    }

    var subscribers = document.querySelectorAll('[data-refresh-on]');

    for(var i = 0; i < subscribers.length; ++i) {
      eventNames.forEach(function(eventName){
        if (check(subscribers[i], eventName)) {
          var elementToRefresh = subscribers[i].parentElement
          elementToRefresh.refresh && elementToRefresh.refresh();
        }
      });
    };
  }
}
```

In short, for each element with the attribute `data-refresh-on` and each event returned in the server response, we check the element's 'data-refresh-on' value against the event and refreshes the element if it matches.

We then need to use the `IncludesRefresher` when we "hijack" the form:

```
$(document).on('submit', 'form.hijack', function(event) {
    // ...

    $.ajax({
      // ...
      success: function(response) {
         IncludesRefresher.refresh(response.events);
      },
    });
  });
```

Since the only code that has knowledge about when the shopping cart needs to be updated relies in the shopping cart service itself, we have removed the coupling between the Products team and the Order team.

When ESI is part of the infrastructure, we can choose to include the shopping cart with ESI instead. If we want to keep our ability to to partial updates to the shopping cart, we only need to wrap the transcluded content in an `h-include-manual-loading`, which disabled automatic client transclusion but keeps the ability to update the content (see the section on [h-include](#h-include)).

---

## Example architecture <a name="example-architecture"></a>

We use &lt;h-include&gt; to keep the initial infrastructure lightweight. For stylesheets and JavaScript local to each component, we use the approach in [Local stylesheets and scripts](#local-stylesheets-and-scripts): return stylesheet references in the transcluded the content and reference a JavaScript that in turn uses a script loader. To partially update the shopping cart component, we use the approach in [Server driven partial updates](#server-driven-partial-updates).

### Optimizations <a name="optimizations"></a>

We replace the &lt;h-include&gt; elements with ESI when appropriate, in order to increase performance and decrease the number of web requests. To reference component local JavaScript, remove the script loaders and inline the script elements with the transcluded content, as described in [Local stylesheets and scripts](#local-stylesheets-and-scripts).

If we want to include the shopping cart with ESI and still partially update the shopping cart when the user adds a product, we need to wrap the transcluded content in an `h-include-manual-loading`.

---

## Conclusion <a name="conclusion"></a>

For a big enough problem in a big enough organisation, in order to not get friction by violating [Conway's Law](https://en.wikipedia.org/wiki/Conway%27s_law), you need to have at least one system per team. If you develop [consumer facing websites](TBD) and value both ["Time To Interaction" performance] and [long-term evolvability from heterogeneity], you should use server-side rendering. [Integrating on data] is safe, but don't allow for economies of scale (the effort is duplicated by each team). [Integrating on code] creates the need for release trains, which is costly and hurts agility. Integrating on content (HTML) allow the team to not duplicate their efforts, while decoupling their release cycles.

Integrating on content is also called [transclusion] and we like the declarative approaches of translusion the best. One server-side transclusion technique is [Edge-Side Includes](#edge-side-includes) (ESI), which is supported by a few CDNs and some caching HTTP reverse proxies. ESI is infrastructure heavy, so it might hurt your initial project delivery speed. It can introduce performance risks, if one of the servers are having performance problems. ESI is something that we can easily refactor to later, if wanted. Declarative [Client-Side Includes](#client-side-includes) (CSI) allows for better initial project delivery speed, and together with HTTP/2 the performance is not too bad for many scenarios. However, CSI introduces some challenges with loading service local JavaScript, but that challenge is solvable with [script loaders](#local-scripts).

Two related CSI libraries are [hinclude and h-include](#hinclude-and-h-include). The latter is a web components port of the former, with a few added features. In summary, h-include supports transitive includes, fragment extraction, extension points and possible lazy loading (on scroll), but the drawback is that it only supports IE9/IE10 and up.

In order to scale web design (HTML/CSS), we recommend an iterative [Pattern Lab](http://www.bigeng.io/the-living-style-guide-pattern-lab/) approach, where learnings and mutations are fed back to the pattern lab.

---

## Acknowledgments

Thanks to Anton Fagerberg, Per Ökvist, Albert Bertilsson, Johan Haleby, Oskar Wickström, and Mikael Freidlitz for valuable feedback during the private preview.

Thank you Sara for your patience and support ❤️

<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-84868846-1', 'auto');
  ga('send', 'pageview');

</script>