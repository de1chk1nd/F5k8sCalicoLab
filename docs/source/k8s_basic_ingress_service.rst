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




.. toctree::
   :maxdepth: 2
