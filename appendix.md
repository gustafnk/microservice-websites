---
layout: default
---

# Microservice Web UIs

## Appendix

### iFrames don’t scale <a name="iframes-dont-scale"></a>

Technically, iFrames should also be considered to be a declarative CSI technology, but the downsides of using iFrames are that

- each iFrame creates an extra browser process
- they are hard to style
- they need a “postMessage bus" to communicate with the parent document

For a small number of integrations on a site, iFrames can be a good solution. But for a microservice web UI solution, iFrames don’t scale – the drawbacks per integration are too large.



<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-84868846-1', 'auto');
  ga('send', 'pageview');

</script>