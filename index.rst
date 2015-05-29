.. doc_docker documentation master file, created by
   sphinx-quickstart on Fri May 29 13:06:59 2015.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

DIRG docker methodology
=======================

DIRG's main purpose of using docker is to provide an easy way of testing our tools. Even though it probably can work
in a production environment, we didn't have it in mind while setting up our docker projects.

The purpose of this documentation is to describe how we used docker, to make it possible for others to reuse and
evaluate our methodologies. We appreciates all help in improving our way of using docker.

All our docker projects are built with the same principle, so you can look at any of them to view an example. All
projects are divided into three parts, building the image, setting up the configuration and running the image.

Contents:

.. toctree::
   :maxdepth: 1

   build
   config
   run


Indices and tables
==================

* :ref:`search`

.. raw:: html

    <a href="https://github.com/its-dirg/doc_docker" class="github" target="_blank">
        <img style="position: absolute; top: 0; right: 0; border: 0;" src="_static/ViewmeonGitHub.png" alt="Fork me on GitHub"  class="github"/>
    </a>