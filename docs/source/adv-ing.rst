.. _finish-bgp-test:

.. raw:: html

   <style> .red {color:red}; font-weight:bold </style>

.. role:: red


.. raw:: html

   <style> .blue {color:blue}; font-weight:bold </style>

.. role:: blue



TLS Operations
==============

Now that we understand how to apply simple http services, I want to showcase two different ways to attach SSL Profiles.

.. warning::
   Beginning with CIS 2.0, we support several SSL profiles on one resource. Before it was a 1:1 mapping.


TLS with local clientssl profile
--------------------------------

First Option is to use a certificate, already hosted on the bigip.

**Ingress Config File / EXAMPLE:**::

    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: singleingress7
      namespace: default
      annotations:
        # Provide an IP address for the BIG-IP pool you want to handle traffic.
        virtual-server.f5.com/ip: "10.1.10.90"
        # Specify the BIG-IP partition containing the virtual server.
        virtual-server.f5.com/partition: "kubernetes"
        # Allow/deny TLS connections
        ingress.kubernetes.io/ssl-redirect: "true"
        # Allow/deny HTTP connections
        ingress.kubernetes.io/allow-http: "false"
        # Uncomment below annotation, to attach Server SSL Profile (local SSL profile)
        # virtual-server.f5.com/serverssl: "/Common/serverssl"
        # Custom monitor
        virtual-server.f5.com/health: '[{"path": "/", "send": "HTTP GET /", "interval": 5, "timeout": 10}]'
    spec:
      tls:
        # Specifies an already-configured SSL Profile on BIG-IP that should be
        # used for this Ingress.
        # Follows the format "/partition/profile_name".
        - secretName: /Common/clientssl
      backend:
        # The name of a single Kubernetes Service you want to expose to external
        # traffic using TLS
        serviceName: coffee-svc
        servicePort: 80

See :download:`Example Code on github <https://github.com/de1chk1nd/F5k8sCalicoLab/blob/main/k8s/ingress/004a_adv-ingress-1.yaml>`


This creates a simple SSL Service with ssl redirect on port 80. Uncommenting "virtual-server.f5.com/serverssl" will allow the f5 to re-encrypt the service.

With **tls: - secretName: /path-to/profile** we define the local SSL Profile to use for decryption.


TLS with k8s secret stored certificate
--------------------------------------

Second Option is to store certificate information within the k8s cluster as secret ... and pull the information from that location to create a profile on the bigip.

**Ingress Config File / EXAMPLE:**::

    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: singleingress8
      namespace: default
      annotations:
        # Provide an IP address for the BIG-IP pool you want to handle traffic.
        virtual-server.f5.com/ip: "10.1.10.91"
        # Specify the BIG-IP partition containing the virtual server.
        virtual-server.f5.com/partition: "kubernetes"
        # Allow/deny TLS connections
        ingress.kubernetes.io/ssl-redirect: "false"
        # Allow/deny HTTP connections
        ingress.kubernetes.io/allow-http: "true"
        # Uncomment below annotation, to attach Server SSL Profile (local SSL profile)
        # virtual-server.f5.com/serverssl: "/Common/serverssl"
        # Custom monitor
        virtual-server.f5.com/health: '[{"path": "/", "send": "HTTP GET /", "interval": 5, "timeout": 10}]'

    spec:
      tls:
      - hosts:
        - mysite.foo.com
        #Referencing this secret in an Ingress tells the Ingress controller to
        #secure the channel from the client to the load balancer using TLS
        secretName: ingress-example-secret-tls
      rules:
        - host: mysite.example.com
          http:
          # path to Service from URL
            paths:
            - path: /
              backend:
                serviceName: coffee-svc
                servicePort: 80

See :download:`Example Code on github <https://github.com/de1chk1nd/F5k8sCalicoLab/blob/main/k8s/ingress/004c_adv-ingress-1.yaml>`

This creates a similar service, as seen before. This time we just specify a name **tls: secretName: ingress-example-secret-tls**

.. admonition:: secret vs. local profile

   CIS controller will **always** lookup local secret store, before falling back to a local profile (but this is a configurable setting).

   In this example, CIS controller finds "ingress-example-secret-tls" in the local store and creates neccessary TLS profiles on f5.

   In the previous example, CIS controller couldn't find /Common/clientssl in the local store - and assumes this will be a local bigip profile.


So we have to provide a local secret::

    apiVersion: v1
    kind: Secret
    metadata:
      name: ingress-example-secret-tls
      namespace: default
    data:
      tls.crt: <base64 encoded cert>
      tls.key: <base64 encoded key>
    type: kubernetes.io/tls

See :download:`Example Code on github <https://github.com/de1chk1nd/F5k8sCalicoLab/blob/main/k8s/ingress/004b_secret_cert.yaml>`



.. toctree::
   :maxdepth: 2
