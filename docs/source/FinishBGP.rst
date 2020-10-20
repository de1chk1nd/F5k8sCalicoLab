.. _finish-bgp:

.. raw:: html

   <style> .red {color:red}; font-weight:bold </style>

.. role:: red


.. raw:: html

   <style> .blue {color:blue}; font-weight:bold </style>

.. role:: blue



Finish the BGP Configuration
============================

...on k8s
+++++++++

Login to :blue:`k8s master (10.1.20.20)`

Copy and paste the text below::

    cat << EOF | calicoctl create -f -
     apiVersion: projectcalico.org/v3
     kind: BGPConfiguration
     metadata:
       name: default
     spec:
       logSeverityScreen: Info
       nodeToNodeMeshEnabled: true
       asNumber: 64512
    EOF

    cat << EOF | calicoctl create -f -
    apiVersion: projectcalico.org/v3
    kind: BGPPeer
    metadata:
      name: bgppeer-global-bigip1
    spec:
      peerIP: 10.1.20.5
      asNumber: 64512
    EOF

See :download:`Example Code on github <https://github.com/de1chk1nd/F5k8sCalicoLab/blob/main/BGP/calico_bgp.txt>`


...on f5 bigip
++++++++++++++

Login to :red:`bigip (10.1.10.5)`

Copy and paste the text below::

    imish
    enable
    config terminal
    router bgp 64512
    neighbor calico-k8s peer-group
    neighbor calico-k8s remote-as 64512
    neighbor 10.1.20.20 peer-group calico-k8s
    neighbor 10.1.20.21 peer-group calico-k8s
    neighbor 10.1.20.22 peer-group calico-k8s
    write
    end

See :download:`Example Code on github <https://github.com/de1chk1nd/F5k8sCalicoLab/blob/main/BGP/f5_imish_bgp.txt>`


...check if BGP is up and running
+++++++++++++++++++++++++++++++++

Login to :blue:`k8s master (10.1.20.20)`
////////////////////////////////////////

*calicoctl get bgpPeer*::

    ubuntu@ip-10-1-1-4:~/tmp$ calicoctl get bgpPeer
    NAME                    PEERIP      NODE       ASN
    bgppeer-global-bigip1   10.1.20.5   (global)   64512


BGP Peer F5 available

Login to :red:`bigip (10.1.10.5)`
/////////////////////////////////

Login to imish and check BGP Peering and BGP Routes::

    imish
    show ip bgp neighbors



.. warning::

   Ensure that you are in "bash", when starting the imish (NOT tmsh)



Check for :red:`BGP state = Established` in the output for each member in the response::

    BGP neighbor is 10.1.20.20, remote AS 64512, local AS 64512, internal link
    Member of peer-group calico-k8s for session parameters
    BGP version 4, remote router ID 10.1.20.20
    BGP state = Established, up for 00:05:03
    Last read 00:05:03, hold time is 90, keepalive interval is 30 seconds
    [...]

Also check if routes are announced (also in imish shell)::

    show ip route bgp
    B       192.168.43.64/26 [200/0] via 10.1.20.21, internal, 00:09:30
    B       192.168.74.128/26 [200/0] via 10.1.20.22, internal, 00:09:30
    B       192.168.106.0/26 [200/0] via 10.1.20.20, internal, 00:09:30


Conclusion
++++++++++

We set up the BGP Peering between the f5 bigip and the kubernetes cluster.

In the next chapter we will dig into the calico SDN and test access to the pods.







.. toctree::
   :maxdepth: 2
