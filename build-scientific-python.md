# Setting up Scientific Python Stack

At work I'm trying to bring up numpy/python on a ARM board (imx6). The device image is being built from a yocto tool chain, but it's proving difficult to get numpy/scipy working with python3. There are python2 scipy yocto recipes around, but it doesn't seem straightfoward to port it to python3.

What works so far:

* Build python3.5 and including both python3.5 itself, and the equivalent of python-dev onto the image
* Build numpy and include onto the image
* Build gfortran and include onto the image (more or less). We're missing `/usr/lib/libgfortran.spec`, but it's its a simple file (we'll get to it below) that we're just manually adding to the image after deploy (at runtime)
* Build openblas and include onto the image (it's ending up at `/opt/openblas`)

What's not working:

* Building scipy from yocto
* Building scipy on the board after deploy
   * We've been able to point scipy to use the openblas on the image
   * We've been able to compile the cpp code by adding swap (on that note, we were reaching just over 1GiB of memory usage while compiling the cpp code)
   * It's failing to build because it cannot find `libnpymath`. Went looking manually, and it is indeed not on the image
   
Our new approach:

* Rebuild everything manually on the device:
    * [x] Python (may as well go to 3.6/3.7)
    * [x] BLAS (OpenBLAS)
    * [x] LAPACK
    * [x] numpy (btw, this needs like ~90 MiB of free space for the src/build directory)
        * Ran into a fun problem where we failed a bunch of numpy tests since our image doesn't come with `ldd`... we'll get that fixed
    * [ ] scipy (btw, this needs like ~300+ MiB of free space for the src/build directory)
    
 Resources that I'm leveraging heavily here:
 
 * https://stackoverflow.com/questions/7496547/does-python-scipy-need-blas
 
 Oh also, if you just following that source's instructions, you'll build static libraries for blas/lapack and statically link them into numpy and scipy. For our application, I think that's ok... but take that into consideration. 
 
 Eventually I would like to talk about:
 
 * What BLAS and LAPACK are actually doing/for and how they relate to numpy/scipy
 * Building more libraries:
     * scikit-image
     * scikit-learn
     * pandas
 * Talk about how we might be able to get some of this into yocto, or at least cross-compiling on a yocto produced cross-compiler environment
