# Single Host Vagrant
This document intends to give a instruction about how to try ovs-cni with docker/namespace in vagrant.

## Environment
You should install vagrant in your system and make sure everything goes well.

## Setup ovs-CNI
- Type `vagrant up` to init a virtual machine.
- Use ssh to connect vagrant VM via `vagrant ssh`.
- Type following command to build the `ovs-cni` binary and move it to CNI directory.
```
$ cd ~/go/src/github.com/John-Lin/ovs-cni
$ sudo cp ./bin/ovs /opt/cni/bin
```
- We need to provide a CNI config example for `ovs-cni`, and you can use build-in config from example directory. Use following command to copy the config to `~/cni` directory.

```
$ sudo cp examples/example.conf ~/cni/
```

## Create NS
In this vagrant environment, we don't install docker related services but you can use `namespace(ns)` to test `ovs-cni`.
Type following command to create a namespace named ns1

```
$ sudo ip netns add ns1
```

## Start CNI
We have setup `ovs-cni` environement and testing namespace(ns1), we can use following command to inform `ovs-cni` to add a network for the namespace.

```
$cd ~/cni
$ sudo CNI_COMMAND=ADD CNI_CONTAINERID=ns1 CNI_NETNS=/var/run/netns/ns1 CNI_IFNAME=eth2 CNI_PATH=`pwd` ./ovs <example.conf
```
and the result looks like below
```
{
    "cniVersion": "0.3.1",
        "interfaces": [
        {
            "name": "br0"
        },
        {
            "name": "veth89fa8564"
        },
        {
            "name": "eth2",
            "sandbox": "/var/run/netns/ns1"
        }
        ],
        "ips": [
        {
            "version": "4",
            "address": "10.244.1.12/16",
            "gateway": "10.244.1.1"
        }
        ],
        "routes": [
        {
            "dst": "0.0.0.0/0"
        }
        ],
        "dns": {}
}
```

Now, we can use some tools to help us check the current network setting, for example.  
You can use `ovs-vsctl show` to show current OVS setting and you it looks like  

```
2a3db108-dc16-44fd-9459-77c6fcb22279
    Bridge "br0"
        fail_mode: standalone
        Port "br0"
            Interface "br0"
                type: internal
        Port "veth656e4f8e"
            Interface "veth656e4f8e"
    ovs_version: "2.5.2"
```

In this setting, the OVS will connect to ns1 via `veth` technology and you can also check the namepsace's networking setting, you can use `sudo ip netns exec ns1 ifconfig` to see its IP config.

```
eth2      Link encap:Ethernet  HWaddr 0a:58:0a:f4:01:0a
          inet addr:10.244.1.10  Bcast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::bc15:faff:fe6b:b414/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1400  Metric:1
          RX packets:18 errors:0 dropped:0 overruns:0 frame:0
          TX packets:10 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1476 (1.4 KB)  TX bytes:828 (828.0 B)
```
