# Containerlab Basics

This workshop section introduces you to containerlab basics - topology file, image management workflows and lab lifecycle. It is loosely based on the official [Containerlab quickstart](https://containerlab.dev/quickstart/).

## Repository

Clone this repository to your workshop VM:

```bash
cd ~ && git clone https://github.com/srlinuxamericas/n93-clab.git \
&& cd n93-clab/10-basics
```

The repo should be cloned and you should be in the `n93-clab` directory as per the output below:

```
user@2:~/n93-clab/10-basics$ 
```

## Topology

The topology file `basic.clab.yml` defines the lab we are going to use in this basics exercise. It consists of the two nodes:

* Nokia SR Linux
* Cisco XRd

The nodes are interconnected with a single link over their respective first Ethernet interfaces.

```yaml
name: basic
topology:
  nodes:
    srl:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux
    xrd:
      kind: cisco_xrd
      image: xrd:7.8.1

  links:
    - endpoints: [srl:e1-1, xrd:Gi0-0-0-0]
```

## Deployment attempt #1

Before deploying the lab, make the below changes to the VM settings. This is required to run Cisco XRd nodes on your VM.

```
sudo sysctl -w fs.inotify.max_user_instances=64000
sudo sysctl -w fs.inotify.max_user_watches=64000
```

Try to deploy the lab:

```bash
sudo clab dep -t basic.clab.yml
```

Deployment fails, why?

## Image management

Check what images are available on the system:

```bash
docker images
```

If SR Linux image is not present, pull SR Linux container image available for free:

```bash
docker pull ghcr.io/nokia/srlinux
```

Import Cisco XRd image localled stored on your VM and pay attention to the 2nd argument for the `docker import` command where you have to specify the image:

```bash
docker load -i ~/images/xrd-7.8.1.tar.gz
```

Expected output:

```bash
Loaded image: registry.srlinux.dev/pub/xrd/xrd-control-plane:7.8.1
```

Then re-tag the xrd image to use a shorter tag.

```bash
docker tag registry.srlinux.dev/pub/xrd/xrd-control-plane:7.8.1 xrd:7.8.1
```

Check the local image store :

```bash
docker images
```

Expected output:

```bash
REPOSITORY                                       TAG       IMAGE ID       CREATED         SIZE
ghcr.io/nokia/srlinux                            latest    eb2a823cd8ce   4 weeks ago     2.35GB
hello-world                                      latest    d2c94e258dcb   19 months ago   13.3kB
xrd                                              7.8.1     7bb8619badbb   2 years ago     1.14GB
registry.srlinux.dev/pub/xrd/xrd-control-plane   7.8.1     7bb8619badbb   2 years ago     1.14GB
```

## Deployment attempt #2

Now that the images are available, try to deploy the lab again:

```bash
sudo clab dep -t basic.clab.yml
```

The deployment should succeed and a table will be displayed with the list of nodes and their management IPs.

```bash
╭────────────────┬───────────────────────┬─────────┬───────────────────╮
│      Name      │       Kind/Image      │  State  │   IPv4/6 Address  │
├────────────────┼───────────────────────┼─────────┼───────────────────┤
│ clab-basic-srl │ nokia_srlinux         │ running │ 172.20.20.3       │
│                │ ghcr.io/nokia/srlinux │         │ 3fff:172:20:20::3 │
├────────────────┼───────────────────────┼─────────┼───────────────────┤
│ clab-basic-xrd │ cisco_xrd             │ running │ 172.20.20.2       │
│                │ xrd:7.8.1             │         │ 3fff:172:20:20::2 │
╰────────────────┴───────────────────────┴─────────┴───────────────────╯
```

## Connecting to the nodes

Connect to the Nokia SR Linux node using the container name:

```bash
ssh clab-basic-srl
```

To logout of SR Linux node, use `CTRL+d` or type `quit`.

Connect to Cisco XRd node using its hostname or IP address. Refer to the card for password.

```bash
ssh clab@clab-basic-xrd
```

To logout of XRd node, use `exit`.

## Containerlab hosts automation

Containerlab creates `/etc/hosts` entries for each deployed lab so that you can access the nodes using their names. Check the entries:

```bash
cat /etc/hosts
```

## Listing running labs

When you are in the directory that contains the lab file, you can list the nodes of that lab simply by running:

```bash
sudo clab inspect
```

Expected output:

```bash
INFO[0000] Parsing & checking topology file: basic.clab.yml 
╭────────────────┬───────────────────────┬─────────┬───────────────────╮
│      Name      │       Kind/Image      │  State  │   IPv4/6 Address  │
├────────────────┼───────────────────────┼─────────┼───────────────────┤
│ clab-basic-srl │ nokia_srlinux         │ running │ 172.20.20.3       │
│                │ ghcr.io/nokia/srlinux │         │ 3fff:172:20:20::3 │
├────────────────┼───────────────────────┼─────────┼───────────────────┤
│ clab-basic-xrd │ cisco_xrd             │ running │ 172.20.20.2       │
│                │ xrd:7.8.1             │         │ 3fff:172:20:20::2 │
╰────────────────┴───────────────────────┴─────────┴───────────────────╯
```

If the topology file is located in a different directory, you can specify the path to the topology file:

```bash
sudo clab inspect -t ~/n93-clab/10-basics/
INFO[0000] Parsing & checking topology file: basic.clab.yml 
╭────────────────┬───────────────────────┬─────────┬───────────────────╮
│      Name      │       Kind/Image      │  State  │   IPv4/6 Address  │
├────────────────┼───────────────────────┼─────────┼───────────────────┤
│ clab-basic-srl │ nokia_srlinux         │ running │ 172.20.20.3       │
│                │ ghcr.io/nokia/srlinux │         │ 3fff:172:20:20::3 │
├────────────────┼───────────────────────┼─────────┼───────────────────┤
│ clab-basic-xrd │ cisco_xrd             │ running │ 172.20.20.2       │
│                │ xrd:7.8.1             │         │ 3fff:172:20:20::2 │
╰────────────────┴───────────────────────┴─────────┴───────────────────╯+
```

You can also list all running labs regardless of where their topology files are located:

```bash
sudo clab inspect --all
```

The output will contain all labs and their nodes.

Shortcuts:

* `sudo clab ins` == `sudo containerlab inspect`
* `sudo clab ins -a` == `sudo containerlab inspect --all`

## Lab directory

Lab directory stores the artifacts generated by containerlab that are related to the lab:

* tls certificates
* startup configurations
* inventory files
* topology export json file
* bind bounted directories

To list the contents of the lab directory, run:

```bash
tree -L 3 clab-basic/
```

## Destroying the lab

When you are done with the lab, you can destroy it:

```bash
sudo clab destroy -t basic.clab.yml
```

You have now finished the basics lab exercise!
