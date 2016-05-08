ccpe2014
========

This is a meta-project for the code used in the experimental evaluation presented in the article:

> [FC2Q: Exploiting Fuzzy Control in Server Consolidation for Cloud Applications with SLA Constraints](http://dx.doi.org/10.1002/cpe.3410)

Please, cite this code by citing the above article:

		@ARTICLE{CPE:CPE3410,
			author = {Cosimo Anglano and Massimo Canonico and Marco Guazzone},
			title = {{FC2Q}: Exploiting Fuzzy Control in Server Consolidation for Cloud Applications with {SLA} Constraints},
			journal = {Concurrency and Computation: Practice and Experience},
			volume = {27},
			number = {17},
			issn = {1532-0634},
			url = {http://dx.doi.org/10.1002/cpe.3410},
			doi = {10.1002/cpe.3410},
			pages = {4491--4514},
			keywords = {cloud computing, resource management, feedback control, fuzzy control, virtualized cloud applications},
			year = {2015},
		}


Introduction
------------

The code is composed by two macro-modules:

* *server-side module* contains code to be run on the server-side (e.g., inside a virtual machine (VM)). The module is composed by:

	* `server/olio-dev.zip`: the archive containing the [patched sources](http://github.com/sguazt/olio) of the [Apache Olio](http://incubator.apache.org/projects/olio.html) application (v0.2)
	* `server/RUBiS-dev.zip`: the archive containing the [patched sources](http://github.com/sguazt/RUBiS) of the [OW2 RUBiS](http://rubis.ow2.org) application (v1.4.3)
	* `server/apache-cassandra-0.7.9-bin.tar.gz`: the archive containing the binaries of the [Apache Cassandra](http://cassandra.apache.org) (v0.7.9)

* *client-side module* contains code to be run on the client-side. The module is composed by:

	* `client/boost-ublasx-v1.zip`: the archive containing the sources of the [Boost.uBLASx](http://github.com/sguazt/boost-ublasx) (v1)
	* `client/dcsxx-commons-v2.zip`: the archive containing the sources of the [dcsxx-commons](http://github.com/sguazt/dcsxx-commons) (v2)
	* `client/dcsxx-control-v2.zip`: the archive containing the sources of the [dcsxx-control](http://github.com/sguazt/dcsxx-control) (v2)
	* `client/dcsxx-sysid-v2.zip`: the archive containing the sources of the [dcsxx-sysid](http://github.com/sguazt/dcsxx-sysid) (v2)
	* `client/dcsxx-testbed-v2.zip`: the archive containing the sources of the [dcsxx-testbed](http://github.com/sguazt/dcsxx-testbed) (v2)
	* `client/rain-workload-toolkit-dev.zip`: the archive containing the sources of the [RAIN](http://github.com/yungsters/rain-workload-toolkit) workload toolkit (development version)
	* `client/YCSB-dev.zip`: the archive containing the sources of the [YCSB](http://github.com/brianfrankcooper/YCSB) benchmark (development version)

For a more updated version of the above components, see the related project Web sites.


Server-side Setup
-----------------

In your host operating system, you must install the [Xen](http://www.xen.org) hypervisor.
Furthermore, each tier of the applications you plan to run, should be installed in a separate VM.
Finally, to enable remote communications between the Xen hypervisor and client-side components, you need to install the [libvirt](http://libvirt.org) virtualization API and run the `libvirtd` daemon.

In our experiments, we used the [Fedora](http://fedoraproject.org) 18 Linux distribution as the host operating system.
For a guide on how to install and setup Xen on Fedora you can, for instance, refer to the [Fedora Host Installation](http://wiki.xen.org/wiki/Fedora_Host_Installation) page of the Xen's wiki site.

For each application tier, we setup a dedicated VM, that is:

* For Olio, we created two VMs, one for the Web tier and another one for the DB tier.
* For RUBiS, we created two VMs, one for the Web tier and another one for the DB tier.
* For Cassandra, we created only one VM.

For each of the above VM, we used the [CentOS](http://www.centos.org) 6.5 Linux distribution as the guest operating system.

### Prerequisites

* [libvirt](http://libvirt.org) virtualization API library (v1.1 or newer)
* [Xen](http://www.xen.org) virtualization platform (v4.2 or newer)

### Libvirt Daemon Setup

The `libvirtd` daemon is to be started on the host system in order to enable remote communications between the client-side components and the (server-side) VMM hypervisor (Xen in our case).

In the following we setup *libvirt* in order to accept remote communications by using secure TLS connections on the public TCP/IP port.
When we refer to the *server machine* we mean the machine that accepts remote requests and where a `libvirtd` daemon instance is running (i.e., the host system).
Instead, when we refer to the *client machine* we mean the machine from which you issue remote requests (i.e., the machine that runs the client-side components).

In the rest of this section, we assume a Red Hat Linux like operating system.

1. On the *server machine*, it is necessary to start the `libvirtd` server in listening mode by running it with the `--listen` option or by editing the `/etc/sysconfig/libvirtd` file to add the `LIBVIRTD_ARGS="--listen"` line, in order to cause `libvirtd` to come up in listening mode whenever it is started.
2. Also, in order to accept remote connections, you need to open in your firewall the `libvirtd` port, which is usually `16514` (check your `/etc/libvirt/libvirtd.conf` file), for the TCP protocol.
3. Setup a Certificate Authority (CA), for instance, by using the `certtool` utility from the *GnuTLS* library.
   After this step, you have two files, say:

	* `cakey.pem`: your CA's (secret) private key
	* `cacert.pem`: your CA's (public) certificate
4. Install the file `cacert.pem` on both the *client* and *server machines* to let them know that they can trust certificates issued by your CA.
   The usual installation directory for `cacert.pem` is `/etc/pki/CA`
5. Issue servers certificates.
   In the *server machine*, you need to issue a certificate with the *X.509 CommonName* (CN) field set to the host name of the server.
   The CN must match the host name that clients will use to connect to the server.
   After this step, you have two files, say:

	* `serverkey.pem`: is the server's private key
	* `servercert.pem`: is the server's certificate

   that have to be installed on the server.
   Also note that the `serverkey.pem` file **must** have file permissions set to `600`.
   The usual installation directory for `serverkey.pem` is `/etc/pki/libvirt/private`.
   The usual installation directory for `servercert.pem` is `/etc/pki/libvirt`.
6. Issue clients certificates.
   In the *client machine* you need to issue a certificate with the *X.509 Distinguished Name* (DN) set to a suitable name (e.g., the client name).
   Also, make sure the server host name is recognized by your client system (e.g., put it in the `/etc/hosts` file).
   After this step, you have two files, say:

	* `clientkey.pem`: is the client's private key
	* `clientcert.pem`: is the client's certificate

   that have to be installed on the client.
   The usual installation directory for `clientkey.pem` is `/etc/pki/libvirt/private`.
   The usual installation directory for `clientcert.pem` is `/etc/pki/libvirt`.

### Olio Setup

1. Create two VMs, one for the Web tier and another one for the DB tier.
2. Copy the file `server/olio-dev.zip` inside each VM.
3. Unzip the file `olio-dev.zip`.
4. Follow building instructions (for the tier running inside the VM) at `olio/docs/php_setup.html`.

### RUBiS Setup

1. Create two VMs, one for the Web tier and another one for the DB tier.
2. Copy the file `server/RUBiS-dev.zip` inside each VM.
3. Log into each VM.
4. Unzip the file `RUBiS-dev.zip`.
5. Follow building instructions (for the tier running inside the VM) at `RUBiS/README.md`.

### Cassandra Setup

The *Apache Cassandra* version included in this distribution does not need to be compiled since it already comes in a binary form.

1. Create one VM
2. Copy the file `server/apache-cassandra-0.7.9-bin.tar.gz` inside the VM.
3. Log into the VM.
4. Unzip the file `apache-cassandra-0.7.9-bin.tar.gz`
5. Follow instructions at [Datastax](http://www.datastax.com/docs/0.7/index)


Client-side Setup
-----------------

### Prerequisites

* A modern C++98 compiler (e.g., GCC v4.8 or newer is fine)
* The GNU make tool
* [Apache Ant](http://ant.apache.org) (v1.8 or newer)
* [Apache Maven](http://maven.apache.org) (v3 or newer)
* [Boost](http://boost.org) C++ libraries (v1.54 or newer)
* [Boost.Numeric Bindings](http://svn.boost.org/svn/boost/sandbox/numeric_bindings) library (v2 or newer)
* [fuzzylite](http://www.fuzzylite.com) fuzzy logic control library (v4 or newer)
* [LAPACK](http://www.netlib.org/lapack/) Linear Algebra PACKage (v3.5 or newer)
* [libvirt](http://libvirt.org) virtualization API library (v1.1 or newer)
* [Oracle Java SE](http://www.oracle.com/technetwork/java/javase/index.html) SDK (v7 or newer)

Also, for a more detailed list of requirements, see the documentation of the various included sub-projects.

### RAIN Setup

1. Unzip file `files/rain-workload-toolkit-dev.zip`
2. Change current directory into `rain-workload-toolkit`
3. Run `ant package`
4. For Olio workload driver:

	1. Edit `config/rain.config.olio.json` and `config/profiles.config.olio.json` (see `rain-workload-toolkit/src/radlab/rain/workload/olio/README.md`, for more information)
	2. Run `ant package-olio`
5. For RUBiS workload driver:

	1. Edit `config/rain.config.rubis.json` and `config/profiles.config.rubis.json` (see `rain-workload-toolkit/src/radlab/rain/workload/rubis/README.md`, for more information)
	2. Run `ant package-rubis`

### YCSB Setup

1. Unzip file `files/YCSB-dev.zip`
2. Change current directory into `YCSB`
3. Run `mvn clean package` (see `YCSB/BUILD` and `YCSB/README.md` for more information)

### dcsxx-testbed Setup

1. Unzip file `files/dcsxx-testbed-v2.zip`
2. Follow instructions in `dcsxx-testbed/README.md`


Bugs and Contributions
----------------------

Bug notification and patches are always welcomed.
Also, other type of contributions (e.g., new features or improvement) are welcomed, as well.

Please, note that, since this is only a meta-project (i.e., a container for other sub-projects), for feedback and contributions you are asked to refer to the specific sub-project.
