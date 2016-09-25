.. _release_notes:

=============
Release Notes
=============

.. contents::
   :local:
   :depth: 2
   :backlinks: none

Bonn (August 2016)
==================

* Leaflet map with time-dimension plugin.
* using twitcher security proxy.
* using conda environments for each birdhouse compartment.
* using ansible to deploy birdhouse compartments.
* added weather-regimes and analogs detection processes.
* allow upload of files to processes.
* updated Phoenix user interface.

Paris (October 2015)
====================

* updated documents on readthedocs
* OAuth2 used for login with GitHub, Ceda, ...
* LDAP support for login
* using ncWMS and adagucwms
* register and use Thredds catalogs as data source
* publish local netcdf files and Thredds catalogs to birdhouse Solr
* qualtiy check processes added (cfchecker, qa-dkrz)
* generation of docker images for each birdhouse component
* using dispel4py as workflow engine in Malleefowl
* using Celery task scheduler/queue to run and monitor WPS processes
* improved Phoenix web client
* using birdy wps command line client


Paris (September 2014)
======================

* Phoenix UI as WPS client with ESGF faceted search component and a wizard to chain WPS processes
* PyWPS based processing backend with supporting processes of Malleefowl
* WMS service (inculded in Thredds) for visualization of NetCDF files
* OGC CSW catalog service for published results and OGC WPS services
* ESGF data access with wget and OpenID
* Caching of accessed files from ESGF Nodes and Catalog Service
* WPS processes: cdo, climate-indices, ensemble data visualization, demo processes
* IPython environment for WPS processes
* initial unit tests for WPS processes
* Workflow engine Restflow for running processing chains. Currently there is only a simple workflow used: get data with wget - process data.
* Installation based on anaconda and buildout
* buildout recipes (birdhousebuilder) available on PyPI to simplify installation and configuration of multiple WPS server
* Monitoring of all used services (WPS, WMS, CSW, Phoenix) with supervisor
* moved source code and documentation to birdhouse on GitHub


Helsinki (May 2014)
===================

* presentation of birdhouse at EGI, Helsinki
* stabilized birdhouse and CSC processes
* updated documenation and tutorials

Vienna (April 2014)
===================

* presentation of birdhouse at EGU, Vienna
* "quality check" workflow for CORDEX data