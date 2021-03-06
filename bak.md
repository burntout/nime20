# Abstract

This paper addresses its title, and what it means for new interfaces
for musical expression, from two perspectives. First, the writing of
musical patterns in live coding performance, and secondly, in the
weaving and braiding of textiles. By these bringing contemporary and
ancient approaches to pattern together, we find commonalities,
reaching a common definition of pattern in terms of the activity of
making. This goes beyond the usual musical definition of a repeating
sequence, to encompass a wide range of transformations including
symmetry, glide, interference and glitch.

# Introduction

What is a Pattern? Grünbaum and Shephard introduce patterns as
"designs repeating some motif in a more or less systematic manner."
They write in the context of geometric tilings, but the same
definition largely holds in the music fields; a sequence becomes a
pattern, once it is repeated. However, in music we too often focus on
the *repetition*, and not the *systematic manner* in which it is
repeated. For a pattern to be interesting, we need to do more than
repeat it, note-for-note; the repetition only provides the ground for
the pattern to act.

In the following paper, I explore the notion of pattern in-depth. The
main focus will be on the TidalCycles environment, and how it
represents pattern, but will be well grounded by examples in textile
pattern. 

# Pattern in TidalCycles

Work on TidalCycles (commonly *Tidal* for short) first began in 2009,
and over the past decade has developed into a comprehensive, free/open
source environment for exploring pattern, mainly in the context of
live coding music. At heart, it is a domain specific language (DSL)
and environment for patterning Open Sound Control network messages
[@ref], embedded in the pure functional language Haskell. Tidal is
most commonly used in tandem with the SuperDirt software, a hybrid
framework for sample manipulation, synthesis and MIDI, implemented in
SuperCollider. Tidal can be applied to any kind of pattern, and has
indeed been used to pattern live choreographic scores [@sicchio],
woven textiles [@mclean], DMX-controlled lighting, and VJing.

While Tidal has been developed alongside creative practice, it upholds
strong computer scientific principles. Crucially, a pattern is defined
as a pure function, and therefore may be composed with other patterns
flexibly and safely.  As Tidal has developed, its core representation
has grown more succinct, and a recent rewrite resulted in more
rigorous understanding of what, in terms of Tidal, a pattern *is*. The
following paper communicates this result, with examples from the key
parts of Tidal's combinator library.

## Principles

TidalCycles is shaped by a number of design principles, strengthened
through a major refactor leading up to the version 1.0+ release.

### Tersity vs cognitive load

In the design of live coding languages, a familiar tradeoff is between
tersity and cognitive load. The greater the tersity, the fewer
keypresses that are required, but as we approach the shortest
description possible (known as Komologorov complexity), we end up with
something difficult to understand and remember. Tidal aims to balance
this with a healthy library of functions for pattern transformations,
together with a terse and flexible 'mini notation' for sequences.

### Rational time

In computer music, time is commonly represented using floating point
numbers or integers. Floating point time is associated with
inaccuracies due to compounded rounding errors, and integral time
imposes a grid which does not fit well to all musics. On the other
hand, representing time as ratios allows arbitrary metrical
subdivision.

### Combination and composition

Being able to flexibly combine patterns supports rich, creative
exploration of interference patterns. Haskell lends Tidal its strict,
pure type system, supporting a declarative approach where a
coder-musician can simply say which patterns they want to combine, and
largely leave it up to Tidal to work out _how_ to do that.

## Representation

Core to Tidal is its representation of pattern, as a pure function of
time. This folllows work on _pure functional reactive programming_,
where rather than representing data using lists, you represent
behaviour with functions. For example, rather than representing a
sequence as a list of events, Tidal represents it as a function which
takes a timespan as input, and returns all the events which are active
during that time.

Lets have a look at the types themselves:

```haskell
-- | an Arc and some named control values
data Arc = Arc {start :: Rational, stop :: Rational}
```

A timespan is expressed as an _arc_ of time, simply consisting of a
rational start and end time. Timespans are referred to as arcs, in
sympathy with a cyclic notion of time.

```
data Event a = Event { 
    whole :: Maybe Arc, 
    part :: Arc, 
    value :: a
}
```

An event has a value of some type `a`, and both a `whole` and `part`
arc. The `part` gives the 'active' portion of the event's `whole`. The
whole is wrapped in a 'maybe' type, because it is not provided in the
case of _continuous_ events, which do not have a discrete whole, only
a sampled part. This allows both discrete and continuously varying
events to co-exist in the same pattern.

```
-- | A function that represents events taking place over time
type Query a = (Arc -> [Event a])
data Pattern a = Pattern {query :: Query a}
```

A query is the core definition of a pattern. It takes an arc as input,
and returns a set of events as output. The values of the events must
all be of the same type.

If an event's 'whole' extends beyond the query, it is returned as-is,
but its 'part' is curtailed. In other words, you may be returned a
fragment of an event for a given query, but will still know the arc of
the whole, of which the event is part.

## Pure functional transformation

