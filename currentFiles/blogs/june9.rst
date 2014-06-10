Monday, June 9
===============

Still workign through the packages one by one to work out the dependencies needs for each.
Here is an error I got for scipy. Evidently I need to compile some gortran! ::
    numpy.distutils.system_info.BlasNotFoundError:

        Blas (http://www.netlib.org/blas/) libraries not found.

        Directories to search for the libraries can be specified in the

        numpy/distutils/site.cfg file (section [blas]) or by setting

        the BLAS environment variable.


Just talked to John and he is saying that is may be better to go down the binary wheels route.


My virtual env was using 34 bit Python so after updating it to 64 bit numpy should be working.
Good so I was able to use *easy_install.py* to install the numpy binary I downloaded from http://www.lfd.uci.edu/~gohlke/pythonlibs/

Got about half the ...