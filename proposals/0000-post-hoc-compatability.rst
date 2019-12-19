.. proposal-number:: Leave blank. This will be filled in when the proposal is
                     accepted.

.. highlight:: haskell

Post-hoc Compatibility
======================

Track the actually compatibility of releases on hackage so downstream packages don't need to work around PVP violations.

Motivation
----------

In Cabal 2.0, support for `^>=` was added, which is great:
Just write down the "known-good" versions you've tested, and all the PVP stuff happens implicitly.
This also allows better cooperation between those that care about PVP and solving, and those that prefer to use manually curated package sets.
`^>=` is low-effort for those that don't care, and high-reward for those that do.
Furthermore, in Cabal 3.0, sugar for this sort of "known good" disjunction was added with::

  tested-with: GHC == { 8.6.3, 8.4.4, 8.2.2, 8.0.2, 7.10.3, 7.8.4, 7.6.3, 7.4.2 }

  build-depends: network ^>= { 2.6.3.6, 2.7.0.2, 2.8.0.0, 3.0.1.0 }

This further enshrines this way of writing correct and easily understood bounds as the best choice, most of the time.

One problem remains, however, and that is packages that don't follow the PVP.
Today, that problem is left to downstream packages, which much carefully write complicated bounds to deal with upstream mistakes.
But this duplicates work, and doesn't fix the problem where it actually occurs.
If we were to somehow enforce that all new uploads follow the PVP, that would work for new packages, but not for existing uploads.
Finally, there is package deprecation (e.g. see `https://hackage.haskell.org/package/containers/preferred`_), but this too doesn't address the problem itself.
Individual releases may be perfectly fine, but just not respect the PVP contracts between their assigned versions.
The problem is the compatibility relation itself, not the atoms the relation ranges over.

All these issues with today's workarounds point to a new solution of separately maintaining the compatibility relation.
Just like deprecation, this is something that is done separate from package releases by either the package maintainer or package trustees.
The individual version numbers *won't* change, that would be too confusing, especially to curated package set users who don't care about version solving.
Instead, just the PVP interpretation of the releases will change to match reality.
This information will be in the Hackage tarball for any solver to take advantage of.
Packages then then use `^>= { ... }` in *all* cases, and still be both sound and "almost" complete [#complete].

Finally, there is an extra benefit to this mechanism beyond just dealing with PVP-violations.
Compatibility relations are fundamentally partial orders [#lattice], but PVP can only natively express certain simple partial orders, and yet solving already works with arbitrary predicates.
Just as when package releases make PVP violations, downstream packages clean up the mess, when package releases are *more* compatible than their versions imply, downstream packages have to take the effort to allow the additional solutions.
The same mechanism we make to allow PVP violations can also encode this "extra compatibility", so any solution can take advantage of it.

Proposed Change
---------------

Hackage
~~~~~~~

Here you should describe in precise terms what the proposal seeks to change.
This should cover several things,

* define the scope of the change, what does it affect? cabal? stack? a web site?
* define the intended operations of the new change
* discuss how the change addresses the points raised in the Motivation section
* discuss how the proposed approach might interact with existing features

Note, however, that this section need not describe details of the
implementation of the feature. The proposal is merely supposed to give a
conceptual specification of the new feature and its behavior.

Drawbacks
---------

A bit complex

What are the reasons for *not* adopting the proposed change. These might include
complicating the language grammar, poor interactions with other features,

Alternatives
------------

Here is where you can describe possible variants to the approach described in
the Proposed Change section.

Unresolved Questions
--------------------

Are there any parts of the design that are still unclear? Hopefully this section
will be empty by the time the proposal is brought up for a final decision.

.. [#lattice]
  In fact, they may even be lattices if you are careful with your package type system

.. [#complete]
  There may be complicated solutions involving non-local reasoning that can only be expressed by making the allowed versions of one dependent dependent on the choice of the other.
  But, I postulate that all solutions that allowed by independently constraining dependency versions are still possible with this restriction.
