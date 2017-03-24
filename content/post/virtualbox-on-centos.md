+++
tags = []
draft = false
menu = ""
featureimage = ""
date = "2012-01-31"
title = "VirtualBox on headless CentOS 6"
categories = []

+++

Due to the need to install a server which will be used mainly for hosting virtual machines, but also be capable to provide other services, such as being FTP and Samba server, I decided to go with the <a href="http://www.centos.org/" target="_blank">CentOS Linux</a>, without having X windows and such unnecessary things installed, Oracle's <a href="https://www.virtualbox.org/" target="_blank">VirtualBox</a>, and the PHP front-end for it called <a href="https://sourceforge.net/projects/phpvirtualbox/" target="_blank">phpVirtualBox</a>.
Anyway, to make a long story short, here's how to do it.

First of, obviously, we'll need to install CentOS on the machine. The only thing that should be highlighted about this phase is to choose the _Basic Server_ installation method and let the installation process finish.

Once the installation is finished and the system is up and running the first thing that we should do is to login as root and update it:<br>
```yum -y update```<br>
After the update is finished reboot the machine to make it boot into the fresh new kernel.

Next, the regular user should be added to the system:<br>
```useradd reguser```<br>
and, to unlock newly created user account and set the password:<br>
```passwd reguser```

The following two steps aren't really necessary, but since I'm running this server inside the LAN, and don't need SELinux and firewall to be enabled I disabled all of those.

Disable SELinux by editing the ```/etc/selinux/config``` file and changing the value of ```SELINUX``` variable from ```enforcing``` to ```disabled```. This change will take effect after the machine is rebooted. To disable SELinux right away type:<br>
```setenforce 0```

Disabling iptables firewall consists of the following commands:
```
service iptables save
service iptables stop
chkconfig iptables off
service ip6tables save
service ip6tables stop
chkconfig ip6tables off
```

After all that we will need to install Apache web server, PHP and PHP SOAP extension (which is needed by the phpVirtualBox to operate properly), start Apache web server and set it to start automatically on boot (didn't need to edit the Apache config file, since the default one was just fine for me):
```
yum -y install httpd php php-soap
service httpd start
chkconfig httpd on
```

We will also need to install the _Development Tools_ group, which is a prerequisite for VirtualBox kernel modules to compile:<br>
```yum -y groupinstall "Development Tools"```

After that, we add the VirtualBox repository to the list of the repositories on our system.<br>
```wget -O /etc/yum.repos.d/virtualbox.repo http://download.virtualbox.org/virtualbox/rpm/rhel/virtualbox.repo```

Now it's time to install VirtualBox:<br>
```yum -y install VirtualBox-4.3```

Once the installation is done add the regular user to the _vboxusers_ user group:<br>
```usermod -G vboxusers reguser```

Download and unzip the latest version of the phpVirtualBox:<br>
```wget 'http://sourceforge.net/projects/phpvirtualbox/files/latest/download' -O phpvirtualbox-latest.zip
unzip phpvirtualbox-*.zip```

Then, copy the phpVirtualBox files to the Apache Documents Root directory:<br>
```cp -r phpvirtualbox-*/* /var/www/html/```

Next, copy and edit the phpVirtualBox configuration file:
```
cp /var/www/html/config.php-example /var/www/html/config.php
vim /var/www/html/config.php
```
The ```username``` and ```password``` variables within this file should be changed to reflect the values set for regular user, under whose credentials we will run VirtualBox, for example:
```
var $username = 'reguser';
var $password = 'verycomplexpass';
```

If not already present create the config file for the VirtualBox web service (```/etc/default/virtualbox```). This file should contain at least the ```VBOXWEB_USER``` variable. My system didn't have it so I just put this one variable into it:<br>
```echo "VBOXWEB_USER='reguser'" > /etc/default/virtualbox```

Now start the _vboxweb_ service and configure it to start automatically on boot:
```
service vboxweb-service start
chkconfig vboxweb-service on
```

Enable USB support in VirtualBox:
```
mkdir /vbusbfs
echo "none /vbusbfs usbfs rw,devgid=$(awk -F : '/vboxusers/ {print $3}' /etc/group),devmode=664 0 0" &gt;&gt; /etc/fstab
mount -a
```

Install the VirtualBox Extension Pack (has to be the same version as VirtualBox itself, and is required in order to be able to run the guest's console through web interface):
```
VBOXLATEST=`wget -q -O - http://download.virtualbox.org/virtualbox/LATEST.TXT` && wget http://download.virtualbox.org/virtualbox/$VBOXLATEST/Oracle_VM_VirtualBox_Extension_Pack-$VBOXLATEST.vbox-extpack && VBoxManage extpack install Oracle_VM_VirtualBox_Extension_Pack-$VBOXLATEST.vbox-extpack
```

After all that, install the custom init script (and its prerequsite package) to start and stop virtual machines automatically on startup and shutdown:
```
yum -y install redhat-lsb-core
wget /downloads/vmsctrl/vbox/vmsctrl.tar.gz
tar xvzf vmsctrl.tar.gz -C /etc/init.d/
chkconfig --add vmsctrl
```

Last, but not least, make your system automatically recompile VirtualBox kernel modules on kernel update:
```
wget /downloads/vmsctrl/vbox/VBoxCompileInstall
chmod +x VBoxCompileInstall
./VBoxCompileInstall
```

That should be it, now reboot your machine once more for everything to settle up, and once it's up connect to the phpVirtualBox interface by simply typing the IP address of your server into your web browser.<br>
Default username/password combination is admin/admin.

***IMPORTANT: All commands mentioned in this guide are to be executed as root.***
