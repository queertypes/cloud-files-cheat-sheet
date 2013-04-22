***************************
Openstack Swift Cheat Sheet
***************************

:version: 0.9.6

This is a collection of notes for using an Openstack Swift endpoint for data storage operations. It also assumes that you have access to a connected Openstack Keystone endpoint.

**Note**: This document was created against the Rackspace Cloud Files and Identity APIs. There should be no differences between these APIs and the Openstack equivalents, but please notify me if there are.

========
Building
========

You can build the cheat sheet locally by running and installing `rst2pdf`_:

.. code-block:: bash

    rst2pdf cheat-sheet.rst

.. _rst2pdf: http://rst2pdf.ralsina.com.ar/

When printing the cheat sheet, it's helpful to remove the table of contents. To do so, just edit the cheat-sheet.rst and remove the following lines:

.. code-block:: rst

    **Table of Contents**

    .. contents::
        :local:
        :depth: 2

    .. raw:: pdf

        PageBreak


Rebuild the document and it's ready for printing!

======
Author
======

If you have any questions, corrections, or contributions, please contact Alejandro Cabrera <alejandro.cabrera@rackspace.com>.
