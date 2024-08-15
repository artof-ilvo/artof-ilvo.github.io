ARTOF documentation project
===========================

Getting started
---------------

This GitHub project provides the documentation for the Agricultural Robot Taskmap Operaiton Framework (ARTOF) using Sphinx docs.

For more information of using Sphinx docs read `the Sphinx documentation <https://docs.readthedocs.io/en/stable/intro/getting-started-with-sphinx.html>`_.

A github action is configured in `documentation.yml <https://github.com/artof-ilvo/artof-ilvo.github.io/blob/main/.github/workflows/documentation.yml>`_ and automatically deploys these docs to https://artof-ilvo.github.io

To build this documentation locally:

1. Create a virtual python environment

2. Install the requirements

.. code-block:: bash

   pip install -r docs/requirements.txt
3. Execute the build command

.. code-block:: bash

    sphinx-build docs/source/ _build/
4. The new index page is now available under _build/index.html

Documentation
-------------

+ reStructuredText `sublime-and-sphinx-guide <https://sublime-and-sphinx-guide.readthedocs.io/en/latest/index.html>`_ documentation.
