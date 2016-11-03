+++
title = "Architecture and Design of DRS"
menu = "main"
url = "thesis/mainch3.html"
+++

{{< raw >}}
<div class="crosslinks"><p class="noindent">[<a
href="mainch4.html" >next</a>] [<a
href="mainch2.html" >prev</a>] [<a
href="mainch2.html#tailmainch2.html" >prev-tail</a>] [<a
href="#tailmainch3.html">tail</a>] [<a
href="main.html#mainch3.html" >up</a>] </p></div>
<h2 class="chapterHead"><span class="titlemark">Chapter 3</span><br /><a
href="main.html#QQ2-8-35" id="x8-310003">Architecture and Design of DRS</a></h2>
<a
id="x8-31001r32"></a>
<h3 class="sectionHead"><span class="titlemark">3.1   </span> <a
href="main.html#QQ2-8-36" id="x8-320001">Functions of DRS</a></h3>
<p>Distributed resource scheduler is responsible for handling the dynamic resource allocation
requirements of VMs on the cloud. The job of a DRS can be divided into three
categories:
   </p>
<dl class="enumerate"><dt class="enumerate">
1. </dt><dd
class="enumerate"><strong><span
class="cmbx-12">Monitoring.</span></strong> Monitoring is essential for detecting the resource requirements
   of individual guest machines running on the cloud and detecting hotspots on
   physical hosts which make up the cluster. It has to be ﬁne-grained enough to
   detect changes in the load proﬁle of the hosts and guests. It has to be coarse
   enough to ﬁlter short-terms changes in the resource requirements of the guests.
   It will also regularly update the load statistics of the host in a shared data
   store which can be used to get a global account of the cluster state at any
   point of time which will be useful in making decisions about live-migration.
   </dd><dt class="enumerate">
2. </dt><dd
class="enumerate"><strong><span
class="cmbx-12">Memory Management.</span></strong> For all the guests running on the same host, DRS
   will  manage  their  ever  changing  memory  demands.  Autoballooning  is  an
   essential technique for the DRS to perform this aspect of its functionality. This
   part of DRS will look at the memory usage statistics of all the guests, ﬁgure
   out which VMs need more memory and which VMs have idle memory, and try
   to meet the demands of each VM by inﬂating/deﬂating its balloon.
   </dd><dt class="enumerate">
3. </dt><dd
class="enumerate"><strong><span
class="cmbx-12">Hotspot Mitigation.</span></strong> In the case of a hotspot, this part of DRS is responsible
   for selecting an appropriate guest to migrate and the best destination for it
   based on the resource usage data of all the machines present in the cluster.</dd></dl>


<a
id="x8-32004r36"></a>
<h3 class="sectionHead"><span class="titlemark">3.2   </span> <a
href="main.html#QQ2-8-37" id="x8-330002">Goals and Non-Goals</a></h3>
<p><em><span
class="cmti-12">Goal:</span></em> Develop a DRS for private clouds which makes the best use of the resources
available i.e. provides the resources to the machines on a best eﬀort basis according to the
priority/importance of each VM. If the cluster is not fully loaded, the demands of all the
VMs can be satisﬁed but if the load is more than the capacity, the VMs will have
to do with less resources than promised. In case of an overload, all the VMs
with the same priority across the cluster should be treated equally in resource
allocation.
</p>
<p>   <em><span
class="cmti-12">Goal:</span></em> The DRS design should be decentralized so that it is easily able to scale up to a
cluster having thousands of physical hosts and there is no single point of failure in the
system.
</p>
<p>   <em><span
class="cmti-12">Non-goal:</span></em> Minimizing the number of machines used. Though using less machines
will save power because the unused machines can be turned oﬀ, the goals of
power management are orthogonal to the goals of our DRS and can be treated
separately.
</p>
<p>   <em><span
class="cmti-12">Non-goal:</span></em> For the purpose of this thesis, we have considered balancing only memory
and CPU resources. Other resources like disk I/O, and network bandwidth have not been
taken into account.
<a
id="x8-33001r37"></a>
</p>

<h3 class="sectionHead"><span class="titlemark">3.3   </span> <a
href="main.html#QQ2-8-38" id="x8-340003">Architecture Overview</a></h3>
<p>The DRS architecture described below is fully decentralized, which is a key requirement
for scaling it up to thousands of hosts. Decentralization also prevents any single point of
failures. Each host in the cluster makes its own decision about the guest machines on that
host and the other hosts co-operate with it to make that decision successful. The DRS is
divided into three components based on the three types of job mentioned earlier that it
has to perform.


</p>
<hr class="figure" /><div class="figure"
>


<a
id="x8-34001r1"></a>



<p><img
src="/images/thesis/arch.png" alt="PIC"  
/>
<a
id="x8-34002"></a>
<br /> </p>
<div class="caption"
><span class="id">Figure 3.1: </span><span  
class="content">Architecture of the DRS</span></div><!--tex4ht:label?: x8-34001r3 -->


</div><hr class="endfigure" />
<a
id="x8-34003r31"></a>
<h4 class="subsectionHead"><span class="titlemark">3.3.1   </span> <a
href="mainli2.html#QQ2-8-39" id="x8-350001">The Monitoring Service</a></h4>
<p>A <em><span
class="cmti-12">monitoring service</span></em> runs on all the hosts of the cluster. On each host, it collects the
resource usage statistics of all the VMs on the host and the total resource usage of the
host. It also ﬁlters out the short-term changes in the resource usage and ﬁgures
out the changes in the load proﬁle. It also detects any hotspot and triggers the
migration service to perform its task of migrating a guest away from the host. The
service also accommodates (start monitoring) any new guest that is migrated to a
host.
</p>
<p>   The service talks to a distributed key-value store to regularly update the resource
usage statistics of the host it is running on. The key-value store should be updated only
when there is a signiﬁcant change in the resource usage proﬁles of the host, which could
aﬀect the decision making of live migration. Small changes in the resource usage are
unnecessary to track as they will not aﬀect the decision making. Short-term spikes in the
resource usage can adversely aﬀect the live-migration decisions by misrepresenting the
resource usage.
<a
id="x8-35001r39"></a>
</p>

<h4 class="subsectionHead"><span class="titlemark">3.3.2   </span> <a
href="mainli2.html#QQ2-8-40" id="x8-360002">The Memory Balancing Service</a></h4>
<p>A <em><span
class="cmti-12">memory balancing Service</span></em> runs on all the hosts. On each host, the service looks at the
memory usage statistics of all the VMs running on the host provided by the monitoring
service. It ﬁgures out which hosts require more memory and which hosts have some idle
memory. It then balances the memory by inﬂating/deﬂating the balloons of the
VMs.
</p>
<p>   If the demands of all the VMs can be satisﬁed, balancing takes idle memory and gives
it to the needy. If the demands of all the VMs cannot be satisﬁed i.e. there is a hotspot on
the host, the service calculates the entitled memory based on the priority of each


VM running on the host for each guest. Entitled memory may be less than the
memory needs of a guest. Then the service distributes memory according to the
entitlement.
</p>
<p>   The service also tries to make space by freeing memory for any new guest that is to
be migrated to the host. Notably, this service does not do anything to resolve
hotspots. It just tries to balance memory amongst the guests present on the
host. Hotspot mitigation is left to the migration service. If there is a hotspot,
some guest(s) may get migrated out of the host, leaving free memory for other
VMs.
</p>
<p>   The memory management requires a separate service while CPU management does
not. There are two reasons for this.
   </p>
<dl class="enumerate"><dt class="enumerate">
1. </dt><dd
class="enumerate">vCPU of each VM is handled as a thread by the hypervisor and hence scheduled
   according to its process scheduling policy, while there is no such system for
   memory.
   </dd><dt class="enumerate">
2. </dt><dd
class="enumerate">Memory,  once  consumed  by  a  VM,  is  not  reclaimed  automatically  by  the
   hypervisor, while this is not the case with the CPU resources. Each vCPU
   thread gets a time slice to run which is decided by the hypervisor and the
   CPU is taken away from the thread after the time slice ends.
   </dd></dl>
<a
id="x8-36003r40"></a>
<h4 class="subsectionHead"><span class="titlemark">3.3.3   </span> <a
href="mainli2.html#QQ2-8-41" id="x8-370003">The Migration Service</a></h4>
<p>This service runs on all the hosts and is triggered by the <em><span
class="cmti-12">monitoring service</span></em> when there is
a hotspot on the host. It collects the resource usage statistics of all the other hosts in the
cluster from the data-store to which the monitoring services of all the hosts
write. This data would be just a few key-value pairs per host and hence, very
small. Using this data (and some other), it tries to ﬁnd out the best migration


(guest VM-destination host pair) performing which will improve some metric
representing the resource requirement satisfaction or the overall performance
of the VMs on the cluster. The service talks to the memory balancing service
on the destination host to free required memory for the incoming guest, and
then it migrates the guest to the destination. The migration service requires
resource usage details of the entire cluster to make an informed decision on
migration.
<a
id="Q1-8-42"></a>
</p>

<h4 class="likesubsectionHead"><a
href="#x8-380003" id="x8-380003">Summary</a></h4>
<p>In this chapter, we have discussed the functions and goals of a DRS. We have also
proposed a decentralized architecture for DRS which can scale well and is free of single
point failures.


</p>
<!--l. 1--><div class="crosslinks"><p class="noindent">[<a
href="mainch4.html" >next</a>] [<a
href="mainch2.html" >prev</a>] [<a
href="mainch2.html#tailmainch2.html" >prev-tail</a>] [<a
href="mainch3.html" >front</a>] [<a
href="main.html#mainch3.html" >up</a>] </p></div>
<p>   <a
id="tailmainch3.html"></a>        </p>
{{< /raw >}}
