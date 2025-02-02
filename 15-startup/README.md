# Startup configuration

This exercise demonstrates how to provide startup configuration to the lab nodes by means of the [`startup-config`](https://containerlab.dev/manual/nodes/#startup-config) node parameter in the topology file.

Startup configuration is a way to provide initial configuration to the lab nodes when they boot up. This is useful when you want to automate the configuration of the nodes and avoid manual intervention. It also brings your lab to a desired state when you need to test a specific scenario.

To enter the lab directory, run the following command from anywhere in your terminal:

```bash
cd ~/n93-clab/15-startup/
```

We start by deploying a lab defined in the [startup.clab.yml](startup.clab.yml) topology file. The lab consists of two nodes: `srl` (Nokia SR Linux) and `xrd` (Cisco XRd). Both nodes are configured with a startup configuration file that resides in the same directory as the topology file.

We will use the shortened syntax when deploying the lab; less typing and more fun!

```bash
sudo clab dep -c
```

> Note, that when calling `clab dep -c` the containerlab will try to find the `*.clab.yml` file in the current working directory. If the file is located elsewhere, you can specify the path to the file as an argument to the `clab dep` command.  
> The `-c` flag stands for `--cleanup` and it will ensure that if the lab is already running, it will be stopped and removed before deploying a new one.

The startup configuration files - [srl.cfg](srl.cfg) and [xrd.cfg](xrd.cfg) - configure the interfaces, IP addressing and ISIS adjacency between SR Linux and Cisco XRd nodes respectively.

Expected output:

```bash
╭──────────────────┬───────────────────────┬─────────┬───────────────────╮
│       Name       │       Kind/Image      │  State  │   IPv4/6 Address  │
├──────────────────┼───────────────────────┼─────────┼───────────────────┤
│ clab-startup-srl │ nokia_srlinux         │ running │ 172.20.20.2       │
│                  │ ghcr.io/nokia/srlinux │         │ 3fff:172:20:20::2 │
├──────────────────┼───────────────────────┼─────────┼───────────────────┤
│ clab-startup-xrd │ cisco_xrd             │ running │ 172.20.20.3       │
│                  │ xrd:7.8.1             │         │ 3fff:172:20:20::3 │
╰──────────────────┴───────────────────────┴─────────┴───────────────────╯
```

After the lab is deployed, we can expect that the nodes will boot up and apply the startup configuration snippets provided in the topology file. Consequently, it is fair to assume that the nodes will establish ISIS adjacency between them.

Let's connect to the `clab-startup-srl` node and ping the remote interface address on the xrd node:

```bash
ssh clab-startup-srl
```

```srl
ping 192.168.1.1 network-instance default -c 3
```
If ping fails, check the xrd node boot logs using `docker logs -f clab-startup-xrd` and make sure that the interfaces have come up.

Expected output:

```srl
Using network instance default
PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
64 bytes from 192.168.1.1: icmp_seq=1 ttl=255 time=8.61 ms
64 bytes from 192.168.1.1: icmp_seq=2 ttl=255 time=6.75 ms
64 bytes from 192.168.1.1: icmp_seq=3 ttl=255 time=8.75 ms

--- 192.168.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 6.754/8.039/8.754/0.910 ms
```

Now, let's verify if the IS-IS adjacency has been established between srl and xrd nodes.

```srl
show network-instance default protocols isis adjacency
```

Expected output:

```srl
----------------------------------------------------------------------------------------------------------------------------------------------------------------
Network Instance: default
Instance        : 1
Instance Id     : 0
+----------------+--------------------+-----------------+-------------+--------------+-------+--------------------------+--------------------+
| Interface Name | Neighbor System Id | Adjacency Level | Ip Address  | Ipv6 Address | State |     Last transition      | Remaining holdtime |
+================+====================+=================+=============+==============+=======+==========================+====================+
| ethernet-1/1.0 | 0020.0200.2002     | L1L2            | 192.168.1.1 | ::           | up    | 2024-12-05T22:45:50.600Z | 21                 |
+----------------+--------------------+-----------------+-------------+--------------+-------+--------------------------+--------------------+
Adjacency Count: 1
----------------------------------------------------------------------------------------------------------------------------------------------------------------
```

We can see that the IS-IS adjacency has been established.

You have successfully deployed the lab with the nodes equipped with the startup configuration. This is a powerful feature that can be used to provision the nodes with the desired configuration when they boot up.
