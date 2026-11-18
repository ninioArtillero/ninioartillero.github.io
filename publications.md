---
title: Publications
layout: page
---
## Book Chapters

<ul>
  {% assign chapter_pdfs = site.static_files | where: "category", "chapters" %}
  {% for pdf in chapter_pdfs %}
    <li>
      <a href="{{ pdf.path }}">{{ pdf.name }}</a>
    </li>
  {% endfor %}
</ul>

## Conference Proceedings

<ul>
  {% assign proceedings_pdfs = site.static_files | where: "category", "proceedings" %}
  {% for pdf in proceedings_pdfs %}
    <li>
      <a href="{{ pdf.path }}">{{ pdf.name }}</a>
    </li>
  {% endfor %}
</ul>

## Online

* [2023 - Algorithmic Pattern Salon - Rhythm, Time and Geometry: A geometric approach to the generation and manipulation of rhythmic patterns](https://alpaca.pubpub.org/pub/s96d870n/)
* [2025 - Tweag Engineering Blog - Qualified Imports and Alias Resolution in Liquid Haskell](https://www.tweag.io/blog/2025-09-11-qualified-aliases/)
* [2026 - Tweag Engineering Blog - Writing static checks to an unsuspecting library with Liquid Haskell](https://www.tweag.io/blog/2026-06-11-diff-package-static-checks/)

### Reports

* [2025 - Haskell.org / GSoC Results - Qualified Imports and Alias Resolution in Liquid Haskell](https://blog.haskell.org/gsoc-2025/#qualified-imports-and-alias-resolution-in-liquid-haskell)
