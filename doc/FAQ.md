FAQ
====

**What are the specs of the VM?**

* CPU Cores: same as host (physical, not logical)
* 40gb HDD (auto-initialized at first boot)
* 1gb memory
* Autoboots to boot2docker
* `virtio` high performance networking
* NAT networked (Docker `4243->4243` and SSH `22->2022` are forwarded to the host)

**How can I solve my problems with SSH?**

If `ssh` complains about the keys:

```
$ ssh-keygen -R '[localhost]:2022'
```

**Login as root**

Run `sudo -s` as the docker user.

**Why not CoreOS?**

I got asked that question a lot, so I thought I should put it here once and for all. [CoreOS](http://coreos.com/) is targeted at building infrastructure and distributed systems. I just wanted the fastest way to boot to Docker.

**Persistent partition choice**

boot2docker will first try to mount a partition labeled ``boot2docker-data``, if that doesn't exist, it will pick the first ``ext4`` partition listed by ``blkid``.

**Local Customisation (with persistent partition)**

If you have a persistence partition, you can make customisations that are run at the end of the boot initialisation
in the ``/var/lib/boot2docker/bootlocal.sh`` file.

You can also set variables that will be used during the boot initialisation (after the automount) by setting them in
``/var/lib/boot2docker/profile``

For example, to download ``pipework``, install its pre-requisites (which you can download using ``tce-load -w package.tcz``), and then start a container:

```
#!/bin/sh


if [ ! -e /var/lib/boot2docker/pipework ]; then
        curl -o /var/lib/boot2docker/pipework https://raw.github.com/jpetazzo/pipework/master/pipework
        chmod 777 /var/lib/boot2docker/pipework
fi

#need ftp://ftp.nl.netbsd.org/vol/2/metalab/distributions/tinycorelinux/4.x/x86/tcz/bridge-utils.tcz
#and iproute2 (and its friends)
su - docker -c "tce-load -i /var/lib/boot2docker/*.tcz"

#start my management container if its not already there
docker run -d -v /var/run/docker.sock:/var/run/docker.sock $(which docker):$(which docker)  -name dom0 svens-dom0
```

Or, if you need to tell the Docker daemon to use a specific DNS server, add the 
following to ``/var/lib/boot2docker/profile``:

```
EXTRA_ARGS="--dns 192.168.1.2"
```

**What is the development process**

We are implementing the same process as [Docker merge approval](https://github.com/dotcloud/docker/blob/master/CONTRIBUTING.md#merge-approval), so all commits need to be done via pull requests, and will need 2 or more LGTMs.

**Is boot2docker only for VirtualBox?**

There are two parts of boot2docker: the iso image, and the boot2docker wrapper script to set up and mange a VM. The wrapper script only works with VirtualBox, but the iso image is designed to also be used with physical hardware. There are no plans to make separate iso images for different configurations.
