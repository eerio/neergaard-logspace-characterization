# A functional language for logarithmic space
This repository contains the original source code for the paper
"A Functional Language for Logarithmic Space" [[5]](#5), along with
small refreshments where they were needed, such as removing the dependency
on obsolete `mosmake` build system, updating the author's contact information,
changing running instructions to those working in 2025. We also preserve here
the non-published historical technical reports, which contain a lot of useful,
and otherwise unaccessible information.

The below is the original README file with minor adjustments.
The whole code obtained from the author is present as-is in the linear-bc-evaluator.tgz file
in this repository.

## $\text{BC}$ Evaluator
This package contains an evaluator for the function algebra $\text{BC}$ for
PTIME introduced by Bellantoni and Cook [[1]](#1).  It furthermore contains
an evaluator running in logspace for the linear fragment $\text{BC}^-$ of $\text{BC}$ as
introduced by Ong and Murawski [[2]](#2) and the linear fragment $\text{BC}^-_\varepsilon$
as introduced by Møller Neergaard [[4]](#4).  The evaluator is described by
Møller Neergaard and Mairson [[3](#3), [4](#4)].  It is intended to provide a proof
of concept on top of the proof given in [[3](#3), [4](#4)].

The evaluator is written in SML/NJ (Version 110.0.7) and has been
tested under GNU/Linux.  The interpreter is provided in several source
files and are easiest compiled using the SML/NJ's Compilation Manager:

Enter the directory of the source files. Then:
```
# This will install a version which is neither new (at the time of writing, it's from 2022) nor
# matching the version which had been used by Peter Moller Neergaard. But it seems to work well
$ sudo apt install smlnj rlwrap
# rlwrap to be able to use arrows
$ rlwrap sml
Standard ML of New Jersey v110.79 [built: Mon Apr 22 10:14:55 2024]
- CM.make "sources.cm";
[...]
- Examples.test_suite CBVEvaluator.eval_bc;
```

The evaluator package consists of four evaluators which all adhere to
the signature given in `Evaluator-sig.sml`.  That means that they
contain the following functions:
```
eval_bc : 'a Syntax.AS -> Numbers.int list -> Numbers.int list -> Numbers.int
eval_bcm : 'a Syntax.AS -> Numbers.int list -> Numbers.int list -> Numbers.int
eval_bcmeps : 'a Syntax.AS -> Numbers.int list -> Numbers.int list -> Numbers.int
```

The first is an evaluator for $\text{BC}$ [[1]](#1), the second is an evaluator for
$\text{BC}^-$ [[2]](#2), and the third is an evaluator for $\text{BC}^-_\varepsilon$ [[4]](#4).  Some of
the evaluators are unable to evaluate all fragments; if the expression
cannot be evaluated, the exception `Types.TypeFailure` is raised.

The evaluators are

- `LogspaceEvaluator`: the logspace evaluator for $\text{BC}^-$ presented in [[3]](#3).

- `LogspaceEvaluatorFunc`: a more functional version for $\text{BC}^-$ and
  $\text{BC}^-_\varepsilon$ as presented in [[4]](#4).

- `LogspaceEvaluatorFunctional`: a completely functional evaluator for
  $\text{BC}^-$ and $\text{BC}^-_\varepsilon$.

- `CBVEvaluator`: a standard call-by-value evaluator for $\text{BC}$, $\text{BC}^-$, and
  $\text{BC}^-_\varepsilon$.  This is intended for reference.

Input to the evaluators are syntax trees of type `Syntax.S` and lists of
integers with the normal and safe arguments.  Syntax trees are build
using the following functions:

```
zero : Syntax.S
s0 : Syntax.S
s1 : Syntax.S
p : Syntax.S
c : Syntax.S
proj : int * int * int -> Syntax.S
scomp : Syntax.S * Syntax.S list * Syntax.S list -> Syntax.S
srec : Syntax.S * Syntax.S * Syntax.S -> S
```

There are a number of predefined syntax trees available in the
structure `Examples`.  In particular, applying `Examples.test_suite` to
one of the evaluator functions tests the evaluator function on all the
input in `Examples`; it precedes the line with `!!` if the output deviates
from the expected output.  Note however that due to the differences in
the arguments available to the recursive function some of the examples
will produce the wrong output (e.g., `CBVEvaluator.eval_bc` on
`Examples.minus` gives the wrong result because `Examples.minus` is for
linear recursion).

There are a couple of points to notice about the of the logspace
implementations:

- the definition of $\text{BC}^-$ in [[2]](#2) explicitly splits the safe arguments of
  safe composition.  We use the standard syntax for $\text{BC}$ and just check
  that the arguments can be split affinely.

- we cheat slightly in the implementation: We evaluate the bits from
  least significant to most significant and stop outputting when we
  reach the first non-existing bit.  This is wrong as it violates
  logspace (the output needs to be reversed) and it might lead to
  vacuous zeroes in front of the first significant digit.  A correct
  implementation would rather evaluate the bits from most significant
  to least significant bit and first output bit after the first 1-bit
  has been encountered.  We have chosen the alternative implementation
  as it does not change the core argument (one bit can be found in
  logspace) and it avoids idling through a number of non-existing
  bits.

- it is instructive to try the logspace-evaluator on the non-linear
  $\text{BC}^-$ function `Examples.cbv_show`.  It ends up looping infinitely
  because more than one bit of the recursive call is needed.

Due to the restrictions on SML/NJ int type the evaluator is limited to
16-bit numbers.

If you have any questions or comments, please contact
peter (at) mollerneergaard (dot) net

<a id="1">[1]</a>
Bellantoni, S., Cook, S. A new recursion-theoretic characterization of the polytime functions. Comput Complexity 2, 97–110 (1992). 
- source: https://doi.org/10.1007/BF01201998
- mirror: https://www.cs.toronto.edu/~sacook/homepage/ptime.pdf

<a id="2">[2]</a> A. S. Murawski and C.-H. L. Ong. Can safe recursion be interpreted
in light logic? In 2nd International Workshop on Implicit
Computational Complexity, June 2000.
- can't find source, but here's a related paper: https://www.sciencedirect.com/science/article/pii/S030439750300522X/pdf?md5=cc7da992d6c40e82c4578555b95bea6c&pid=1-s2.0-S030439750300522X-main.pdf&_valck=1 

<a id="3">[3]</a> P. Møller Neergaard and H. G. Mairson.  How Light is Safe
Recursion?  Translations Between Logics of Polynomial Time.
- source: https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=a1670570dbe8ff1f464a826db9b79e0054108c35
- mirror: http://web.archive.org/web/20250411115047/https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=a1670570dbe8ff1f464a826db9b79e0054108c35
- mirror: https://github.com/eerio/neergaard-logspace-characterization/blob/main/HowLightIsSafeRecursion.pdf

<a id="4">[4]</a> P. Møller Neergaard. $\text{BC}^-_\varepsilon$ : A recursion-theoretic
characterization of logspace. Technical report, Brandeis University,
March 2004. Preliminary version.
- source: https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=d00d19870bb43abb4b0098e382ddc7f54d2ddf48
- mirror: http://web.archive.org/web/20250411114452/https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=d00d19870bb43abb4b0098e382ddc7f54d2ddf48
- mirror: https://github.com/eerio/neergaard-logspace-characterization/blob/main/NeergaardRecursionTheoreticCharacterizationOfLogspace.pdf

<a id="5">[5]</a> Neergaard, P.M. (2004). A Functional Language for Logarithmic Space. In: Chin, WN. (eds) Programming Languages and Systems. APLAS 2004. Lecture Notes in Computer Science, vol 3302. Springer, Berlin, Heidelberg.
- source: https://doi.org/10.1007/978-3-540-30477-7_21

### Author's historical notes:
[03/11/22] I have started implemented a srec_k constructor which
  extends $\text{BC}^-$ such that the recursion scheme maintains a fixed k-bit
  register.  It works in CBV-evaluator, but I have not continued into
  the Logspace-evaluator for lack of appropriate syntax.  It should be
  implementable even there, but it is of no use for a TM simulation
  since we cannot communicate between that and a representation of the
  tape.