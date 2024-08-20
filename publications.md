---
title: Publications
layout: page
---
## Proceedings Papers

{% assign publication_files = site.static_files | where: "proceedings", true %}
{% for mypublication in publication_files %}
  [{{mypublication.basename}}]({{ mypublication.path }})
{% endfor %}

### Online

[2023 - Algorithmic Pattern Salon - Rhythm, Time and Geometry: A geometric approach to the generation and manipulation of rhythmic patterns](https://alpaca.pubpub.org/pub/s96d870n/)

## Book Chapters

{% assign publication_files = site.static_files | where: "chapter", true %}
{% for mypublication in publication_files %}
  [{{mypublication.basename}}]({{ mypublication.path }})
{% endfor %}

