.. _k8s-basic-ingress:

.. raw:: html

   <style> .red {color:red}; font-weight:bold </style>

.. role:: red


.. raw:: html

   <style> .blue {color:blue}; font-weight:bold </style>

.. role:: blue


Installing & testing basic k8s ingress services
===============================================

Prerequisite for ingress services
---------------------------------

Following tasks have to be done, in order to start configuring basic ingress services.

Install AS3 on :red:`bigip`.

Login to :red:`bigip (10.1.10.5)` via TMUI (Web UI)

========  ========  ==========
 Device   Username   Password
========  ========  ==========
 bigip     admin    f5twister!
========  ========  ==========

Check if AS3 is installed (iApps > Package Management LX):

.. image:: images/bigip-as3.png
   :width: 400
   :alt: Lab Overview
   :align: left

Communication from BigIP Controller for Controller Ingress Service is using AS3.


Check if basic Partition is installed (Check Partition in the top right):

   .. image:: images/bigip-partition.png
      :width: 400
      :alt: Lab Overview
      :align: left

We need to specify a default partition in the controller config - this one must match the local partition.



Download current copy from github repro
---------------------------------------

* cd ~
* svn export https://github.com/de1chk1nd/F5k8sCalicoLab/trunk/k8s k8s --force








.. toctree::
   :maxdepth: 2
