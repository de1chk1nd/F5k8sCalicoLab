Welcome to f5 k8s set up guide for k8s w calico
===============================================

Welcome to my Lab Guide for integrating F5 Container Ingress Services with k8s and Calico as CNI.

This lab consist of 2 major parts - one for setting up the SDN (to access the k8s pods) - and the second part to set up and test ingress services.

Please also check the Pre Work, as this includes Lab information and **important** information around k8s.

If you find any errors/mistakes or want to contribute to the overall quality of the lab, please send an email to *m.petersen@f5.com* or create an issue in github.

For f5 Solution Engineers:

* The UDF Blueprint used is : **f5 CIS for k8s w calico**


Link to my **github repo** with further information and *raw* config files : `F5k8sCalicoLab <https://github.com/de1chk1nd/F5k8sCalicoLab>`_



.. toctree::
   :numbered:
   :maxdepth: 2
   :caption: Preparing the lab

   Lab Setup & Pre Reading <PreWork>
   Finishing BGP Set Up <FinishBGP>
   Test Calico SDN Networking <FinishBGP-Test>
   Preparing for basic k8s ingress <k8s_basic_ingress>


.. toctree::
   :numbered:
   :maxdepth: 2
   :caption: Creating Services

   k8s basic ingress <k8s_basic_ingress_service>
   k8s advanced ingress <adv-ing>
   k8s config maps <configmap>
   k8s custom cesource definition <crd>
   ingress vs config map vs CRDs <comparision>
