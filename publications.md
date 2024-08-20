---
title: Publications
layout: page
---
{% assign publication_files = site.static_files | where: "publication", true %}
{% for mypublication in publication_files %}
  [{{mypublication.basename}}]({{ mypublication.path }})
{% endfor %}
