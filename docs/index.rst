
==========================================================
PyLipID: A Python package for analyzing lipid interactions
==========================================================

PyLipID is a Python package for analyzing lipid interactions with membrane proteins in
molecular dynamics simulations. Its main features include:

    * Detection of binding sites through community analysis of interaction networks.
    * Calculation of lipid dissociation rates (koff) and residence times at binding sites and residues.
    * Analysis of lipid interactions using multiple metrics.
    * Generation and clustering of representative bound poses.
    * A dual-cutoff scheme to address the "rattling-in-the-cage" effect in coarse-grained simulations.
    * Generation of publication-ready figures.


Citation |DOI for Citing PyLipID|
=================================

If you use PyLipID in scientific research, please cite the following paper: ::

	@article{song_pylipid_2022,
		author = {Song, Wanling and Corey, Robin A. and Ansell, T. Bertie and
		            Cassidy, C. Keith and Horrell, Michael R. and Duncan, Anna L.
		            and Stansfeld, Phillip J. and Sansom, Mark S.P.},
		title = {PyLipID: A Python Package for Analysis of Protein-Lipid Interactions from Molecular Dynamics Simulations},
		journal = {J. Chem. Theory Comput.},
		volume = {18},
		number = {2},
		pages = {1188--1201},
		year = {2022},
		doi = {10.1021/acs.jctc.1c00708},
		url = {https://doi.org/10.1021/acs.jctc.1c00708},
	}

.. |DOI for Citing PyLipID| image:: https://img.shields.io/badge/DOI-10.1021/acs.jctc.1c00708-blue
   :target: https://doi.org/10.1021/acs.jctc.1c00708



Installation
============

PyLipID can be installed with `pip <https://pip.pypa.io>`_

.. code-block:: bash

  $ pip install pylipid

Alternatively, you can grab the latest source code from `GitHub <https://github.com/stansfeldlab/PyLipID.git>`_:

.. code-block:: bash

  $ git clone https://github.com/stansfeldlab/PyLipID.git
  $ cd PyLipID
  $ pip install .


Usage
=====

See the :doc:`tutorial` to learn how to use PyLipID. The :doc:`api/index`
provides API documentation.

The :doc:`demo` provides a complete example of the analysis workflow.


License
=======

PyLipID is made available under the MIT License. For more details,
see `LICENSE.txt <https://github.com/stansfeldlab/PyLipID/blob/master/LICENSE.txt>`_.


Table of Contents
=================

.. toctree::
   :maxdepth: 2

   INSTALL
   api/index
   tutorial
   gallery
   demo



