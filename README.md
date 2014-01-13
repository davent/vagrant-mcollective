Vagrant built mCollective Test/Dev Environment
==============================================
### Using Packer, Vagrant, Chef, RabbitMQ & mCollective
---

I recenly started a project to automate the execution of a huge collection of scripts across servers in different hosting environments, from Amazon AWS to Development VMs on interally hosted Xen servers, and to allow them to be executed by anyone in the business from any location.

I decided to write some automation tools based on the [mCollective](http://puppetlabs.com/mcollective) framework to allow me to be able to retrieve information and execute commands no matter where they were (as long as they could all talk to the same message queue of course).

To write and test these tools I wanted a developemnt environment which matched the production environment as closely as possible and therefore decided to build such an environment using the applications mentioned above:

* Packer - To create the Operating System images
* Vagrant - To create virtual servers using the Operating System images
* Chef - To configure the virtual servers and install required software
* RabbitMQ - The message queue between all the server nodes
* mCollective - The framework to build the automation tools on

I have included everything required to achieve this development and testing environment in this repository and below is how to use it.

---

### Prerequisites

You will need:

* VirtualBox (tested with 4.2.12)
* [Vagrant](http://www.vagrantup.com/downloads.html) (tested on 1.4.1)
* [Packer](http://www.packer.io/downloads.html) (tested on 0.5.1)

Once you have installed VirtualBox (not covered in these instructions), downloaded the Vagrant install package and the packer binary zip, make sure Vagrant is installed and working
```bash
$ sudo dpkg -i /home/davent/Downloads/vagrant_1.4.1_x86_64.deb
$ vagrant -v
Vagrant 1.4.1
```
check out the project files and change in to the working directoy
```bash
$ git clone --recursive git@bitbucket.org:davent/vagrant-mcollective.git
$ cd vagrant-mcollective
```
unzip the packer binaries in to the packer/bin directory and then from the packer directory build the Vagrant box using packer
```bash
$ unzip /home/davent/Downloads/0.4.1_linux_amd64.zip -d packer/bin
$ cd packer
$ ./bin/packer build centos-6.5-x86_64.json
(or)
$ ./bin/packer build debian-7.3.0-amd64.json
```
we should now have a Centos 6.5 or a Debian Wheezy Vagrant box in the build directory which Vagrant can use.

Finally we can build our environemnt using Vagrant. From the root of the working directory run Vagrant
```bash
$ cd ../
$ vagrant up
```
and that's it! We should now be able to log in to the mCollective client node and test that mCollective is working
```bash
$ vagrant ssh mco-client
[vagrant@mco-client ~]$ sudo mco ping
mco-node-0                               time=84.15 ms
mco-client                               time=90.55 ms


---- ping statistics ----
2 replies max: 90.55 min: 84.15 avg: 87.35
```

### Want more nodes?

Simply open the Vagrant file in your favourite editor and edit this line (line 4):
```bash
num_of_nodes = 1
```

### Enjoy!









