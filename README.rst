========================
Scientific Working Group
========================

This repository is for storing `Scientific Working Group
<https://wiki.openstack.org/wiki/Scientific_working_group>`_ resources.
Principally, this includes the text for the WG's book,
`The Crossroads of Cloud and HPC: OpenStack for Scientific Research
<https://www.openstack.org/assets/science/OpenStack-CloudandHPC6x9Booklet-v4-online.pdf>`_

All documents are in RST format.

Building
========

Python Tox is used to automate the creation of virtual environments for
building the working group's documentation resources.

Prerequisites
-------------

To get started, you need to install all necessary tools:

 * `virtualenv`
 * `pip` (use the latest from `https://bootstrap.pypa.io/get-pip.py`)
 * `tox`

Run the build
-------------

To build the resources::

   $ tox -e docs

This will generate build artifacts in ``doc/build``.  To view the generated
HTML artifacts navigate to
`file:///path/to/scientific-wg/doc/build/html/index.html`.
