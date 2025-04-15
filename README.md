# A Functional Language for Logarithmic Space

This repository contains the source code for the paper "A Functional Language for Logarithmic Space" [[5]](#5), provided by the original author, Peter Møller Neergaard, and modernized for easier use:
- Removed dependencies on the obsolete `mosmake` build system
- Updated the author's contact information
- Revised running instructions to work in 2025

We've also preserved several unpublished technical reports that contain valuable information otherwise difficult to find.

Below is the original README with minor adjustments for clarity.

The complete code obtained from the author is available in its original form in the [linear-bc-evaluator.tgz](https://github.com/eerio/neergaard-logspace-characterization/blob/main/linear-bc-evaluator.tgz) archive.

## BC Evaluator: Overview

This package provides an evaluator for the function algebra $\text{BC}$ for
PTIME introduced by Bellantoni and Cook [[1]](#1). Additionally, it includes:

1. A logarithmic-space evaluator for the linear fragment $\text{BC}^-$ of $\text{BC}$ as
introduced by Ong and Murawski [[2]](#2)
2. An evaluator for $\text{BC}^-_\varepsilon$, another linear fragment described by Møller Neergaard and Mairson [[3](#3), [4](#4)]

The evaluator is written in Standard ML of New Jersey (SML/NJ) and has been
tested under GNU/Linux. The implementation is spread across several source
files that can be compiled together using SML/NJ's Compilation Manager (CM).

### Installation and Running

To run the evaluator:

```
# Install SML/NJ and rlwrap (for better command line editing)
$ sudo apt install smlnj rlwrap

# Note: The version you'll get (likely v110.79+) is newer than the original 
# version (v110.0.7) used by the author, but it should work fine

# Start SML with enhanced readline support
$ rlwrap sml
Standard ML of New Jersey v110.79 [built: Mon Apr 22 10:14:55 2024]

# Compile the project using CM
- CM.make "sources.cm";
[...]

# Run the test suite with the reference evaluator
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

### Evaluator Implementations

The package includes four different evaluator implementations:

- `LogspaceEvaluator`: A logspace evaluator for $\text{BC}^-$ as presented in [[3]](#3). This is 
  the core implementation demonstrating that $\text{BC}^-$ can be evaluated in logarithmic space.

- `LogspaceEvaluatorFunc`: A more functional implementation supporting both $\text{BC}^-$ and
  $\text{BC}^-_\varepsilon$ as presented in [[4]](#4). This version improves upon the first implementation
  while maintaining the logspace bounds.

- `LogspaceEvaluatorFunctional`: A completely functional evaluator for $\text{BC}^-$ and
  $\text{BC}^-_\varepsilon$.

- `CBVEvaluator`: A standard call-by-value evaluator supporting all fragments ($\text{BC}$, $\text{BC}^-$, and
  $\text{BC}^-_\varepsilon$). This implementation serves as a reference implementation and does not
  guarantee logspace evaluation.

### Creating and Using Syntax Trees

Input to the evaluators consists of syntax trees (type `Syntax.S`) and two lists of integers 
representing the normal and safe arguments. You can construct syntax trees using these functions:

```
zero : Syntax.S                                 // Constant zero function
s0 : Syntax.S                                   // Prepend 0 bit function
s1 : Syntax.S                                   // Prepend 1 bit function
p : Syntax.S                                    // Predecessor function
c : Syntax.S                                    // Conditional function
proj : int * int * int -> Syntax.S              // Projection function
scomp : Syntax.S * Syntax.S list * Syntax.S list -> Syntax.S  // Safe composition
srec : Syntax.S * Syntax.S * Syntax.S -> S      // Safe recursion
```

### Predefined Examples and Testing

The `Examples` structure provides a collection of predefined syntax trees for testing. You can use the `Examples.test_suite` function to run an evaluator against all predefined examples. The output is prefixed with `!!` if the result deviates from the expected value.

Note that due to differences in available arguments for recursive functions, some examples might produce incorrect output with certain evaluators (e.g., `CBVEvaluator.eval_bc` on `Examples.minus` because `Examples.minus` is designed for linear recursion).

### Important Implementation Details

Here are some key aspects of the logspace evaluator implementations:

- **Safe Composition:** The original definition of $\text{BC}^-$ [[2]](#2) explicitly separates safe arguments during safe composition. This implementation uses the standard $\text{BC}$ syntax but verifies that arguments can be split affinely.

- **Bit Evaluation Order:** The implementation evaluates bits from least significant to most significant, stopping when the first non-existing bit is encountered. This is a cheat, as it violates logspace constraints (output needs to be reversed) and might introduce leading zeros. A correct logspace implementation would evaluate bits from most significant to least significant, outputting bits only after encountering the first '1' bit. This optimization was chosen to simplify the core argument (finding one bit in logspace) and avoid iterating through non-existent bits.

- **Non-Linear Functions:** It's instructive to test the logspace evaluators with the non-linear $\text{BC}^-$ function `Examples.cbv_show`. This will result in an infinite loop because it requires more than one bit from the recursive call.

### Limitations

Due to limitations of the SML/NJ `int` type, the evaluator is restricted to 16-bit numbers.

### Contact

For questions or comments, please contact `peter (at) mollerneergaard (dot) net` (the original author) or `p (dot) balawender (at) mimuw (dot) edu (dot) pl` (the author of this revision).

### References

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

<a id="4">[4]</a> P. Møller Neergaard. $\text{BC}^-_\varepsilon$ : A recursion-theoretic
characterization of logspace. Technical report, Brandeis University,
March 2004. Preliminary version.
- source: https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=d00d19870bb43abb4b0098e382ddc7f54d2ddf48
- mirror: http://web.archive.org/web/20250411114452/https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=d00d19870bb43abb4b0098e382ddc7f54d2ddf48

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
