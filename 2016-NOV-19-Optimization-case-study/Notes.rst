Unicode Normalization Optimization
==================================

Git repo: https://github.com/harendra-kumar/unicode-transforms

* http://unicode.org/
* http://unicode.org/faq/normalization.html Normalization FAQ
* http://www.unicode.org/versions/Unicode9.0.0/ch03.pdf Normalization spec
* http://www.unicode.org/reports/tr15/ UNICODE NORMALIZATION FORMS

Unicode 9
---------

+------------+--------------------------------+-------------------------+
| 273 blocks | 16-65536 code points per block | 0x0 - 0x10FFFF (21 bit) |
+------------+--------------------------------+-------------------------+

Normalization
-------------

* Decomposed
* Composed

Utf8proc
--------

The C implementation that was used with Haskell bindings as the original
implementation in this package.

Decomposition
-------------

* Decompose
* reorder using combining class

s1 c1 c2 c3 s2c ...

s1 c1 c2 c3 s2 c4 c5 c6 ...

Character Map
-------------

* 200K decomposable / 2^21
* combining class lookup
* decompose map lookup

Naive Approach
--------------

* The original code was written by Antonio Nikishaev as part of the prose
  package
* `Adapted to use IntMap instead of Map <https://github.com/harendra-kumar/unicode-transforms/commit/e41b6bd1507cb9f8fb958751843b49463112bfc8>`_ ; IntMap works slightly better.

* The figures in the table are time taken in milliseconds to optimize a sample dataset.

+-----------------------------------------+
| Concise and beautiful haskell code      |
+---------+---------+------------+--------+
|         | English | Devanagari | Korean |
+---------+---------+------------+--------+
| ICU     | 3       | 10         | 32     |
+---------+---------+------------+--------+
| Haskell | 204     | 243        | 742    |
+---------+---------+------------+--------+
|         | 68x     | 24x        | 23x    |
+---------+---------+------------+--------+

Some Naive Optimizations
------------------------

* `ghc pattern match <https://github.com/harendra-kumar/unicode-transforms/commit/8085d1a468b3cab841f74f847cb06c3d5c78f101>`_
* `Add missing files <https://github.com/harendra-kumar/unicode-transforms/commit/c70a8a4342c281e207391ec9865206c364eba9c7>`_
* `pattern match on cc <https://github.com/harendra-kumar/unicode-transforms/commit/7c1ccb7f0ac86b7f0234d7e9e928921689bd46b9>`_

+---------+---------+------------+--------+
| Haskell | 172     | 195        | 381    |
+---------+---------+------------+--------+

First level lookup using bitmap
-------------------------------

* `decomposable check using bitmap <https://github.com/harendra-kumar/unicode-transforms/commit/5e87d6d109f9b4f8690e8d911f9a30f8e05076cb>`_
* `enable the check <https://github.com/harendra-kumar/unicode-transforms/commit/0a607730c981eeb2f6a66223f5df4f3902769c5c>`_

+---------+---------+------------+--------+
| Haskell | 162     | 187        | 408    |
+---------+---------+------------+--------+

More Optimizations
------------------

* `rewrite decompose <https://github.com/harendra-kumar/unicode-transforms/commit/bdf733e09e033770fa06d9f2ae00ada22b3ce459>`_
* `fix decompose <https://github.com/harendra-kumar/unicode-transforms/commit/4412c6791903c5fb071a3a678c332459f9114a6f>`_
* `rewrite reorder <https://github.com/harendra-kumar/unicode-transforms/commit/28afe9b645b5c15f576b893f522b8305fcd9848c>`_

  * sequence of starters
  * Just one combining char
  * sorting of 2 or more CC

* `bitmap for combining class (zero/non-zero) check <https://github.com/harendra-kumar/unicode-transforms/commit/9b110443e1fa9237eec2a619fd03e65a3c3665b9>`_

+---------+---------+------------+--------+
| Haskell | 21      | 32         | 166    |
+---------+---------+------------+--------+

Hangul-Jamo Normalization
~~~~~~~~~~~~~~~~~~~~~~~~~

.. _5a379d: https://github.com/harendra-kumar/unicode-transforms/commit/5a379d4a223aecb325aed160627d62b250d0af04
.. _d4d006: https://github.com/harendra-kumar/unicode-transforms/commit/d4d006bc31d19960b7c081d23ebbddf98fbfb00f
.. _a67e1c: https://github.com/harendra-kumar/unicode-transforms/commit/a67e1c8f6538caa3fb63b326083d09b020732497
.. _c8aff4: https://github.com/harendra-kumar/unicode-transforms/commit/c8aff459a7ed5e61b13842247858dbd70e670abf
.. _40c119: https://github.com/harendra-kumar/unicode-transforms/commit/40c119a0523beb9a3edc57192348efe6fd0097a5

+-----------+-------------------------------------------------------------+
| `5a379d`_ | (algorithmic decomposition)                                 |
+-----------+-------------------------------------------------------------+
| `d4d006`_ | * use quot/rem instead of div/mod                           |
|           | * use strict list construction                              |
+-----------+-------------------------------------------------------------+
| `a67e1c`_ | * return tuples instead of list                             |
|           | * Special case hangul path (avoid decomposable check)       |
+-----------+-------------------------------------------------------------+
| `c8aff4`_ | * Use quotRem instead of separate quot and rem (for hangul) |
+-----------+-------------------------------------------------------------+
| `40c119`_ | * Fuse reorder and decompose functions manually             |
+-----------+-------------------------------------------------------------+

Other Optimizations
~~~~~~~~~~~~~~~~~~~

.. _cbfab0: https://github.com/harendra-kumar/unicode-transforms/commit/cbfab0fc200476736c57065cca16f7b41364cbde
.. _4c2bca: https://github.com/harendra-kumar/unicode-transforms/commit/4c2bca04f7602d53aa6ef4fbfe0d2016d1cdad01

+-----------+----------------------------------------------------------------------------+
| `cbfab0`_ | Manually unfold string concatenation (for short strings) (10% improvement) |
+-----------+----------------------------------------------------------------------------+
| `4c2bca`_ | Optimized sorting for the case of 2 chars (5-10% boost)                    |
+-----------+----------------------------------------------------------------------------+

+---------+---------+------------+--------+
| Haskell | 14      | 23         | 36     |
+---------+---------+------------+--------+

Better than the C utf8proc implementation, using String!

Back to basics
--------------

* `string-text-overhead <https://github.com/harendra-kumar/unicode-transforms/blob/string-text-overhead/benchmark/Benchmark.hs>`_

::

  stringOp = map (chr . (+ 1) . ord)
  textOp = T.map (chr . (+ 1) . ord)

+-------------------+------------------------+----------------------+
| ICU Normalization | Do nothing String loop | Do nothing Text loop |
+-------------------+------------------------+----------------------+
| 4                 | 17                     | 11                   |
+-------------------+------------------------+----------------------+

Even with a simple operation on a string/text we are far off from the icu
figures of full normalization!

Text Stream-Unstream overhead
-----------------------------

* `stream-unstream-overhead <https://github.com/harendra-kumar/unicode-transforms/blob/stream-unstream-overhead/benchmark/Benchmark.hs>`_
* `Fixed the text library <https://github.com/bos/text/commit/09843692c764a3628c7161e586321ed60adfe3c7>`_

+-------------------+---------------------------+-----------+
| ICU Normalization | Text just stream/unstream | With Fix  |
+-------------------+---------------------------+-----------+
| 2.7               | 4                         | 1.3       |
+-------------------+---------------------------+-----------+

Text Stream Based Implementation
--------------------------------

.. _8996d4: https://github.com/harendra-kumar/unicode-transforms/commit/8996d48c421729e2d0e58765d977c49acc10f0c4
.. _189cbe: https://github.com/harendra-kumar/unicode-transforms/commit/189cbe1d08e64259fbb68a5966fde091f17ad0c0

+-----------+------------------------------------------------------+
| `8996d4`_ | unstream NFD Text, Inlining (6.3 ms on English)      |
+-----------+------------------------------------------------------+
| `189cbe`_ | * inline the combining char check                    |
|           | * Provides around 16% improvement (6.3 ms => 5.3 ms) |
|           |   on English benchmark                               |
+-----------+------------------------------------------------------+

Hack to generate better code
----------------------------

* `hack <https://github.com/harendra-kumar/unicode-transforms/commit/6793fc5999d02dae269ba3ca62a06f3e41379629>`_
* Force GHC to unfold instead of generating a function call
* Provides around 5% improvement (5.25 ms => 4.95 ms on English benchmark)

LLVM
----

* `use llvm <https://github.com/harendra-kumar/unicode-transforms/commit/3066ba9922cf2eeb57f6a0619935e0db07dd0110>`_
* shift vs range checks
* llvm backend

10% improvement - 4.95 to 4.45 ms on English

Reorder buf single char case
----------------------------

* `reorder special case <https://github.com/harendra-kumar/unicode-transforms/commit/3bc138714f4bf887068f3e7db507a4627a46fc2a>`_
* 5% improvement

With LLVM (English)

+-----+--------------------+
| ICU | Unicode transforms |
+-----+--------------------+
| 3   | 4.2                |
+-----+--------------------+

Without LLVM

+---------+---------+------------+--------+
|         | English | Devanagari | Korean |
+---------+---------+------------+--------+
| ICU     | 2.6     | 9.1        | 32     |
+---------+---------+------------+--------+
| Haskell | 5.8     | 16         | 22     |
+---------+---------+------------+--------+

ICU uses QuickCheck properties from UCD  which this implementation does not
use.

Want to contribute?
-------------------

* Start here - https://github.com/harendra-kumar/unicode-transforms/issues

GHC Issues
----------

* https://ghc.haskell.org/trac/ghc/ticket/12231
* https://ghc.haskell.org/trac/ghc/ticket/12232
* https://github.com/harendra-kumar/unicode-transforms/tree/GHC_PERF_ISSUE/GHC_PERF_ISSUE

Lessons Learnt
--------------

* A lot can be improved just by high level optimizations like using the right
  data structures e.g. pattern match for lookup, using a two level lookup with
  bitmap at the first level.
* Strings are not too bad (we could get performance equivalent to the utf8proc
  C implementation just with Strings). But text works better if you need that
  extra performance.
* We had to sacrifice some modularity for performance by fusing the decompose
  and reorder steps together. I would have ideally liked those to be separate.
* Strictness annotations used at the right places can make a significant
  difference
* division is expensive, use quotRem when you need both quotient and remainder
* inlining plays a very important role in optimization. Manual unfolding was
  required in some cases.
* -fspec-constr came into play and helped in several cases. Sometimes
  programmer awareness and code adaptation is needed to enable this
  optimization more effectively:

  * `-fspec-constr <https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/using-optimisation.html#ghc-flag--fspec-constr>`_
  * `Spec constr paper <http://research.microsoft.com/en-us/um/people/simonpj/papers/spec-constr/spec-constr.pdf>`_
* There should be a better way (GHC change?) to avoid this `hack <https://github.com/harendra-kumar/unicode-transforms/commit/6793fc5999d02dae269ba3ca62a06f3e41379629>`_
* GHC can improve native code generation. LLVM generates better code but subtle
  changes at the source level can change the balance.
