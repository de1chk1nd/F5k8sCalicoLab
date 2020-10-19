.. _k8s-basic-ingress-service:

.. raw:: html

   <style> .red {color:red}; font-weight:bold </style>

.. role:: red


.. raw:: html

   <style> .blue {color:blue}; font-weight:bold </style>

.. role:: blue

Basic Ingress Service
=====================

Now we are getting closer ;)

In this chapter we will deploy the first, basic ingress services.

Deploy "dedicated" Ingress Services
-----------------------------------

Login to :blue:`k8s master (10.1.20.20)`::

    ubuntu@ip-10-1-1-4:~/k8s/ingress$ cd /home/ubuntu/k8s/ingress
    ubuntu@ip-10-1-1-4:~/k8s/ingress$ kubectl apply -f 002a_app1.yaml
    ingress.extensions/singleingress1 created
    ubuntu@ip-10-1-1-4:~/k8s/ingress$ kubectl apply -f 002b_app2.yaml
    ingress.extensions/singleingress2 created
    ubuntu@ip-10-1-1-4:~/k8s/ingress$ kubectl apply -f 002c_app3.yaml
    ingress.extensions/singleingress3 created


Login to :red:`bigip (10.1.10.5)` via TMUI (Web UI).

Please check **[in partition kubernetes]** created services by ingress ressources

.. image:: images/ingress_vs.PNG
   :width: 800
   :alt: Ingress Service
   :align: left


We can see three services - each listening on another IP address.

Please also check the Pool & Pool Member (for service coffee).

.. image:: images/ingress_pool.PNG
   :width: 800
   :alt: Ingress Service
   :align: left


.. image:: images/ingress_poolmember.PNG
   :width: 800
   :alt: Ingress Service
   :align: left


If you check the IP addresses on the k8s cluster (kubectl describe pod *[podname]*), you'll see that f5 received POD IPs from the CIS Controller.



.. toctree::
   :maxdepth: 2
