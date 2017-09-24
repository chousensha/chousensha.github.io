---
layout: post
title: "KVM virtualization on Linux hosts"
date: 2017-09-24 11:34:34 -0400
comments: true
categories: [linux, sysadmin]
keywords: kvm, virtualization, linux virtualization, libvirt, virsh
description: Using the KVM hypervisor on a Linux physical host
---



By now you probably noticed that the last topics were centered more around Linux than the usual. That's because I am studying for my LFCS certification, and creating posts as I go through the material. This time, we're going to look at using virtualization on a Linux physical host. Since my only physical Linux at the moment is Kali, this is what I'm going to use for today's post.

<!-- more -->

The first step before starting anything else, is to check if virtualization is enabled on your machine. For this, you can query your <code>/proc/cpuinfo</code> file and look for the following flags:

- **vmx** for Intel CPUs

- **svm** for AMD CPUs

I already knew the result in my case, but I checked anyway with <code>grep --color vmx /proc/cpuinfo</code>:

{% img center /images/sysadmin/kvm/vmx.png 'vmx' 'vmx enabled' %}

Ok, the next step is to install the virtualization utilities:

```
apt-get install qemu-kvm virt-manager libvirt0 qemu-system virt-viewer virt-top
```

KVM (Kernel Virtual Machine) is a full virtualization solution for Linux on x86 and x64 hardware containing virtualization extensions (Intel VT or AMD-V). It consists of a loadable kernel module, kvm.ko, that provides the core virtualization infrastructure and a processor specific module, kvm-intel.ko or kvm-amd.ko.

libvirt is an API toolkit for managing virtualization hosts

The virt-manager application is a graphical tool for managing virtual machines through libvirt. It primarily targets KVM VMs, but also manages Xen and LXC (linux containers).

virt-viewer allows access to the virtual machine console

virt-top is for monitoring VMs performance

After installing everything, you can go to <code>/etc/libvirt/qemu/networks</code> and look inside the **default.xml** file to see how the default virtual network will be configured:

```
<!--
WARNING: THIS IS AN AUTO-GENERATED FILE. CHANGES TO IT ARE LIKELY TO BE
OVERWRITTEN AND LOST. Changes to this xml configuration should be made using:
  virsh net-edit default
or other application using the libvirt API.
-->

<network>
  <name>default</name>
  <uuid>d67cdb82-233e-44f1-a6ac-9293faa2258d</uuid>
  <forward mode='nat'/>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:b2:a7:6e'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>
```

Let's see if the default network has been created. Don't forget to start the **libvirtd** daemon first, with <code>systemctl start libvirtd</code> and then use virsh to list the defined networks. **virsh** is a powerful command line interface tool for managing guests and the hypervisor. It can also be used as an interactive prompt.

```
virsh
Welcome to virsh, the virtualization interactive terminal.

Type:  'help' for help with commands
       'quit' to quit

virsh # help
Grouped commands:

 Domain Management (help keyword 'domain'):
    attach-device                  attach device from an XML file
    attach-disk                    attach disk device
    attach-interface               attach network interface
    autostart                      autostart a domain
    blkdeviotune                   Set or query a block device I/O tuning parameters.
    blkiotune                      Get or set blkio parameters
    blockcommit                    Start a block commit operation.
    blockcopy                      Start a block copy operation.
    blockjob                       Manage active block operations
    blockpull                      Populate a disk from its backing image.
    blockresize                    Resize block device of domain.
    change-media                   Change media of CD or floppy drive
    console                        connect to the guest console
    cpu-baseline                   compute baseline CPU
    cpu-compare                    compare host CPU with a CPU described by an XML file
    cpu-stats                      show domain cpu statistics
    create                         create a domain from an XML file
    define                         define (but don't start) a domain from an XML file
    desc                           show or set domain's description or title
    destroy                        destroy (stop) a domain
    detach-device                  detach device from an XML file
    detach-disk                    detach disk device
    detach-interface               detach network interface
    domdisplay                     domain display connection URI
    domfsfreeze                    Freeze domain's mounted filesystems.
    domfsthaw                      Thaw domain's mounted filesystems.
    domfsinfo                      Get information of domain's mounted filesystems.
    domfstrim                      Invoke fstrim on domain's mounted filesystems.
    domhostname                    print the domain's hostname
    domid                          convert a domain name or UUID to domain id
    domif-setlink                  set link state of a virtual interface
    domiftune                      get/set parameters of a virtual interface
    domjobabort                    abort active domain job
    domjobinfo                     domain job information
    domname                        convert a domain id or UUID to domain name
    domrename                      rename a domain
    dompmsuspend                   suspend a domain gracefully using power management functions
    dompmwakeup                    wakeup a domain from pmsuspended state
    domuuid                        convert a domain name or id to domain UUID
    domxml-from-native             Convert native config to domain XML
    domxml-to-native               Convert domain XML to native config
    dump                           dump the core of a domain to a file for analysis
    dumpxml                        domain information in XML
    edit                           edit XML configuration for a domain
    event                          Domain Events
    inject-nmi                     Inject NMI to the guest
    iothreadinfo                   view domain IOThreads
    iothreadpin                    control domain IOThread affinity
    iothreadadd                    add an IOThread to the guest domain
    iothreaddel                    delete an IOThread from the guest domain
    send-key                       Send keycodes to the guest
    send-process-signal            Send signals to processes
    lxc-enter-namespace            LXC Guest Enter Namespace
    managedsave                    managed save of a domain state
    managedsave-remove             Remove managed save of a domain
    memtune                        Get or set memory parameters
    perf                           Get or set perf event
    metadata                       show or set domain's custom XML metadata
    migrate                        migrate domain to another host
    migrate-setmaxdowntime         set maximum tolerable downtime
    migrate-compcache              get/set compression cache size
    migrate-setspeed               Set the maximum migration bandwidth
    migrate-getspeed               Get the maximum migration bandwidth
    migrate-postcopy               Switch running migration from pre-copy to post-copy
    numatune                       Get or set numa parameters
    qemu-attach                    QEMU Attach
    qemu-monitor-command           QEMU Monitor Command
    qemu-monitor-event             QEMU Monitor Events
    qemu-agent-command             QEMU Guest Agent Command
    reboot                         reboot a domain
    reset                          reset a domain
    restore                        restore a domain from a saved state in a file
    resume                         resume a domain
    save                           save a domain state to a file
    save-image-define              redefine the XML for a domain's saved state file
    save-image-dumpxml             saved state domain information in XML
    save-image-edit                edit XML for a domain's saved state file
    schedinfo                      show/set scheduler parameters
    screenshot                     take a screenshot of a current domain console and store it into a file
    set-user-password              set the user password inside the domain
    setmaxmem                      change maximum memory limit
    setmem                         change memory allocation
    setvcpus                       change number of virtual CPUs
    shutdown                       gracefully shutdown a domain
    start                          start a (previously defined) inactive domain
    suspend                        suspend a domain
    ttyconsole                     tty console
    undefine                       undefine a domain
    update-device                  update device from an XML file
    vcpucount                      domain vcpu counts
    vcpuinfo                       detailed domain vcpu information
    vcpupin                        control or query domain vcpu affinity
    emulatorpin                    control or query domain emulator affinity
    vncdisplay                     vnc display
    guestvcpus                     query or modify state of vcpu in the guest (via agent)
    setvcpu                        attach/detach vcpu or groups of threads
    domblkthreshold                set the threshold for block-threshold event for a given block device or it's backing chain element

 Domain Monitoring (help keyword 'monitor'):
    domblkerror                    Show errors on block devices
    domblkinfo                     domain block device size information
    domblklist                     list all domain blocks
    domblkstat                     get device block stats for a domain
    domcontrol                     domain control interface state
    domif-getlink                  get link state of a virtual interface
    domifaddr                      Get network interfaces' addresses for a running domain
    domiflist                      list all domain virtual interfaces
    domifstat                      get network interface stats for a domain
    dominfo                        domain information
    dommemstat                     get memory statistics for a domain
    domstate                       domain state
    domstats                       get statistics about one or multiple domains
    domtime                        domain time
    list                           list domains

 Host and Hypervisor (help keyword 'host'):
    allocpages                     Manipulate pages pool size
    capabilities                   capabilities
    cpu-models                     CPU models
    domcapabilities                domain capabilities
    freecell                       NUMA free memory
    freepages                      NUMA free pages
    hostname                       print the hypervisor hostname
    maxvcpus                       connection vcpu maximum
    node-memory-tune               Get or set node memory parameters
    nodecpumap                     node cpu map
    nodecpustats                   Prints cpu stats of the node.
    nodeinfo                       node information
    nodememstats                   Prints memory stats of the node.
    nodesuspend                    suspend the host node for a given time duration
    sysinfo                        print the hypervisor sysinfo
    uri                            print the hypervisor canonical URI
    version                        show version

 Interface (help keyword 'interface'):
    iface-begin                    create a snapshot of current interfaces settings, which can be later committed (iface-commit) or restored (iface-rollback)
    iface-bridge                   create a bridge device and attach an existing network device to it
    iface-commit                   commit changes made since iface-begin and free restore point
    iface-define                   define an inactive persistent physical host interface or modify an existing persistent one from an XML file
    iface-destroy                  destroy a physical host interface (disable it / "if-down")
    iface-dumpxml                  interface information in XML
    iface-edit                     edit XML configuration for a physical host interface
    iface-list                     list physical host interfaces
    iface-mac                      convert an interface name to interface MAC address
    iface-name                     convert an interface MAC address to interface name
    iface-rollback                 rollback to previous saved configuration created via iface-begin
    iface-start                    start a physical host interface (enable it / "if-up")
    iface-unbridge                 undefine a bridge device after detaching its slave device
    iface-undefine                 undefine a physical host interface (remove it from configuration)

 Network Filter (help keyword 'filter'):
    nwfilter-define                define or update a network filter from an XML file
    nwfilter-dumpxml               network filter information in XML
    nwfilter-edit                  edit XML configuration for a network filter
    nwfilter-list                  list network filters
    nwfilter-undefine              undefine a network filter

 Networking (help keyword 'network'):
    net-autostart                  autostart a network
    net-create                     create a network from an XML file
    net-define                     define an inactive persistent virtual network or modify an existing persistent one from an XML file
    net-destroy                    destroy (stop) a network
    net-dhcp-leases                print lease info for a given network
    net-dumpxml                    network information in XML
    net-edit                       edit XML configuration for a network
    net-event                      Network Events
    net-info                       network information
    net-list                       list networks
    net-name                       convert a network UUID to network name
    net-start                      start a (previously defined) inactive network
    net-undefine                   undefine a persistent network
    net-update                     update parts of an existing network's configuration
    net-uuid                       convert a network name to network UUID

 Node Device (help keyword 'nodedev'):
    nodedev-create                 create a device defined by an XML file on the node
    nodedev-destroy                destroy (stop) a device on the node
    nodedev-detach                 detach node device from its device driver
    nodedev-dumpxml                node device details in XML
    nodedev-list                   enumerate devices on this host
    nodedev-reattach               reattach node device to its device driver
    nodedev-reset                  reset node device
    nodedev-event                  Node Device Events

 Secret (help keyword 'secret'):
    secret-define                  define or modify a secret from an XML file
    secret-dumpxml                 secret attributes in XML
    secret-event                   Secret Events
    secret-get-value               Output a secret value
    secret-list                    list secrets
    secret-set-value               set a secret value
    secret-undefine                undefine a secret

 Snapshot (help keyword 'snapshot'):
    snapshot-create                Create a snapshot from XML
    snapshot-create-as             Create a snapshot from a set of args
    snapshot-current               Get or set the current snapshot
    snapshot-delete                Delete a domain snapshot
    snapshot-dumpxml               Dump XML for a domain snapshot
    snapshot-edit                  edit XML for a snapshot
    snapshot-info                  snapshot information
    snapshot-list                  List snapshots for a domain
    snapshot-parent                Get the name of the parent of a snapshot
    snapshot-revert                Revert a domain to a snapshot

 Storage Pool (help keyword 'pool'):
    find-storage-pool-sources-as   find potential storage pool sources
    find-storage-pool-sources      discover potential storage pool sources
    pool-autostart                 autostart a pool
    pool-build                     build a pool
    pool-create-as                 create a pool from a set of args
    pool-create                    create a pool from an XML file
    pool-define-as                 define a pool from a set of args
    pool-define                    define an inactive persistent storage pool or modify an existing persistent one from an XML file
    pool-delete                    delete a pool
    pool-destroy                   destroy (stop) a pool
    pool-dumpxml                   pool information in XML
    pool-edit                      edit XML configuration for a storage pool
    pool-info                      storage pool information
    pool-list                      list pools
    pool-name                      convert a pool UUID to pool name
    pool-refresh                   refresh a pool
    pool-start                     start a (previously defined) inactive pool
    pool-undefine                  undefine an inactive pool
    pool-uuid                      convert a pool name to pool UUID
    pool-event                     Storage Pool Events

 Storage Volume (help keyword 'volume'):
    vol-clone                      clone a volume.
    vol-create-as                  create a volume from a set of args
    vol-create                     create a vol from an XML file
    vol-create-from                create a vol, using another volume as input
    vol-delete                     delete a vol
    vol-download                   download volume contents to a file
    vol-dumpxml                    vol information in XML
    vol-info                       storage vol information
    vol-key                        returns the volume key for a given volume name or path
    vol-list                       list vols
    vol-name                       returns the volume name for a given volume key or path
    vol-path                       returns the volume path for a given volume name or key
    vol-pool                       returns the storage pool for a given volume key or path
    vol-resize                     resize a vol
    vol-upload                     upload file contents to a volume
    vol-wipe                       wipe a vol

 Virsh itself (help keyword 'virsh'):
    cd                             change the current directory
    echo                           echo arguments
    exit                           quit this interactive terminal
    help                           print help
    pwd                            print the current directory
    quit                           quit this interactive terminal
    connect                        (re)connect to hypervisor
```

Ok, default network check now:

```
virsh net-list --all
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              inactive   no            yes
```

The default network is inactive, so we have to enable it:

```
virsh net-start default
Network default started
```

Now, if you do an *ifconfig*, you should see the new bridge interface:

```
virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:b2:a7:6e  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

To get more info about your bridge, you can use the **brctl** command:

```
brctl
Usage: brctl [commands]
commands:
	addbr     	<bridge>		add bridge
	delbr     	<bridge>		delete bridge
	addif     	<bridge> <device>	add interface to bridge
	delif     	<bridge> <device>	delete interface from bridge
	hairpin   	<bridge> <port> {on|off}	turn hairpin on/off
	setageing 	<bridge> <time>		set ageing time
	setbridgeprio	<bridge> <prio>		set bridge priority
	setfd     	<bridge> <time>		set bridge forward delay
	sethello  	<bridge> <time>		set hello time
	setmaxage 	<bridge> <time>		set max message age
	setpathcost	<bridge> <port> <cost>	set path cost
	setportprio	<bridge> <port> <prio>	set port priority
	show      	[ <bridge> ]		show a list of bridges
	showmacs  	<bridge>		show a list of mac addrs
	showstp   	<bridge>		show bridge stp info
	stp       	<bridge> {on|off}	turn stp on/off
```

> brctl is used to set up, maintain, and inspect the ethernet bridge configuration in the linux kernel.

> An ethernet bridge is a device commonly used to connect different  networks of ethernets together, so that these 
> ethernets will appear as one ethernet to the participants.

> Each of the ethernets  being  connected  corresponds  to  one  physical interface  in  the  bridge. These individual
> ethernets are bundled into one bigger ('logical') ethernet, this bigger  ethernet  corresponds  to the bridge 
> network interface.

```
brctl show
bridge name	bridge id		STP enabled	interfaces
virbr0		8000.525400b2a76e	yes		virbr0-nic
```

Ok, now we have the default network set. It's time to actually start some VMs. In KVM, VMs can be managed both from the command line with virsh, and from the GUI, with Virtual Network Manager, which you can find under System Tools.

{% img center /images/sysadmin/kvm/virtmanager.png 'virtmanager' 'virtmanager' %}

**Before starting to create VMs, please reboot your host to save you time and headaches**.

I grabbed a Debian iso to showcase its live CD, and I created a VM from it:

{% img center /images/sysadmin/kvm/1-new.png 'new vm' 'new vm' %}

{% img center /images/sysadmin/kvm/2-os.png 'os' 'os' %}

{% img center /images/sysadmin/kvm/3-hw.png 'hardware' 'hardware specs' %}

{% img center /images/sysadmin/kvm/4-space.png 'disk' 'disk' %}

{% img center /images/sysadmin/kvm/5-final.png 'create vm' 'create vm' %}

I had to do one more step before getting rid of errors every time I finished with the VM creation. libvirt was pointing to the qemu-kvm binary, but I had no such file, so I created a symlink as specified in this [Arch Linux forum](https://bbs.archlinux.org/viewtopic.php?pid=1240205#p1240205):

```
ln -s /usr/bin/qemu-{system-x86_64,kvm}
ls -la /usr/bin/qemu-kvm
lrwxrwxrwx 1 root root 27 Sep 17 13:19 /usr/bin/qemu-kvm -> /usr/bin/qemu-system-x86_64
```

{% img center /images/sysadmin/kvm/vm.png 'vm' 'working vm' %}

{% img center /images/sysadmin/kvm/details.png 'vm details' 'vm details' %}

KVM graphical interface is nice, but we are not limited to a GUI. We can create and interact with VMs from the command line!

First, let's see if what VMs we've got:

```
virsh list --all
 Id    Name                           State
----------------------------------------------------
 -     generic                        shut off
```

If we only wanted to list the running VMs, we could do it with <code>virsh list</code>

Now, I'm going to start a VM, which in virsh talk is called a domain, and look at some info about it:

```
virsh # start generic
Domain generic started

virsh # dominfo generic
Id:             1
Name:           generic
UUID:           c4982e35-ec8d-4b5f-9ad3-c8491bcd9181
OS Type:        hvm
State:          running
CPU(s):         1
CPU time:       0.4s
Max memory:     1048576 KiB
Used memory:    1048576 KiB
Persistent:     yes
Autostart:      disable
Managed save:   no
Security model: none
Security DOI:   0
```

Now we have a better grasp on Linux-based hypervisors, and we don't have to rely on VMware and Virtualbox only. In a future post I will show how to install a VM from the command line.

``` 
 ______________________________________
/ Q: How did you get into artificial   \
| intelligence? A: Seemed logical -- I |
\ didn't have any real intelligence.   /
 --------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

