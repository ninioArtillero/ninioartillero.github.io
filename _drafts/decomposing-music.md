---
title: "Musical dimensions"
layout: post
author: Xavier Góngora
categories:
- programming languages
- music notation
- music theory
---
Inspired by computer music software, we attempt a reckless decomposition of musical structure into the orthogonal dimensions of \emph{sample} and \emph{rhythm}. 
The former alludes to an audio sample file, working as
an abstraction over musical properties such as pitch and timbre. From a
physical perspective, a sample is related to the frequency of air
pressure waves as a function of time. 
On the other hand, rhythm is inextricably bound
to a discrete event stream dividing continuous time flow in
non-overlapping \emph{inter-onset intervals} (IOIs).
In other words, our concept of rhythm is a simplification of the MIDI protocol, were all events have the same \emph{velocity}, no offset and without a notion of
simultaneity. In this way, a rhythmic pattern is a sequence of atomic
events (sound onsets) with all \emph{inner} structure abstracted away.
Of course, this doesn't mean a rhythmic pattern is stripped of all structure.

On a somewhat similar strain, Hudak and Quick \citeyearpar{Hudak2018Haskell}
works on musical structure in two complementary domains. 
One is that of \emph{signals}, which is related
to \emph{sound} synthesis, while the dimension of \emph{notes}
corresponds to musical notation proper. This division could be explained
in terms of This approach fits quite well with computer and electronic
music, where both activities share interfaces. Before this practices is 
plausible that the signal dimension was considered as given 
by western composers, as this was a matter of concern for 
performers and luthiers. 
The rhythm dimension might be found embeded in the
\texttt{Music\ a} polymorphic data type, in relation the \texttt{:+:}
operator for sequential composition.

