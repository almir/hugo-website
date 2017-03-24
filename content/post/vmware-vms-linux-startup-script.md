+++
title = "VMware VMs Linux startup script"
menu = ""
featureimage = ""
categories = []
tags = []
draft = false
date = "2011-03-14"

+++

Once the hardware on my home PC got upgraded I decided to run a couple of virtual machines on it at all times.

I've chosen VMware Workstation for the job, considering that, as a VCP, I have a license for it. Underneath it is Ubuntu Linux 11.04 64-bit.

Now, because I like it all to be automatic I've written this little script which uses the ```vmrun``` command to start and stop my virtual machines automatically during the boot-up and shutdown.

Recently, a <a href="https://launchpad.net/~almirdz/+archive/ppa" target="_blank">Launchpad PPA</a> containing Ubuntu packages has been created.

The packages have been tested to work on 32 and 64-bit **Ubuntu 10.04 (Lucid)**, **Ubuntu 10.10 (Maverick)** and** Ubuntu 11.04 (Natty)**.

They should also work on Ubuntu 11.10 (Oneiric), but that one hasn't really been tested yet.

Installation requires a few steps:

1 . Add the repositories and the PGP key to your system:

To add the repositories and the PGP key automatically type

```sudo add-apt-repository -y ppa:almirdz/ppa```

followed by

```sudo apt-get update```

to refresh the local packages list.

To manually add the repositories do the following:

```sudo vim /etc/apt/sources.list.d/almirdz-ppa.list```

and put the links to the repositories for your Ubuntu version, eg.:

```deb http://ppa.launchpad.net/almirdz/ppa/ubuntu natty main
deb-src http://ppa.launchpad.net/almirdz/ppa/ubuntu natty main
```

Then, add the PGP key to the list of trusted keys:

```sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 3604082A```

After all is done run

```sudo apt-get update```

to refresh the local packages list.

2. Install the package by simply typing:

```sudo apt-get -y install vmsctrl```

That's it, enjoy! ;-)

***IMPORTANT***: The path where virtual machines are stored, or the name of the virtual machine itself, **must not** contain spaces in order for script to work properly. Also, the path to the ```vmrun``` **must** be configured within your environment variables.

For any suggestions and bug reports please <a href="https://github.com/almir/vmsctrl/issues/new" target="_blank">create an issue on GitHub</a>.
