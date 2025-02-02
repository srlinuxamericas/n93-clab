# Containerlab Installation

## All In One installer

Installs

* docker
* latest containerlab,
* `gh` cli

all in one, multi-OS installer:

```bash
curl -L http://containerlab.dev/setup | \
sudo bash -s "all"
```

The automation script adds the `docker` group to your `user`, in order for these changes to take effect, log out from the current session and log back in.

After logout and login again, check that docker is installed and running:

```bash
docker run --rm hello-world
```

Expected output: `Hello from Docker!`

Check that containerlab is installed successfully:

```bash
containerlab version
```

* Alternative Containerlab installation options are available [here](https://containerlab.dev/install/).
* Alternative Docker installation options can be found [here](https://docs.docker.com/engine/install/).
