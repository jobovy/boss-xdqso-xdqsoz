# boss-xdqso-xdqsoz

This repository contains the code that was used in training and testing the BOSS XDQSO and XDQSOz methods for quasar selection, used in the  quasar selection for BOSS' main and bonus samples. This is all described in [Bovy et al. (2010)](http://adsabs.harvard.edu/abs/2011ApJ...729..141B) and [Bovy et al. (2011)](http://adsabs.harvard.edu/abs/2012ApJ...749...41B).

## Training procedure

Training the XDQSO algorithm for all of the different i-band bins, all subsamples (stars, low-z, mid-z, and high-z quasars), and all different combinations of data sets is quite complicated in the code. The main driver is
```
deconvolve.py
```
This code takes a bunch of options, that you can see in the get_options(...) function (near the bottom). The main routine is ``deconvolve`` at the top of the file. This code does the following (for one combination of subsample and data sets):

1. It fits the Gaussians using XD for each i-band bin;
2. It makes plots of the data, model, and re-convolved model in flux-flux space (re-convolved=model convolved with representative data uncertainties)
3. It compiles a latex file with these plots
4. It makes plot of the data... in color-color space
5. It compiles a latex file with these plots

Those plots come out looking like the ones here:

http://cosmo.nyu.edu/~jb2777/bossqso/dc_galex_fluxdist_17.7_22.5_0.2_0.1_20_2010_05_29.pdf

and similar files in that directory. These are for QA purposes.

The main thread follows ``deconvolve_bin``, which runs the deconvolution for a single i-band bin. The deconvolution is done in IDL and the glue is written using a Makefile, so ``deconvolve_bin`` basically sets up the options that get passed to the Makefile and then runs ``make``. There are two relevant Makefiles: ``makefile.coadd`` and ``makefile.qso``, which are for stars and quasars respectively.

Let's take, for example, the ``makefile.qso`` Makefile. This makes a series of plots of the flux--flux distribution, but first runs the deconvolution. For lowz quasars (at the top), the code first runs ``deconvolve_qso.py`` to do the deconvolution. A bunch of options are taken by appropriately cutting up the filenames to get the i-band bin and the filename to save to. The commands following this (``plot_deconvolved_all`` and ``plot_qso_data_magbin``) makes plots of the model and data (the plots that go into the tex file).

In ``deconvolve_qso.pro``, the appropriate data is loaded (low-z quasars in our example here); these are the files that sit in a directory $BOVYQSOEDDATA. If UV (GALEX) or IR (UKIDSS) data is needed, these are loaded and matched to the SDSS data. The fluxes of all the different data sets are combined and rescaled to the i-band flux. Then everything is put into the appropriate arrays to be fed to the extreme deconvolution code. Typically, the bins are initialized with the brighter bin's best-fit (to get some continuity in the fits); therefore the code cannot be parallelized currently (although it's not necessary to start with the previous bin's best fit). The code then runs extreme deconvolution (which for historical reasons is a routine called ``projected_gauss_mixtures_c``) and saves the result.

The way GALEX and UKIDSS are incorporated is by extending this code. So, the way additional surveys could be included is to look everywhere in the code for how UKIDSS data is added and then add similar code for the new data source. In particular, this means adding a bunch of code in ``deconvolve.py``, similar to how UKIDSS options are added; adding these options to the Makefile; loading, matching, and combining the new fluxes with the rest of the fluxes; editing the plotting code to make new plots. The latter is annoying, but it's probably good to do a lot of QA. The nice thing about this approach is that the code could then work with any combination of SDSS and GALEX, UKIDSS, or the new data. 

Some files relevant for the training can be found here:

http://cosmo.nyu.edu/~jb2777/trainXDQSO/
