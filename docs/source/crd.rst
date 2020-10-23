.. _finish-bgp:

.. raw:: html

   <style> .red {color:red}; font-weight:bold </style>

.. role:: red


.. raw:: html

   <style> .blue {color:blue}; font-weight:bold </style>

.. role:: blue


Custom Ressource Definition
===========================

Preperation
-----------

Please check, that ALL **ingress** Ressources are deleted properly::

   ubuntu@ip-10-1-1-4:~$ kubectl get ingress
   No resources found.


If ingress services are showing up, delete them with *kubectl delete ingress <ingress name>*::

   ubuntu@ip-10-1-1-4:~/k8s/ingress$ kubectl delete ingress singleingress1
   NAME             HOSTS   ADDRESS   PORTS   AGE
   singleingress1   *                 80      7s
   ubuntu@ip-10-1-1-4:~$ kubectl delete ingress singleingress1


Delete old Ingress Controller::

   kubectl delete -f /home/ubuntu/k8s/cis/002_setup_cis_bigip.yaml


What is a custom Ressource Definition
-------------------------------------

Definition from clouddocs.f5.com:

.. admonition:: What is a custom Ressource...

   * Custom resources are extensions of the Kubernetes API
   * A resource is an endpoint in the Kubernetes API that stores a collection of API objects. For example, the built-in pods resource contains a collection of Pod objects
   * A custom resource is an extension of the Kubernetes API that is not necessarily available in a default Kubernetes installation. It represents a customization of a particular Kubernetes installation. However, many core Kubernetes functions are now built using custom resources, making Kubernetes more modular
   * Custom resources can appear and disappear in a running cluster through dynamic registration, and cluster admins can update custom resources independently of the cluster itself. Once a custom resource is installed, users can create and access its objects using kubectl, just as they do for built-in resources like Pods
   * CIS supports the following Custom Resources:
      * VirtualServer (VirtualServer resource defines load balancing configuration for a **domain name**)
      * TLSProfile (TLSProfile is used to specify the TLS termination for a single/list of services in a VirtualServer Custom Resource. **TLS termination relies on SNI**)
      * TransportServer (The TransportServer resource exposes the **non-HTTP traffic** configuration for a virtual server address in BIG-IP)

   For an overview about supported parameters and a link to the schema file, please check here `here <https://clouddocs.f5.com/containers/latest/userguide/crd.html#id1>`_


.. warning::

   CRD are not working with config maps and Ingress - see notes below from clouddocs.f5.com:

   * --custom-resource-mode=true deploys CIS in Custom Resource Mode.
   
   * CIS does not watch for Ingress/Routes/ConfigMaps when deployed in CRD Mode.
   
   * CIS does not support combination of CRDs with any of Ingress/Routes and ConfigMaps.


Installation of Custom Ressource in k8s
---------------------------------------

Change folder to /home/ubuntu/k8s/crd and take a look at the config file. Deploy the config::

   ubuntu@ip-10-1-1-4:~/k8s/crd$ kubectl apply -f 001_customresourcedefinitions.yml
   customresourcedefinition.apiextensions.k8s.io/virtualservers.cis.f5.com created
   customresourcedefinition.apiextensions.k8s.io/tlsprofiles.cis.f5.com created
   customresourcedefinition.apiextensions.k8s.io/transportservers.cis.f5.com created


.. warning::

   **Double-Chek, that the old POD is not running (deployment deleted in Step 1 of this CRD Guide)**

   | ubuntu@ip-10-1-1-4:~/k8s/crd$ kubectl get pods -n kube-system
   | NAME                                       READY   STATUS    RESTARTS   AGE
   | calico-kube-controllers-7567d8d9dd-m9zt2   1/1     Running   2          3h
   | calico-node-8lst4                          1/1     Running   2          3h
   | calico-node-stpvd                          1/1     Running   2          178m
   | calico-node-wlkwb                          1/1     Running   2          178m
   | coredns-66bff467f8-s6v8r                   1/1     Running   2          3h7m
   | coredns-66bff467f8-z8896                   1/1     Running   2          3h7m
   | etcd-ip-10-1-1-4                           1/1     Running   2          3h7m
   | kube-apiserver-ip-10-1-1-4                 1/1     Running   2          3h7m
   | kube-controller-manager-ip-10-1-1-4        1/1     Running   2          3h7m
   | kube-proxy-269bk                           1/1     Running   2          178m
   | kube-proxy-cljb4                           1/1     Running   2          178m
   | kube-proxy-n9w8z                           1/1     Running   2          3h7m
   | kube-scheduler-ip-10-1-1-4                 1/1     Running   2          3h7m


Deploy the contoller in CRD mode::

   ubuntu@ip-10-1-1-4:~/k8s/crd$ kubectl apply -f 002_crd_cis.yml
   deployment.apps/k8s-bigip1-ctlr-deployment created


As described in the previous chapter, controller will be deplyoed with **--custom-resource-mode=true**

See :download:`Example Code on github <https://github.com/de1chk1nd/F5k8sCalicoLab/blob/main/k8s/crd/002_crd_cis.yml>`


Now we can start deplying CRD services.


Simple HTTP Service
-------------------

Service Deplyoment::

   apiVersion: "cis.f5.com/v1"
   kind: VirtualServer
   metadata:
   name: tea-virtual-server
   labels:
      f5cr: "true"
   spec:
   # This is an insecure virtual, Please use TLSProfile to secure the virtual
   # check out tls examples to understand more.
   virtualServerAddress: "10.1.10.92"
   host: cafe.de1chk1nd.de
   pools:
   - path: /coffee
      service: coffee-svc
      servicePort: 80
   - path: /tea
      service: tea-svc
      servicePort: 80

See :download:`Example Code on github <https://github.com/de1chk1nd/F5k8sCalicoLab/blob/main/k8s/crd/003a_crd_simple_http.yaml>`




.. toctree::
   :maxdepth: 2
