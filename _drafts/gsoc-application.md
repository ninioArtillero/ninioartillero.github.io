---
title: "Crafting My Proposal for Google Summer of Code"
layout: post
author: Xavier Góngora
locale: es_MX
categories:
- google summer of code
- open source
- codigo abierto
- programming
- programación
- software development process
- desarrollo de software
- career development
- carrera profesional
- haskell
- education
- educacion
- comunidad digital
- community contributions
---
En este post quiero compartir mi experiencia de aplicar al GSoC dos veces, quedando seleccionado la segunda vez.

Para aplicar efectivamente al [Google Summer of Code 2025](https://summerofcode.withgoogle.com/) es importante conocer su funcionamiento.
Claro, es recomendable leer todos los manuales y documentación disponibles en el sitio oficial.
Pero aquí me enfocaré en ahondar en su caracter comunitario, el perfil de «principiante» de codigo abierto, 
tiempos de preparación y algunos detalles
técnicos como el tamaño del proyecto y la proyección de tiempo.

[Google Summer of Code 2025](https://summerofcode.withgoogle.com/),
as a second attempt to participate in the program. Last year, I started working
on the proposal a bit late. Nevertheless, I managed  to submit a proposal to work
on the idea proposed by Facundo Dominguez (engineer at
[Tweag](https://www.tweag.io/)). The goal was to fix the
[Liquid Haskell](https://ucsd-progsys.github.io/liquidhaskell/) (LH) name
resolution implementation to avoid resolving the names twice. My proposal was not
selected; perhaps it was a bit superficial as I wasn't able to wrap my head
around some of the quirks of the LH codebase in order to propose a clear approach.
Facundo took up the
[issue](https://github.com/ucsd-progsys/liquidhaskell/issues/2169) himself, which
led to a major
[refactoring of the code base](https://www.tweag.io/blog/2025-02-06-refactoring-lh/).
For this year I took the follow up project
[idea](https://github.com/haskell-org/summer-of-haskell/blob/3c9efd21fc0022f7b9c21ac2001ca1049d888dc9/content/ideas/lh-aliases.md),
focusing on type and predicate aliases, two annotation constructs which where left
out of the refactor.

