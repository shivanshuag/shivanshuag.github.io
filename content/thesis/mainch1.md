+++
title = "Introduction: Virtualization and Resource Management in Cloud"
menu = "main"
url = "thesis/mainch1.html"
+++

{{< raw >}}
<div class="crosslinks"><p class="noindent">[<a
href="mainch2.html" >next</a>] [<a
href="mainli4.html" >prev</a>] [<a
href="mainli4.html#tailmainli4.html" >prev-tail</a>] [<a
href="#tailmainch1.html">tail</a>] [<a
href="main.html#mainch1.html" >up</a>] </p></div>
<h2 class="chapterHead"><span class="titlemark">Chapter 1</span><br /><a
href="main.html#QQ2-6-7" id="x6-50001">Introduction: Virtualization and Resource Management in Cloud</a></h2>
<p>With the advent of large scale cloud computing, the users can get compute resources on
demand with ﬂexible pricing models. Cloud vendors pool their massive hardware
resources and provide virtual machines on top of it to the users. To best utilize the
resources of a virtualized cloud infrastructure, resource overcommitment is used.
Allocating more virtual resources to a machine or a group of machines than are physically
present is called resource overcommitment.
</p>
<p>   Since most applications will not use all of the resources allocated to them at all the
times, most of the resources of a cloud provider will remain idle without overcommitment.
Hence, this approach is more proﬁtable and less wasteful. Orthogonal to overcommitment
is the fact that cloud vendors have to satisfy some SLAs (Service Level Agreements)
which they have with the users i.e. the promised resources should be available to the
users whenever they need it. Distributed resource scheduling (DRS) is used to meet these
SLAs.
</p>
<p>   Apart from the public cloud oﬀerings like Amazon Web Services and Microsoft
Azure, many companies and educational institutions are virtualizing their IT
infrastructure to create private clouds. Private clouds may have less strict SLA&#x2019;s but
require resource scheduling to enhance performance of the virtual machines.
Overcommitment without resource management may lead to degradation in
performance.
<a
id="x6-5001r1"></a>
</p>

<h3 class="sectionHead"><span class="titlemark">1.1   </span> <a
href="main.html#QQ2-6-8" id="x6-60001">Virtualization</a></h3>
<p>Virtualization is one of the driving technologies behind IaaS (Infrastructure as a Service).
Virtualization makes it possible to run multiple operating systems with diﬀerent
conﬁgurations on a physical machine at the same time.
</p>
<p>   To run virtual machines on a system, a software layer called hypervisor or virtual


machine monitor (VMM) is required. The hypervisor has the control of all the hardware
resources and can take away resources from one VM to give it to another. The hypervisor
also maintains the state of all the VMs at all the times. It does these by trapping all the
privileged instructions executed by the guest VM and emulating the resource they access.
The hypervisor is responsible for emulating all the hardware devices and providing
proper resource isolation between multiple machines running on the same physical
machine.
</p>
<p>   There are three diﬀerent techniques used for virtualization [<a id="page.4"></a><a
href="mainli5.html#X0-horne2007understanding" >1</a>] which mainly
diﬀer in the way they trap the privileged instructions executed by the guest
kernel.
   </p>
<dl class="enumerate"><dt class="enumerate">
1. </dt><dd
class="enumerate"><strong><span
class="cmbx-12">Full Virtualization with Binary Translation.</span></strong> In this approach, user mode
   code runs directly on CPU without any translation, but the non-virtualizable
   instructions [<a
href="mainli5.html#X0-Popek:1974:FRV:361011.361073" >2</a>] in the guest kernel code are translated on the ﬂy to code which
   has the intended eﬀect on the virtual hardware.
   </dd><dt class="enumerate">
2. </dt><dd
class="enumerate"><strong><span
class="cmbx-12">Hardware Assisted Full Virtualization.</span></strong>To make virtualization simpler,
   hardware vendors have developed new features in the hardware to support
   virtualization.  Intel  VT-x  and  AMD-V  are  two  technologies  developed  by
   Intel and AMD respectively which provide special instructions in their ISA
   (Instruction Set Architecture) for virtual machines and a new ring privilege
   level for VM. Privileged and sensitive calls are set to automatically trap to the
   VMM, removing the need for either binary translation or paravirtualization.
   It also has modiﬁed MMU with support for multi level page tables [<a
href="mainli5.html#X0-bhargava2008accelerating" >3</a>] and
   tagged TLBs.
   </dd><dt class="enumerate">
3. </dt><dd
class="enumerate"><strong><span
class="cmbx-12">Paravirtualization.</span></strong> This technique requires modiﬁcation of the guest kernel.
   The non-virtualizable/privileged instructions in the source code of the guest
   kernel are replaced with hypercalls which directly call the hypervisor [<a
href="mainli5.html#X0-barham2003xen" >4</a>]. The


   hypervisor  provides  hypercall  interfaces  for  kernel  operations  like  memory
   management, interrupt handling, and communication to devices. It diﬀers from
   full virtualization, where unmodiﬁed guest kernel is used and the guest OS
   does not know that it is running in a virtualized environment.</dd></dl>
<p>   Hypervisors can be bare-metal hypervisors or hosted hypervisors. A Bare-metal
hypervisor runs directly on the physical hardware while the hosted hypervisor runs on top
of conventional operating systems. There are several hypervisors available in the market
with VMWare ESX and Xen [<a id="page.5"></a><a
href="mainli5.html#X0-barham2003xen" >4</a>] being the popular bare-metal hypervisors, while
KVM-QEMU [<a
href="mainli5.html#X0-kivity2007kvm" >5</a>, <a
href="mainli5.html#X0-bellard2005qemu" >6</a>] being a popular hosted hypervisor which runs on top of the Linux
operating system. KVM is a kernel module providing support for hardware assited
virtualization in Linux while, QEMU is a userspace emulator. KVM uses QEMU mainly
for emulating the hardware [<a
href="mainli5.html#X0-Habib:2008:VK:1344209.1344217" >7</a>]. So, both these pieces of software work together as a
complete hypervisor for linux. KVM-QEMU and Xen are open source while ESX is
proprietary. For the purpose of this thesis, we will refer to KVM-QEMU wherever
hypervisor is used unless speciﬁed otherwise.
</p>
<p>   Virtualization provides a number of beneﬁts other than resource isolation, which
makes it the fundamental technology behind IaaS.
   </p>
<dl class="enumerate"><dt class="enumerate">
1. </dt><dd
class="enumerate">It provides the ability to treat disks of virtual machine as ﬁles which can be
   easily snapshotted for backup and restore.
   </dd><dt class="enumerate">
2. </dt><dd
class="enumerate">It provides ease of creation of new machines and deployment of applications
   through pre-built images of the ﬁlesystem of the machine.
   </dd><dt class="enumerate">
3. </dt><dd
class="enumerate">Virtual machines can be easily migrated or relocated if the physical machines
   may require maintenance or develop some failure.
   </dd><dt class="enumerate">
4. </dt><dd
class="enumerate">Ease in increasing the resource capacity (RAM or CPU cores) of the machine


   at runtime by CPU or memory hotplug [<a
href="mainli5.html#X0-Hansen_hotplugmemory" >8</a>], or otherwise.
   </dd><dt class="enumerate">
5. </dt><dd
class="enumerate">Since  the  hardware  resources  are  emulated  by  the  hypervisor,  there  is  an
   opportunity for overcommitment of CPU and memory resources here.</dd></dl>
<a
id="x6-6009r1"></a>
<h4 class="subsectionHead"><span class="titlemark">1.1.1   </span> <a
href="mainli2.html#QQ2-6-9" id="x6-70001">Memory Overcommitment and Ballooning</a></h4>
<p>In memory overcommitment, more memory is allocated to the virtual machines(VM) than
is physically present in hardware. This is possible because hypervisors allocate memory to
the virtual machines on demand. KVM-QEMU treats all the running VMs as
processes of the host system and uses malloc to allocate memory for a VM&#x2019;s RAM.
Linux uses demand paging for its processes, so a VM on bootup will allocate
only the amount of memory required by it for booting up, and not its whole
capacity.
</p>
<p>   On demand memory allocation in itself is not enough to make memory overcommitment a
viable option. There is no way for the hypervisor to free a memory page that has been
freed by the guest OS. Hence a page once allocated to a VM always remains allocated.
The hypervisor should be able to reclaim free memory from the guest machines, otherwise
the memory consumption of guest machines will always keep on increasing till
they use up all their memory capacity. If the memory is overcommited, all the
guests trying to use their maximum capacity will lead to swapping and very poor
performance.
</p>
<p>   There exists a mechanism called <em><span
class="cmti-12">memory ballooning</span></em> to reclaim free memory from
guest machines. This is possible through a device driver that exists in guest
operating system and a backend virtual device in the hypervisor which talks to that
device driver. The balloon driver takes a target memory from the balloon device.
If the target memory is less than the current memory of the VM, it allocates
<!--l. 62--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mrow ><mo
class="MathClass-open">(</mo><mrow><mi
>c</mi><mi
>u</mi><mi
>r</mi><mi
>r</mi><mi
>e</mi><mi
>n</mi><mi
>t</mi> <mo
class="MathClass-bin">−</mo> <mi
>t</mi><mi
>a</mi><mi
>r</mi><mi
>g</mi><mi
>e</mi><mi
>t</mi></mrow><mo
class="MathClass-close">)</mo></mrow></math> pages
from the machine and gives them back to the hypervisor. This process is called balloon


inﬂation. If the target memory is more than the current memory, the balloon driver frees
required pages from the balloon. This process is called balloon deﬂation. Memory
ballooning is an opportunistic reclamation technique and does not guarantee reclamation.
The hypervisor has limited control over the success of reclamation and the amount of
memory reclaimed, as it depends on the balloon driver which is loaded inside the guest
operating system.
</p>
<p>   The Memory ballooning technique leverages modiﬁed guest kernel as it uses an extra
device driver and hence falls under Paravirtualization. But the virtio balloon driver [<a id="page.7"></a><a
href="mainli5.html#X0-russell2008virtio" >9</a>]
exists in the linux source code since version 2.6 and a backend balloon device exists in
QEMU to facilitate ballooning. The virtio drivers can be installed additionally in a
Windows system.
<a
id="x6-7001r9"></a>
</p>

<h4 class="subsectionHead"><span class="titlemark">1.1.2   </span> <a
href="mainli2.html#QQ2-6-10" id="x6-80002">CPU Overcommitment</a></h4>
<p>CPU overcommitment happens through time sharing of CPU cores on a physical
machine by multiple VMs. The virtual CPU cores for a VM are called vCPU.
Most hypervisors allow multiple virtual machines to share same physical CPU
core for multiple vCPUs. QEMU-KVM runs one vCPU thread per guest CPU
[<a
href="mainli5.html#X0-qemu-multi" >10</a>]. The thread scheduling onto physical CPU is handled by the host operating
system. Time sharing CPU cores can incur a performance penalty, which is
measured by the time a vCPU spends in the ready queue, i.e. the time for which a
vCPU is ready to be scheduled but does not get scheduled. This time is called
the <em><span
class="cmti-12">steal time</span></em> as it represents the CPU cycles that have been stolen from the
vCPU. If the guests on a host experience high steal, the CPU of the host is
overloaded.
<a
id="x6-8001r8"></a>
</p>



<h3 class="sectionHead"><span class="titlemark">1.2   </span> <a
href="main.html#QQ2-6-11" id="x6-90002">Virtual Machine Live Migration</a></h3>
<p>VM live migration [<a
href="mainli5.html#X0-Clark:2005:LMV:1251203.1251223" >11</a>] is a technique to migrate a running VM from one host to another
without shutting it down. Live migration involves migrating the disk, memory and CPU
states of the running VM to the destination host and resuming the VM there. Migrating
disk can be as easy as just copying the disk to destination or using a ﬁle-system shared
over the network like Network File System (NFS). There are two popular techniques for
migrating the VM memory and state:
   </p>
<dl class="enumerate"><dt class="enumerate">
1. </dt><dd
class="enumerate"><strong><span
class="cmbx-12">Pre-Copy Live Migration.</span></strong> It follows an iterative page copying technique
   wherein ﬁrst, all the pages of the VM are copied to the destination. From
   next iteration onward, only the pages which were dirtied during the previous
   iteration are copied to the destination. This process continues till the page
   dirtying speed is less than the page transfer speed. Then the VM at the source
   is stopped, remaining dirty pages and VM state is copied to the destination,
   and the VM is resumed at the destination. QEMU-KVM uses pre-copy live
   migration [<a id="page.8"></a><a
href="mainli5.html#X0-qemu-migration" >12</a>].
   </dd><dt class="enumerate">
2. </dt><dd
class="enumerate"><strong><span
class="cmbx-12">Post-Copy Live Migration.</span></strong> The VM is stopped at the source, its state
   is copied to the destination, and the VM resumed there. The pages of the
   VM are transferred to the destination in background, with the pages that are
   immediately demanded by the VM via page fault given the highest priority in
   transfer. Thus, the performance of the VM is degraded before its working set
   is transferred.</dd></dl>
<p>   The migration time of a VM is the time taken to resume the VM on the
destination after triggering the migration. There is also a small amount of downtime
involved in live migration in which the VM is neither running on the host, nor on
the destination. Live migration of VMs connected to a network is especially
tricky because the new VM has to assigned the same IP address and all the


packets have to rerouted to the new destination without any delay to prevent
packet loss and downtime of any service running inside the VM which uses the
network.
</p>
<div class="table">


<p>   <a
id="x6-9003r1"></a></p>
<hr class="float" /><div class="float"
>


<a
id="x6-9004"></a>
<div class="caption"
><span class="id">Table 1.1: </span><span  
class="content">Diﬀerences between pre-copy and post-copy live migration</span></div><!--tex4ht:label?: x6-9003r1 -->
<div class="center"
>
<p>
<table id="TBL-4" class="tabular"
cellspacing="0" cellpadding="0"  
><colgroup id="TBL-4-1g"><col
id="TBL-4-1" /><col
id="TBL-4-2" /></colgroup><tr
class="hline"><td><hr /></td><td><hr /></td></tr><tr  
style="vertical-align:baseline;" id="TBL-4-1-"><td  style="text-align:left;" id="TBL-4-1-1"  
class="td11">                             </td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-4-2-"><td  style="text-align:left;" id="TBL-4-2-1"  
class="td11"> <p>                             </p>
</td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-4-3-"><td  style="text-align:left;" id="TBL-4-3-1"  
class="td11"> <p>Pre-Copy                                 </p>
</td><td  style="text-align:left;" id="TBL-4-3-2"  
class="td11"> <p>Post-Copy                               </p>
</td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-4-4-"><td  style="text-align:left;" id="TBL-4-4-1"  
class="td11">                              </td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-4-5-"><td  style="text-align:left;" id="TBL-4-5-1"  
class="td11"> <p>                             </p>
</td>
</tr><tr
class="hline"><td><hr /></td><td><hr /></td></tr><tr  
style="vertical-align:baseline;" id="TBL-4-6-"><td  style="text-align:left;" id="TBL-4-6-1"  
class="td11">                              </td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-4-7-"><td  style="text-align:left;" id="TBL-4-7-1"  
class="td11"> <p>                             </p>
</td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-4-8-"><td  style="text-align:left;" id="TBL-4-8-1"  
class="td11"> <p>Total  migration  time  =  (RAM
size/link  speed)  +  overhead  +
non-deterministic (depending on
dirtying pattern)                       </p>
</td><td  style="text-align:left;" id="TBL-4-8-2"  
class="td11"> <p>Total migration Time = (RAM
size/link speed) + overhead        </p>
</td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-4-9-"><td  style="text-align:left;" id="TBL-4-9-1"  
class="td11"> <p>                             </p>
</td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-4-10-"><td  style="text-align:left;" id="TBL-4-10-1"  
class="td11"> <p>Worst  downtime  =  (VM  state
time) + (RAM size/link speed)    </p>
</td><td  style="text-align:left;" id="TBL-4-10-2"  
class="td11"> <p>Worst  downtime  =  VM  state
time                                       </p>
</td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-4-11-"><td  style="text-align:left;" id="TBL-4-11-1"  
class="td11"> <p>                             </p>
</td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-4-12-"><td  style="text-align:left;" id="TBL-4-12-1"  
class="td11"> <p>It  can  cope  with  network  or
system failure.                          </p>
</td><td  style="text-align:left;" id="TBL-4-12-2"  
class="td11"> <p>In  case  of  network  or  system
failure, VM cannot be recovered.  </p>
</td>
</tr><tr
class="hline"><td><hr /></td><td><hr /></td></tr><tr  
style="vertical-align:baseline;" id="TBL-4-13-"><td  style="text-align:left;" id="TBL-4-13-1"  
class="td11"> <p>                             </p>
</td></tr></table></div>


</div><hr class="endfloat" />
</div>
<a
id="x6-9005r11"></a>
<h3 class="sectionHead"><span class="titlemark">1.3   </span> <a
href="main.html#QQ2-6-12" id="x6-100003">Resource Management in Cloud</a></h3>
<p>Resource management is an essential technique to utilize the underlying hardware of the
cloud eﬃciently. The role of the resource manager is to manage the allocation of physical
resources to the virtual machines deployed on a cluster of nodes in a cloud. Diﬀerent
resource management systems may have diﬀerent aims depending upon the needs. For a
private cloud like in an educational institution, the most common aim might be to
maximize performance of the virtual machines while minimizing the operational costs of
the cloud infrastructure.
</p>
<p>   Minimizing operational costs involves minimizing the number of physical machines
used. This can be achieved through overcommitment of resources. Resource
overcommitment comes with some new problems like hotspot elimination and where to
schedule new incoming VMs to minimize chances of hotspot. If the total capacity of the
virtual machines running on a physical machine is more than the total capacity of the
physical machines, a situation may arise wherein the VMs may want to use a sum total of
more resources than are present. Not satisfying those resource requirements may lead to
violation of SLAs and poor performance of the VMs. This situation is called a hotspot. A
distributed resource scheduler (DRS) is a piece of software that runs on diﬀerent nodes in
the cluster and handles dynamic resource allocation to diﬀerent VMs in the
cluster.
</p>
<p>   Memory ballooning and live migration of VM can help in mitigating hotspots. The
basic idea is that if a VM is short on memory, ballooning can be used to take away some
memory from another guest on the same host which has some free memory and give it to
the needy guest. If none of the guests have any free memory, the host is overloaded. A
guest has to be migrated from this host to another host while taking into account the
overall load of the cluster. This might sound simple, but there are several challenges


involved in this process. Some of the challenges are determining the amount of free
memory a VM can give away without aﬀecting its own performance, determining whether
beneﬁts of migration are more than performance loss, selecting which virtual
machine to migrate such that maximum beneﬁt is achieved out of the migration,
selecting destination host to minimize the chances of future migrations, ﬁltering
intermittent spikes from resource usage graph of VMs to determine their actual load
proﬁle and distributed monitoring of VMs which can scale to a large number of
machines. In this thesis, we address large scale monitoring and ballooning of virtual
machines.
<a
id="x6-10001r12"></a>
</p>

<h3 class="sectionHead"><span class="titlemark">1.4   </span> <a
href="main.html#QQ2-6-13" id="x6-110004">Organization of this Thesis </a></h3>
<p>The rest of the thesis is organized as follows. Chapter 2 describes related work that has
been done in this area. Chapter 3 consolidates the requirements of a DRS, outlines our
approach to building the DRS, and proposes a decentralized and scalable architecture for
it. Chapter 4 describes the details of implementation of the monitoring and
autoballooning components of our DRS. Chapter 5 analyses the monitoring and
autoballooning components of the DRS through experimental data. Finally, Chapter 6
concludes the work done in this thesis and provides some insights on possible extensions
to this work.
<a
id="Q1-6-14"></a>
</p>

<h4 class="likesubsectionHead"><a
href="#x6-120004" id="x6-120004">Summary</a></h4>
<p>In this chapter, we discussed the diﬀerent technologies behind cloud computing and
looked at how they are used. We have looked at how resource overcommitment helps in
utilizing the hardware eﬃciently and why resource management is essential for
overcommitment to work. We have also identiﬁed several problems and challenges related
to resource management which have been resolved in this thesis.


</p>
<!--l. 1--><div class="crosslinks"><p class="noindent">[<a
href="mainch2.html" >next</a>] [<a
href="mainli4.html" >prev</a>] [<a
href="mainli4.html#tailmainli4.html" >prev-tail</a>] [<a
href="mainch1.html" >front</a>] [<a
href="main.html#mainch1.html" >up</a>] </p></div>
<p>   <a
id="tailmainch1.html"></a>        </p>
{{< /raw >}}
