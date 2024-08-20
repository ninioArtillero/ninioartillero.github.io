---
title: Publications
layout: page
---
{% assign publication_files = site.static_files | where: "publication", true %}
{% for mypublication in publication_files %}
  [{{mypublication.basename}}]({{ mypublication.path }})
{% endfor %}

[2023 - Algorithmic Pattern Salon - Rhythm, Time and Geometry: A geometric approach to the generation and manipulation of rhythmic patterns](https://alpaca.pubpub.org/pub/s96d870n/)
