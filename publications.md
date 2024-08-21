---
title: Publications
layout: page
---
## Book Chapters

<ul>
  {% assign chapter_pdfs = site.static_files | where: "category", "chapters" %}
  {% for pdf in chapter_pdfs %}
    <li>
      <a href="{{ pdf.path }}">{{ pdf.name }}</a> - Last modified: {{ pdf.modified_time }}
    </li>
  {% endfor %}
</ul>

## Conference Proceedings

<ul>
  {% assign proceedings_pdfs = site.static_files | where: "category", "proceedings" %}
  {% for pdf in proceedings_pdfs %}
    <li>
      <a href="{{ pdf.path }}">{{ pdf.name }}</a> - Last modified: {{ pdf.modified_time }}
    </li>
  {% endfor %}
</ul>

### Online

* [2023 - Algorithmic Pattern Salon - Rhythm, Time and Geometry: A geometric approach to the generation and manipulation of rhythmic patterns](https://alpaca.pubpub.org/pub/s96d870n/)
