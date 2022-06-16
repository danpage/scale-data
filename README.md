# [SCALE](http://www.github.com/danpage/scale): data-oriented material

<!--- -------------------------------------------------------------------- --->

## Concept

Set within the context of
[side-channel attacks](http://en.wikipedia.org/wiki/Side-channel_attack)
based on
[power analysis](http://en.wikipedia.org/wiki/Power_analysis),
the
[hardware material of SCALE](http://www.github.com/danpage/scale-hw) 
has high-level goals that include reducing
[Total Cost of Ownership (TCO)](http://en.wikipedia.org/wiki/Total_cost_of_ownership)
wrt. the equipment involved.  Whether or not it succeeds, it is useful
to consider scenarios where the acquisition platform is eliminated: by
offering a suite of example data sets, the SCALE data-oriented material 
aims to support use-cases such as

- (offline) development of attacks, with zero infrastructure required
  to physically acquire traces,
- fair comparison between or benchmarking of said attacks.

It has a strict remit, namely educational use (e.g., as a lab. exercise 
or assignment), which to some extent determines goals, design decisions, 
and, ultimately, the content.

- The data sets relate to execution of
  [AES-128](http://en.wikipedia.org/wiki/Advanced_Encryption_Standard)
  encryption on a given target board.
  More specifically, the implementation roughly matches 
  FIPS-197 [Section 6.4, 1]
  (or at least the references it then cites):
  this implies that it focuses on use of an 8-bit data-path, but uses
  small, pre-computed look-up tables to support the S-box and `xtime` 
  operations.

- For each target board there are in fact *two* data sets, namely

  - one for traces acquired wrt. a    known cipher key
    (which is the same for all target boards),
    and
  - for for traces acquired wrt. an unknown cipher key
    (which differs for each target board).

  Note that the known cipher key is 
  `2B7E151628AED2A6ABF7158809CF4F3C`, 
  matching the test vector in FIPS-197 [Appendix B, 1].
  
- Each trace 

  - includes *several* fields, namely
  
    - a  plaintext,
    - a ciphertext,
      and
    - a      trace,
      or sequence of samples, that was acquired during encryption of said 
      plaintext to produce said ciphertext,
  
  - was acquired using a 
    [Picoscope 2206B](http://www.picotech.com/download/manuals/picoscope-2000-series-data-sheet.pdf),
    tuned to roughly the highest sampling frequency possible to fit the
    trace; attacks are possible with a (much) lower sampling frequency,
    but this approach captures a best-case scenario wrt. attacks using 
    said equipment,
  
  - is trimmed wrt. a trigger signal used to delineate and hence isolate 
    execution of the encryption itself, meaning they are reasonably well 
    aligned.

<!--- -------------------------------------------------------------------- --->

## Quickstart

- Install any pre-requisites, e.g., support for
  [Git Large File Storage (LFS)](http://git-lfs.github.com/)
  (without this, some content will appear as a set of pointers to data vs. the data itself).

- Clone the repo.

  ```sh
  git clone https://github.com/danpage/scale-data.git ./scale-data
  cd ./scale-data
  git submodule update --init --recursive
  source ./bin/conf.sh
  ```

- Create and populate a suitable Python
  [virtual environment](https://docs.python.org/3/library/venv.html)
  based on 
  [`${REPO_HOME}/requirements.txt`](./requirements.txt) 
  by executing
   
  ```sh
  make venv
  ```
   
  then activate it by executing
   
  ```sh
  source ${REPO_HOME}/build/venv/bin/activate
  ``` 

- Select a target board, e.g., by setting the environment variable

  ```sh
  export TARGET="lpc1313fbd48"
  ```

- Make use of the material:


  - fix the working directory:
  
    ```sh
    cd ${REPO_HOME}/data/scale/known
    ```
  
    then decompress the data set:
  
    ```sh
    gunzip ${TARGET}.hdf5.gzip
    ```
  
  - start an interactive 
    [Python](http://www.python.org)
    session:
  
    ```sh
    python3
    ```  
  
  - open and inspect the data set:
  
    ```py
    import os, h5py
  
    fd = h5py.File( os.environ[ 'TARGET' ] + '.hdf5', 'r' )
  
    print( fd[ 'm'     ] )
    print( fd[ 'c'     ] )
  
    print( fd[ 'trace' ] )
    ```
  
  - validate a plaintext/ciphertext pair wrt. the known cipher key:
  
    ```py
    import binascii, Crypto.Cipher.AES as AES
     
    k = binascii.a2b_hex( '2B7E151628AED2A6ABF7158809CF4F3C' )
     
    print( bytes( fd[ 'c' ][ 0 ] ) == AES.new( k, AES.MODE_ECB ).encrypt( bytes( fd[ 'm' ][ 0 ] ) ) )
    print( bytes( fd[ 'm' ][ 0 ] ) == AES.new( k, AES.MODE_ECB ).decrypt( bytes( fd[ 'c' ][ 0 ] ) ) )
    ```
  
  - visualise a trace:
  
    ```py
    import matplotlib.pyplot as mpl
   
    mpl.show( mpl.plot( fd[ 'trace' ][ 0 ] ) )
    ```

<!--- -------------------------------------------------------------------- --->

## References

1. National Institute of Standards and Technology (NIST).
   [Advanced Encryption Standard (AES)](http://doi.org/10.6028/NIST.FIPS.197).
   Federal Information Processing Standards Publication (FIPS) 197, 2001.

<!--- -------------------------------------------------------------------- --->
