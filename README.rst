The Mersenne Twister in Go
==========================

An implementation of Takuji Nishimura's and Makoto Matsumoto's
`Mersenne Twister`_ pseudo random number generator in Go.

Copyright (C) 2013  Jochen Voss

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

Please send any comments or bug reports to the program's author,
Jochen Voss <voss@seehuhn.de>.

.. _Mersenne Twister: http://en.wikipedia.org/wiki/Mersenne_twister

Overview
--------

The Mersenne Twister is a pseudo random number generator (PRNG),
developed by Takuji Nishimura and Makoto Matsumoto.  The Mersenne
Twister is, for example, commonly used in Monte Carlo simulations and
is the default random number generator for many programming languages,
e.g. Python_ and R_.  This package implements the `64bit version`_ of the
algorithm.

.. _Python: http://www.python.org/
.. _R: http://www.r-project.org/
.. _64bit version: http://www.math.sci.hiroshima-u.ac.jp/~m-mat/MT/emt64.html


Installation
------------

This package can be installed using the ``go get`` command::

    go get github.com/seehuhn/mt19937


Usage
-----

Detailed usage instructions are available via the package's online
help, either on pkg.go.dev_ or on the command line::

    go doc github.com/seehuhn/mt19937

.. _pkg.go.dev: https://pkg.go.dev/github.com/seehuhn/mt19937

The class ``MT19937`` represents instances of the Mersenne Twister.
New instances can be allocated using the ``mt19937.New()`` function.
A seed can be set using the .Seed() or .SeedFromSlice() methods.
``MT19937`` implements the ``rand.Source`` interface from the
``math/rand`` package.  Typically the PRNG is wrapped in a rand.Rand
object as in the following example::

    rng := rand.New(mt19937.New())
    rng.Seed(time.Now().UnixNano())

Note that MT19937 is not safe for concurrent accesss by different
goroutines.  If more than one goroutine accesses the PRNG, the callers
must synchronise access using sync.Mutex or similar.


Comparison to the Go Default PRNG
---------------------------------

Go has a built-in PRNG provided by the math/rand package.  I did not
find any information about this built-in PRNG except for a comment in
the source code which says "algorithm by DP Mitchell and JA Reeds".
In contrast, the MT19737 generator provided in this package is a
well-understood random number generator.  Relevant references include
[Ni2000]_ and [MatNi1998]_.

.. [Ni2000] T. Nishimura, *Tables of 64-bit Mersenne Twisters*, ACM
     Transactions on Modeling and Computer Simulation 10, 2000, pages
     348-357.
.. [MatNi1998] M. Matsumoto and T. Nishimura, *Mersenne Twister: a
     623-dimensionally equidistributed uniform pseudorandom number
     generator*, ACM Transactions on Modeling and Computer Simulation
     8, 1998, pages 3--30.

The unit tests for the mt19937 package verify that the output of the
Go implementation coincides with the output of the reference
implementation.

The mt19937 generator is slightly slower than the Go default PRNG.
A speed comparison can be performed using the following command::

    go test -bench=. github.com/seehuhn/mt19937

On my laptop, using go version 1.15.5, I get the following results:

    +----------------+---------------+----------------+
    | method         | time per call |      thoughput |
    +================+===============+================+
    | MT19937.Uint64 |  5.63 ns/op   |   1422.14 MB/s |
    +----------------+---------------+----------------+
    | MT19937.Int63  |  5.69 ns/op   |   1405.04 MB/s |
    +----------------+---------------+----------------+
    | builtin Uint64 |  4.05 ns/op   |   1973.47 MB/s |
    +----------------+---------------+----------------+
    | builtin Int63  |  4.15 ns/op   |   1929.62 MB/s |
    +----------------+---------------+----------------+

This shows that, on my system, a call to the ``Int63()`` method of the
built-in PRNG takes about 73% of the time that MT19937.Int63() takes.
