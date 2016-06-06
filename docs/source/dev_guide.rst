.. _devguide:

Developer Guide
===============

.. contents::
    :local:
    :depth: 2


.. _writing_docs:
            
Writing Documentation
---------------------

Documentation is written in `ReStructuredText <http://sphinx-doc.org/rest.html>`_ and generated with `Sphinx <http://sphinx-doc.org/index.html>`_. The Birdhouse components use the buildout recipe `birdhousebuilder.recipe.sphinx <https://pypi.python.org/pypi/birdhousebuilder.recipe.sphinx>`_ which sets up Sphinx and a minimal ``docs`` folder. With ``make docs`` the documentation is generated locally. The documentation is published to `Read the Docs <https://readthedocs.org/>`_ with each commit to the `master` branch. The API reference is generated automatically using the Sphinx plugin `AutoAPI <http://sphinx-autoapi.readthedocs.org/en/latest/index.html>`_.

* http://sphinx-doc.org/tutorial.html
* http://quick-sphinx-tutorial.readthedocs.org/en/latest/

.. _anaconda:

Using Anaconda in Birdhouse
---------------------------

The installation of the Birdhouse components and especially the processes involve many software dependencies. The core dependencies are of course the WPS related packages like :term:`PyWPS` and :term:`OWSLib` from the :term:`GeoPython` project. But most dependencies come from the processes itself served by the WPS like `numpy`, `R`, `NetCDF`, `CDO`, `matplotlib`, `ncl`, `cdat`, ... and many more. 

The aim of Birdhouse is to take care of all these dependencies so that the user does not need to install them manually. If these dependencies would only be *pure* Python packages then using the :term:`Buildout` build tool together with the Python package index :term:`PyPi` would be sufficient. But many Python packages have `C` extensions and there are also non-Python packages we need to install like `R` and :term:`NetCDF`.

In this situation the :term:`Anaconda Python distribution` comes helpful. Anaconda has already a lot of Python related packages available for different platforms (Linux, MacOSX, Windows) and there is no compilation needed on the installation host. Anaconda makes it easy to build own packages (*conda recipes*) and to upload them to the free Anaconda server on :term:`Binstar`.

Conda recipes by Birdhouse
~~~~~~~~~~~~~~~~~~~~~~~~~~

Birdhouse is using :term:`Anaconda` to maintain package dependencies. 
Anaconda gives you the possibility to write your own `conda recipes <http://conda.pydata.org/docs/build.html>`_.
In Birdhouse we have written several conda recipes for the packages that were not available on Anaconda.  
These `additional conda recipes by Birdhouse <https://github.com/bird-house/conda-recipes>`_ are available on GitHub. 
Some of the missing packages are: :term:`PyWPS`, :term:`OWSLib`, :term:`cfchecker`, :term:`Nginx`, ...

Anaconda provides a free :term:`Anaconda Server`. Here you can upload your builded conda packages for different platforms (Linux, MacOX, Windows). These packages are then available for installation with the :term:`conda` installer.

`Birdhouse has an organisation <https://anaconda.org/birdhouse>`_ where all conda packages are collected which are 
builded from the conda recipes on GitHub. These packages can be installed with the :term:`conda` installer using the `birdhouse` channel.
For example if you are already using Anaconda, you can install :term:`CDO` with the following command:

.. code-block:: sh

    $ conda install --channel birdhouse pywps

Building conda packages
~~~~~~~~~~~~~~~~~~~~~~~

There are several ways to build conda packages and upload them to *Anaconda Server*:

* You can `build packages locally <http://conda.readthedocs.io/en/latest/#building-your-own-packages>`_ and upload them with the Binstar command line tool.
* You can also `build packages remotely on Anaconda <http://docs.anaconda.org/draft/using.html>`_. Additionally you can set a GitHub Webhook so that on each commit of your recipe a build will be run on Binstar. 
* The remote build on Anaconda are done using Docker images. The `Anaconda docker image for Linux-64 <https://hub.docker.com/r/binstar/linux-64/>`_ is available on :term:`Docker Hub`.  

In Birdhouse we usually use the remote build on Anaconda which is triggered by commits to GitHub. 
But sometimes the docker image for Linux-64 provided by Binstar fails for some packages. 
That is why `Birdhouse has in addition its own Linux-64 build image <https://hub.docker.com/r/birdhouse/binstar-linux-64/>`_ which is based on the Anaconda image. 
The `Dockerfile for this image <https://github.com/bird-house/birdhouse-build/tree/master/docker/binstar-linux-64>`_ is on GitHub.

.. warning::

   When you build conda packages for Linux-64 you need to be very careful to ensure that these packages will run on most Linux distributions (like :term:`CentOS`, :term:`Debian`, :term:`Ubuntu`, ...). Our experience is that packages build on CentOS 6.x will also run on recent Debian/Ubuntu distributions. The Docker build images are also CentOS 6.x based.  


.. note::

   You can build a conda package with the provided docker image for Linux-64. See the `readme <https://github.com/bird-house/birdhouse-build/tree/master/docker/binstar-linux-64>`_ on how to use it.

Example: building conda package for pygbif
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``pygbif`` is a Python package available on :term:`PyPi`. Generate conda package files using ``conda skeleton``::

    $ conda skeleton pypi pygbif
    $ cd pygbif
    $ vim meta.yaml  # check dependencies, test, build number
    $ vim build.sh   # in case of non-python packges here is most of the work to do
    
Enable anaconda build::
     
    $ cd pygbif
    $ anaconda-build init
    $ vim .binstar.yml
    
Edit the anaconda config (``binstar.yml``) to have the following entries (change the package name for a different recipe):

.. literalinclude:: binstar.yml
    :language: yaml
    :linenos:

Run binstar build for the first time:

.. code-block:: sh

    $ binstar package --create birdhouse/pygbif
    $ anaconda-build submit .
    $ anaconda-build tail -f birdhouse/pygbif 1    # checks logs

On successful build go to the birdhouse channel on binstar and search for the `pygbif package` (``http://anaconda.org/birdhouse/pygbif/files``). 
Go to the ``files`` tab and add the channel `main` for the successfully builded package. 
All packages on the `main` channel are available for public usage.

.. image:: _images/binstar_channel.png

Register GitHub webhook for pygbif: 

on Anaconda Server go to `Settings/Continuous Integration` of the ``pygbif`` package. 

Edit the fields:

* `github.com/` = `bird-house/conda-recipes`
* Subdirectory = pygbif
 
.. image:: _images/binstar_ci.png

.. warning::
   
   If you're logged into anaconda with your own rather than the `birdhouse` organization account then the ``anaconda-build submit .`` way mentioned above seem to cause some problems (as of October 2015). A more reliable way to upload your package is to build it locally, upload it to your own account and then transfer the ownership to `birdhouse` via the web interface:

.. code-block:: sh

    $ anaconda-build init # just as before
    $ vim .binstar.yaml
    $                     # skip package creation here
    $ conda build .       # build locally
    $ anaconda upload /your/path/to/conda-bld/platform/packagename-version.tar.bz2 # full path is listed in conda build output

   Now switch to `anaconda.org/yourname/packagename` and go to `Settings` -> `Admin` -> `Transfer` to transfer the package to `birdhouse`. (You could use ``-u birdhouse`` to upload it to `birdhouse` directly, but it seems to make some difference e.g. some fields in the web interface will not be filled in automatically, so I figured the other workaround to be more reliable.)

Using Conda
~~~~~~~~~~~

See the `conda documentation <http://conda.pydata.org/docs/index.html>`_.

.. warning::

   To fix the SSL cert issues in conda when updating to python 2.7.9 do the following::

     $ conda config --set ssl_verify False
     $ conda update requests openssl
     $ conda config --set ssl_verify True

   See this conda issue at https://github.com/conda/conda/issues/1624


Anaconda Alternatives
~~~~~~~~~~~~~~~~~~~~~

If Anaconda would not be available one could also provide these packages from source and compile them on each installation host. Buildout does provide ways to do so. But an initial installation with most of the software used in climate science could *easily take hours*. 

Alternative package manager to Anaconda are for example :term:`Homebrew` (MacOSX only) and :term:`Linuxbrew` (a fork of Homebrew for Linux).

Using Buildout in Birdhouse
---------------------------

Birdhouse uses the :term:`Buildout` build tool to install and configure all Birdhouse components (:term:`Phoenix`, :term:`Malleefowl`, :term:`Emu`...). The main configuration file is ``buildout.cfg`` which is in the root folder of the application. 
As an example have a look at the `buildout.cfg from Emu <https://github.com/bird-house/emu/blob/master/buildout.cfg>`_. 

Before building an application with Buildout you have an initial bootstrap step:

.. code-block:: sh

    $ python bootstrap-buildout.py -c buildout.cfg

This will generate the ``bin/buildout`` script.
Now you can build the application:

.. code-block:: sh

    $ bin/buildout -c buildout.cfg

The default configuration in the ``buildout.cfg`` should always work to run your application on ``localhost`` with default ports. You can customize the configuration by editing the ``custom.cfg`` which extends and overwrites the settings of ``buildout.cfg``. You may have a look at the
`custom.cfg example of Emu <https://github.com/bird-house/emu/blob/master/custom.cfg.example>`_. So, instead of using ``buildout.cfg`` you should use ``custom.cfg`` for the build:

.. code-block:: sh

    $ bin/buildout -c custom.cfg

For convenience Birdhouse has a Makefile which hides all these steps. If you want to build an application you just need to run:

.. code-block:: sh

    $ make install

See the `Makefile example of Emu <https://github.com/bird-house/emu/blob/master/Makefile>`_
For more details see the :ref:`installation` section and the :ref:`Makefile documentation <bootstrap:makefile>`.


Buildout Recipes by Birdhouse
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

:term:`Buildout` has a plugin mechanism to extend the build tool functionality with `recipes <http://www.buildout.org/en/latest/docs/recipe.html>`_. Buildout can handle Python dependencies by its own. But in Birdhouse we install most dependencies with Anaconda. We are using a Buildout extension to install conda packages with Buildout. Buildout does use these Python packages instead of downloading them from :term:`PyPi`. 
There is also a set of recipes to set up Web Processing Service with :term:`PyWPS`, :term:`Nginx`, :term:`Gunicorn` and :term:`Supervisor`. 
All these `Buildout recipes are on GitHub <https://github.com/bird-house?query=birdhousebuilder.recipe>`_ and can be `found on PyPi <https://pypi.python.org/pypi?%3Aaction=search&term=birdhousebuilder.recipe&submit=search>`_. 

Here is the list of currently used Buildout recipes by Birdhouse:

* `birdhousebuilder.recipe.conda <https://pypi.python.org/pypi/birdhousebuilder.recipe.conda>`_: A Buildout recipe to install Anaconda packages.
* `birdhousebuilder.recipe.pywps <https://pypi.python.org/pypi/birdhousebuilder.recipe.pywps>`_: A Buildout recipe to install and configure PyWPS Web Processing Service with Anaconda.
* `birdhousebuilder.recipe.pycsw <https://pypi.python.org/pypi/birdhousebuilder.recipe.pycsw>`_: A Buildout recipe to install and configure pycsw Catalog Service (CSW) with Anaconda.
* `birdhousebuilder.recipe.nginx <https://pypi.python.org/pypi/birdhousebuilder.recipe.nginx>`_: A Buildout recipe to install and configure Nginx with Anaconda.
* `birdhousebuilder.recipe.supervisor <https://pypi.python.org/pypi/birdhousebuilder.recipe.supervisor>`_: A Buildout recipe to install and configure supervisor for Anaconda.
* `birdhousebuilder.recipe.docker <https://pypi.python.org/pypi/birdhousebuilder.recipe.docker>`_: A Buildout recipe to generate a Dockerfile for Birdhouse applications.
* `birdhousebuilder.recipe.sphinx <https://pypi.python.org/pypi/birdhousebuilder.recipe.sphinx>`_: A Buildout recipe to generate documentation with Sphinx.
* `birdhousebuilder.recipe.ncwms <https://pypi.python.org/pypi/birdhousebuilder.recipe.ncwms>`_: A Buildout recipe to install and configure ncWMS2 Web Map Service.
* `birdhousebuilder.recipe.adagucserver <https://pypi.python.org/pypi/birdhousebuilder.recipe.adagucserver>`_: A Buildout recipe to install and configure Adagucserver Web Map Service.

Python Packaging
----------------

Links:

* https://python-packaging-user-guide.readthedocs.org/en/latest/

Example:

.. code-block:: sh

    $ python setup.py sdist
    $ python setup.py bdist_wheel
    $ python setup.py register -r pypi
    $ twine upload dist/*

Check the rst docs in the long_description of ``setup.py``:

* https://github.com/collective/collective.checkdocs

Example:

.. code-block:: sh

    $ python setup.py checkdocs


Writing a WPS process
---------------------

In Birdhouse we are using the :term:`PyWPS` implementation of a :term:`Web Processing Service`. Writing a WPS process in Birdhouse is the same as in PyWPS. The PyWPS documentation has a `tutorial on writing a process <http://pywps.org/docs/>`_. *Please* follow this PyWPS tutorial. 

To get easier started you can install :ref:`Emu <emu:installation>` with some example processes for PyWPS.


 










