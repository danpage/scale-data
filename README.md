# [SCALE](http://www.github.com/danpage/scale): data-oriented offshoot

<!--- -------------------------------------------------------------------- --->

## Concept

Set within the context of
[side-channel attacks](http://en.wikipedia.org/wiki/Side-channel_attack)
based on
[power analysis](http://en.wikipedia.org/wiki/Power_analysis),
the
[hardware offshoot of SCALE](http://www.github.com/danpage/scale-hw) 
has high-level goals that include reducing
[Total Cost of Ownership (TCO)](http://en.wikipedia.org/wiki/Total_cost_of_ownership)
wrt. the equipment involved.  Whether or not it succeeds, it is useful
to consider scenarios where the acquisition platform is eliminated: by
offering a suite of example data sets, the SCALE data-oriented offshoot 
aims to support use-cases such as

- (offline) development of attacks, with zero infrastructure required
  to physically acquire traces,
- fair comparison between or benchmarking of said attacks.

It has a strict remit, namely educational use (e.g., as a lab. exercise 
or assignment), which to some extent determines goals, design decisions, 
and, ultimately, the content.

<!--- -------------------------------------------------------------------- --->

## Content

#### Prepare pre-requisites

Some content is stored using
[Git Large File Storage (LFS)](http://git-lfs.github.com/):
it makes sense to install this first, otherwise that content appears
as a set of pointers to data vs. the data itself.

#### Prepare     repository

This is actually a 
[submodule](http://www.git-scm.com/docs/git-submodule)
of the
[SCALE](http://www.github.com/danpage/scale)
repository: you can *either*

1. follow the instructions to prepare the SCALE repository, then
   use

   ```
   cd ${SCALE}/scale-data ; export SCALE_DATA="${PWD}"
   ```

   to fix the working directory,
   *or*

2. treat it as a standalone repository,
   using

   ```
   git clone http://www.github.com/danpage/scale-data.git
   ```
   
   to clone the repository, then
   
   ```
   cd scale-data ; export SCALE_DATA="${PWD}"
   ```

   to fix the working directory,

Note that, either way, the associated path is denoted by 
`${SCALE_DATA}` 
from here on.  

#### Use       content

Consider an example wherein

```
export TRACE="${SCALE_DATA}/trace/lpc1114fn28"
```

denotes the path to a particular target board data set,
in this case `lpc1114fn28`.
Each data set constitutes *two* trace sets, where each trace captures 
execution of one
[AES-128](http://en.wikipedia.org/wiki/Advanced_Encryption_Standard)
encryption:

- the
  `${TRACE}/known`
  trace set represents traces acquired wrt. a    known key
  (which is the same for all target boards),
  and
- the
  `${TRACE}/unknown`
  trace set represents traces acquired wrt. an unknown key
  (which differs for each target board).

Note that:

- The known key is `2B7E151628AED2A6ABF7158809CF4F3C` (matching the
  test vector in FIPS-197 [Appendix B, 1]).

- The implementation roughly matches 
  FIPS-197 [Section 6.4, 1]
  (or at least the references it then cites),
  in the sense it 

  - focuses on use of an 8-bit datapath,
    but
  - uses look-ups into pre-computed tables for the S-box and `xtime` operations.

- Each trace actually includes *several* fields, namely

  - a  plaintext (of type `bytearray`),
  - a ciphertext (of type `bytearray`),
    and
  - a trace      (of type `numpy.ndarray`, potentially implying a need for [NumPy](http://www.numpy.org)),
    or sequence of samples, that was acquired during encryption of said 
    plaintext to produce said ciphertext,

  captured as a single
  [pickle](http://docs.python.org/library/pickle.html)
  object then compressed using
  [gzip](http://en.wikipedia.org/wiki/Gzip).

- Each trace was acquired using a 
  [Picoscope 2206B](http://www.picotech.com/download/manuals/picoscope-2000-series-data-sheet.pdf),
  tuned to roughly the highest sampling frequency possible to fit the
  trace; attacks are possible with a (much) lower sampling frequency,
  but this approach captures a best-case scenario wrt. attacks using 
  said equipment.  

- Each trace is trimmed wrt. a trigger signal used to delineate and
  thus isolate execution of the encryption itself, meaning they are
  reasonably well aligned.

In terms of *using* the traces, a minimal working example could be 
as follows:

1. fix the working directory

   ```
   cd ${TRACE}/known
   ```

2. start an interactive 
   [Python](http://www.python.org)
   session

   ```
   python
   ```  

3. load a compressed example trace
  
   ```
   import gzip, pickle
     
   fd = gzip.open( 'trace-000.pkl.gz', 'r' )
     
   m = pickle.load( fd )
   c = pickle.load( fd )
   t = pickle.load( fd )
     
   fd.close()
   ```
  
4. validate the plaintext and ciphertext wrt. the known key
  
   ```
   import binascii, Crypto.Cipher.AES as AES
   
   k = bytearray( binascii.a2b_hex( '2B7E151628AED2A6ABF7158809CF4F3C' ) )
   
   str( c ) == AES.new( str( k ) ).encrypt( str( m ) )
   str( m ) == AES.new( str( k ) ).decrypt( str( c ) )
   ```
  
5. inspect the trace itself

   ```
   import matplotlib.pyplot as mpl
 
   mpl.show( mpl.plot( t ) )
   ```

<!--- -------------------------------------------------------------------- --->

## FAQs

- **"Is there anything else similar I could look at?"**
  Yes: there are plenty of great resources available, some offering more
  direct alternatives to SCALE and some of more tangential interest.
  In particular, you should have a look at

  - the [DPA contest](http://www.dpacontest.org),
  - the [CHES 2016](http://ctf.newae.com) CTF challenge.

- **"Where can I find the actual AES implementation used (i.e., the source code)?"**
  We've *deliberately* opted not to distribute the implementation: this
  choice is rationalised by two facts, namely

  1. the implementation strategy, which is basically what you need to
     reason about an associated attack, is well documented (as noted),
     and
  2. the development of such an implementation, and attacks on it, are
     a common challenge when using
     [SCALE](http://www.github.com/danpage/scale)
     in an educational context: we didn't want to give away a solution!

<!--- -------------------------------------------------------------------- --->

## References

1. National Institute of Standards and Technology (NIST).
   [Advanced Encryption Standard (AES)](http://doi.org/10.6028/NIST.FIPS.197).
   Federal Information Processing Standards Publication (FIPS) 197, 2001.

<!--- -------------------------------------------------------------------- --->
