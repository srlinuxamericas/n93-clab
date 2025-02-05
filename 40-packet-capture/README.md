# Packet capture

Every lab emulation software must provide its users with the packet capturing abilities. Looking at the frames as they traverse the network links is not only educational, but also helps to troubleshoot the issues that might arise during the lab development.

Containerlab offers a simple way to capture the packets from any interface of any node in the lab since every interface is exposed to the underlying Linux OS. This article will explain how to do that.

Everything we are going to do in this exercise is explained in details in the [Containerlab documentation](https://containerlab.dev/manual/wireshark/).

**Note: We will be using the [startup lab](../15-startup) for this section. If this is already destroyed, please re-deploy it before continuing. Once the lab is deployed, login to the SR Linux node and start a ping using**

```srl
ping 192.168.1.1 network-instance default
```

## Local capture

To capture the packets on the local host VM directly, run the below command on the VM host. This captures packets on the SR Linux interface `ethernet-1/1 `.

```bash
sudo ip netns exec clab-startup-srl tcpdump -nni e1-1
```

The output is displayed directly on the screen after the capture is stopped using CTRL+c.

```bash
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on e1-1, link-type EN10MB (Ethernet), snapshot length 262144 bytes
^C21:41:34.519518 IP 192.168.1.2.53896 > 192.168.1.1.179: Flags [P.], seq 585107004:585107023, ack 625985616, win 502, options [nop,nop,TS val 3540017866 ecr 2310269600], length 19: BGP
21:41:34.521095 IP 192.168.1.1.179 > 192.168.1.2.53896: Flags [.], ack 19, win 31290, options [nop,nop,TS val 2310287756 ecr 3540017866], length 0
21:41:36.022378 IP 10.10.10.1 > 10.10.10.2: ICMP echo request, id 25939, seq 1, length 64
21:41:36.022410 IP 10.10.10.2 > 10.10.10.1: ICMP echo reply, id 25939, seq 1, length 64
21:41:37.024267 IP 10.10.10.1 > 10.10.10.2: ICMP echo request, id 25939, seq 2, length 64
21:41:37.024305 IP 10.10.10.2 > 10.10.10.1: ICMP echo reply, id 25939, seq 2, length 64
21:41:37.486390 LLDP, length 146: srl
21:41:38.026241 IP 10.10.10.1 > 10.10.10.2: ICMP echo request, id 25939, seq 3, length 64
21:41:38.026275 IP 10.10.10.2 > 10.10.10.1: ICMP echo reply, id 25939, seq 3, length 64

9 packets captured
9 packets received by filter
0 packets dropped by kernel
```


## Remote capture

To capture the packets from a containerlab node running on a remote host that we are going to explore is called "remote capture". In this scenario a user has a network connectivity (ssh) to the host that runs containerlab topology and wishes to get the packet capture displayed in the Wireshark running locally.

To achieve this, we will execute the `tcpdump` command on the remote host and pipe the output to the local Wireshark app. Here is a command that does it all.

It captures the traffic from SR Linux (`clab-startup-srl`) port `ethernet-1/1` running on your host and displaying the capture in the Wireshark.

Login to `clab-startup-srl` and initiate a ping to `192.168.1.1`:

```srl
ping 192.168.1.1 network-instance default
```

<small>The command is provided for WSL and Mac systems, assuming default Wireshark installation path. Replace `X` with your VM number.</small>

Windows/WSL:

```bash
ssh nanoguser@<X>.wrkshpz.net \
"sudo ip netns exec clab-startup-srl tcpdump -U -nni e1-1 -w -" | \
/mnt/c/Program\ Files/Wireshark/wireshark.exe -k -i -
```

macOS:

```bash
ssh nanoguser@<X>.wrkshpz.net \
"sudo ip netns exec clab-startup-srl tcpdump -U -nni e1-1 -w -" | \
/Applications/Wireshark.app/Contents/MacOS/Wireshark  -k -i -
```

## Edgeshark

[Edgeshark](https://edgeshark.siemens.io/#/) is a set of tools that offer (among many things) a Web UI that displays every interface of every container and can start a wireshark as easy as clicking a button.

Edgeshark installation consists of two parts:

1. A service that runs on the host that runs containerlab topologies
2. A wireshark capture plugin that runs next to the Wireshark on a user's PC

To install the service, paste the installer command that uses docker compose to deploy the service:

```bash
curl -sL \
https://github.com/siemens/edgeshark/raw/main/deployments/wget/docker-compose.yaml | \
docker compose -f - up -d
```

Now, you have to install the client plugin based on the OS of your PC.

### Windows

Windows users get to enjoy a simple installer-based workflow that installs the URL handler and the Wireshark plugin in one go.

Download the installer file - <https://github.com/siemens/cshargextcap/releases/download/v0.10.7/cshargextcap_0.10.7_windows_amd64.zip>

Unzip the archive and launch the installer script.

### MacOs

MacOs users have to suffer a little. But it is not that bad either.

To install the URL handler paste the following in the Mac terminal app:

```bash
mkdir -p /tmp/pflix-handler && cd /tmp/pflix-handler && \
rm -rf packetflix-handler.zip packetflix-handler.app __MACOSX && \
curl -sLO https://github.com/srl-labs/containerlab/files/14278951/packetflix-handler.zip && \
unzip packetflix-handler.zip && \
sudo mv packetflix-handler.app /Applications
```

To install the extpcap wireshark plugin execute in the Mac terminal:

```bash
# for x86_64 MacOS use https://github.com/siemens/cshargextcap/releases/download/v0.10.7/cshargextcap_0.10.7_darwin_amd64.tar.gz
DOWNLOAD_URL=https://github.com/siemens/cshargextcap/releases/download/v0.10.7/cshargextcap_0.10.7_darwin_arm64.tar.gz
mkdir -p /tmp/pflix-handler && curl -sL $DOWNLOAD_URL | tar -xz -C /tmp/pflix-handler && \
open /tmp/pflix-handler && open /Applications/Wireshark.app/Contents/MacOS/extcap
```

The command above will open two Finder windows, one with the `cshargextcap` binary and the other with the Wireshark's existing plugins. Move the `cshargextcap` file over to the window with Wireshark plugins.

### Web UI

To access the Edgeshark UI, open a browser and navigate to the following URL (substitute {ID} with your assigned VM):

<http://{ID}.wrkshpz.net:5001>

Note, the http schema is important, since https is not enabled.

To startup a packet capture from Edgeshark UI, click on the `wireshark` icon next to `e1-1` interface under clab-startup-srl. Ensure you ping is still running. This will open Wireshark on your PC and display the ICMP packets.
