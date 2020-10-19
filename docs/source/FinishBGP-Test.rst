.. _finish-bgp-test:

.. raw:: html

   <style> .red {color:red}; font-weight:bold </style>

.. role:: red


.. raw:: html

   <style> .blue {color:blue}; font-weight:bold </style>

.. role:: blue

Testing the SDN
===============

Interpretation of bgp Routes
----------------------------

Please remember the BGP Route output from the previous lab::

    show ip route bgp
    B       192.168.43.64/26 [200/0] via 10.1.20.21, internal, 00:09:30
    B       192.168.74.128/26 [200/0] via 10.1.20.22, internal, 00:09:30
    B       192.168.106.0/26 [200/0] via 10.1.20.20, internal, 00:09:30

We have 3 routes announced via BGP:

* 192.168.106.0/26 via 10.1.20.20 :red:`(kubernetes master | 10.1.20.20)`

* 192.168.43.64/26 via 10.1.20.21 :red:`(kubernetes worker I | 10.1.20.21)`

* 192.168.74.128/26 via 10.1.20.22 :red:`(kubernetes worker II | 10.1.20.22)`

Each k8s node is running a certain amount of pods - these pods have an 192.168.*.* IP assigned.
Let's check this for on particular pod.

Login to :blue:`k8s master (10.1.20.20)`::

    ubuntu@ip-10-1-1-4:~$ kubectl get pods
    NAME                                   READY   STATUS    RESTARTS   AGE
    coffee-2-76f45cdb7b-qbfkr              1/1     Running   5          158d
    coffee-2-76f45cdb7b-r2zcb              1/1     Running   5          158d
    coffee-6567c98884-54qnd                1/1     Running   6          159d
    coffee-6567c98884-xngq9                1/1     Running   6          159d
    f5-hello-world-2-69bddf75fc-7bf5s      1/1     Running   5          158d
    f5-hello-world-2-69bddf75fc-ktxxh      1/1     Running   5          158d
    f5-hello-world-847698f5c6-jnk7b        1/1     Running   6          159d
    f5-hello-world-847698f5c6-wknk4        1/1     Running   6          159d
    f5-hello-world-web1-56f5ff778d-9hbxk   1/1     Running   5          158d
    f5-hello-world-web1-56f5ff778d-m7cdf   1/1     Running   5          158d
    f5-hello-world-web2-86c88c8685-gck6l   1/1     Running   5          158d
    f5-hello-world-web2-86c88c8685-vnv74   1/1     Running   5          158d
    tea-2-f55bc49c4-s6znp                  1/1     Running   5          158d
    tea-2-f55bc49c4-shwrz                  1/1     Running   5          158d
    tea-b7b558b96-25t4j                    1/1     Running   6          159d
    tea-b7b558b96-vrjnr                    1/1     Running   6          159d

This will list all pods in the *default namespace*.

Now check the pod details::

    ubuntu@ip-10-1-1-4:~$ kubectl describe pod coffee-2-76f45cdb7b-qbfkr
    Name:               coffee-2-76f45cdb7b-qbfkr
    Namespace:          default
    Priority:           0
    PriorityClassName:  <none>
    Node:               ip-10-1-1-6/10.1.1.6
    Start Time:         Thu, 14 May 2020 11:11:27 +0000
    Labels:             app=coffee-2
                        pod-template-hash=76f45cdb7b
    Annotations:        cni.projectcalico.org/podIP: 192.168.74.168/32
    Status:             Running
    IP:                 192.168.74.168
    Controlled By:      ReplicaSet/coffee-2-76f45cdb7b
    Containers:
      coffee-2:
        Container ID:   docker://9368b2611be99ebaa5235bbdec2a01407785135fd8f3ae132a29d1684d4db299
        Image:          nginxdemos/nginx-hello:plain-text
        Image ID:       docker-pullable://nginxdemos/nginx-hello@sha256:71b4a63b6d31ae846c431d0dc7134bc0ae490fe4698eebf3ed297668db3b2939
        Port:           8080/TCP
        Host Port:      0/TCP
        State:          Running
          Started:      Mon, 19 Oct 2020 17:41:18 +0000
        Last State:     Terminated
          Reason:       Completed
          Exit Code:    0
          Started:      Thu, 09 Jul 2020 19:57:50 +0000
          Finished:     Thu, 09 Jul 2020 20:31:18 +0000
        Ready:          True
        Restart Count:  5
        Environment:    <none>
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-qk8c5 (ro)
    Conditions:
      Type              Status
      Initialized       True
      Ready             True
      ContainersReady   True
      PodScheduled      True
    Volumes:
      default-token-qk8c5:
        Type:        Secret (a volume populated by a Secret)
        SecretName:  default-token-qk8c5
        Optional:    false
    QoS Class:       BestEffort
    Node-Selectors:  <none>
    Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                     node.kubernetes.io/unreachable:NoExecute for 300s
    Events:          <none>

**Important** part is of this output is:

* Node:               ip-10-1-1-6/10.1.1.6
* IP:                 192.168.74.168

Node equals the k8s Node; IP is the POD IP Address.
As this output shows the MGMT Address, you have to know following table:


========  ========  ==========
 Device    IP MGMT   IP Prod
========  ========  ==========
 master   10.1.1.4  10.1.20.20
worker-1  10.1.1.5  10.1.20.21
worker-2  10.1.1.6  10.1.20.22
========  ========  ==========


Mapping all the information above:

1. POD 192.168.74.168 runs on **worker-2** (IP MGMT 10.1.1.6)
2. BGP Route points 192.168.74.128/26 to worker-II  (IP PROD 10.1.20.22)


We can test direct POD access from :red:`bigip`, to verify the above information.

Login to :red:`bigip (10.1.10.5)`::

    [admin@ip-10-1-1-7:Active:Standalone] ~ # ping 192.168.74.168
    PING 192.168.74.168 (192.168.74.168) 56(84) bytes of data.
    64 bytes from 192.168.74.168: icmp_seq=1 ttl=63 time=0.569 ms
    64 bytes from 192.168.74.168: icmp_seq=2 ttl=63 time=0.650 ms
    64 bytes from 192.168.74.168: icmp_seq=3 ttl=63 time=0.383 ms
    64 bytes from 192.168.74.168: icmp_seq=4 ttl=63 time=0.580 ms
    ^C
    --- 192.168.74.168 ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 3000ms
    rtt min/avg/max/mdev = 0.383/0.545/0.650/0.101 ms

    [admin@ip-10-1-1-7:Active:Standalone] ~ # curl http://192.168.74.168:8080
    Server address: 192.168.74.168:8080
    Server name: coffee-2-76f45cdb7b-qbfkr
    Date: 19/Oct/2020:21:27:44 +0000
    URI: /
    Request ID: cfae2478ceb7e7fef28d1040c380fa13


Although F5 Container Ingress Service is not yet deployed, we already have access to the PODs. We could go ahead, and manually define Services (Pool Member = POD IPs). But since k8s is dynamic, we want to automatically receive service information. This will be covered in the next chapter *basic ingress*.

We will still continue examining the current set up, to understand the calico set up for now.


Calico SDN
----------

.. admonition:: For the *really* impatient ...

   If you skipped the tigera video about the calico installation - and might miss something in the part below, I recommend to check the video again, as this explains the architecture very well and is helpfull.

Calico SDN is based on routing - more or less.

1. PODs have a default route to their interface
2. Calico creates tunnel interfaces to *"link"* PODs to the Host Network
3. To be able to route traffic to specific hosts, local routes are Created
4. To use PODs on *"external"* nodes (e.g. worker-I to worker-II), Host needs a route to point to the other worker node
5. To automate "4", BGP is used (dynamic update of *"external"* routes)

Let take the example from the previous chapter: **pod coffee-2-76f45cdb7b-qbfkr** (IP: 192.168.74.168)

Login to :blue:`k8s worker-II (10.1.20.22)`::

    ubuntu@ip-10-1-1-6:~$ netstat -rn
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
    0.0.0.0         10.1.1.1        0.0.0.0         UG        0 0          0 eth0
    10.1.1.0        0.0.0.0         255.255.255.0   U         0 0          0 eth0
    10.1.1.1        0.0.0.0         255.255.255.255 UH        0 0          0 eth0
    10.1.20.0       0.0.0.0         255.255.255.0   U         0 0          0 eth1
    172.17.0.0      0.0.0.0         255.255.0.0     U         0 0          0 docker0
    192.168.43.64   10.1.20.21      255.255.255.192 UG        0 0          0 tunl0
    192.168.74.128  0.0.0.0         255.255.255.192 U         0 0          0 *
    192.168.74.167  0.0.0.0         255.255.255.255 UH        0 0          0 caliebba1cc5ca6
    192.168.74.168  0.0.0.0         255.255.255.255 UH        0 0          0 cali29ae8c33ae7
    192.168.74.169  0.0.0.0         255.255.255.255 UH        0 0          0 cali8a95b88fef1
    192.168.74.170  0.0.0.0         255.255.255.255 UH        0 0          0 cali0f22605673e
    192.168.74.171  0.0.0.0         255.255.255.255 UH        0 0          0 cali7298375bc35
    192.168.74.172  0.0.0.0         255.255.255.255 UH        0 0          0 calia648f6ad6f3
    192.168.74.173  0.0.0.0         255.255.255.255 UH        0 0          0 calid5170f9f77a
    192.168.106.0   10.1.20.20      255.255.255.192 UG        0 0          0 tunl0


We can see a route, pointing to tunnel interface **cali29ae8c33ae7**::

    ubuntu@ip-10-1-1-6:~$ ifconfig cali29ae8c33ae7
    cali29ae8c33ae7: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1440
            inet6 fe80::ecee:eeff:feee:eeee  prefixlen 64  scopeid 0x20<link>
            ether ee:ee:ee:ee:ee:ee  txqueuelen 0  (Ethernet)
            RX packets 12  bytes 1206 (1.2 KB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 32  bytes 2403 (2.4 KB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

Let's assume a traffic flow from :red:`bigip`:

1. curl to http://192.168.74.168:8080
2. :red:`bigip` looks up routing table and finds the announced bgp route to :blue:`k8s worker-II`
3. :blue:`k8s worker-II` receives traffic from the f5 - and finds a route to local tunnel interface
4. return traffic leaving the POD (through tunnel interface), will be sent back to f5 (either because of SNAT on :red:`bigip` or via default route)







.. toctree::
   :maxdepth: 2
