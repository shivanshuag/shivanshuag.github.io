+++
title = "Experimental Evaluation"
menu = "main"
url = "thesis/mainch5.html"
+++
{{< raw >}}
<div class="crosslinks"><p class="noindent">[<a
href="mainch6.html" >next</a>] [<a
href="mainch4.html" >prev</a>] [<a
href="mainch4.html#tailmainch4.html" >prev-tail</a>] [<a
href="#tailmainch5.html">tail</a>] [<a
href="main.html#mainch5.html" >up</a>] </p></div>
<h2 class="chapterHead"><span class="titlemark">Chapter 5</span><br /><a
href="main.html#QQ2-10-58" id="x10-520005">Experimental Evaluation</a></h2>
<p>We have built our DRS for the QEMU-KVM hypervisor. It uses the <em><span
class="cmti-12">libvirt</span></em> APIs for
managing the virtual machines and <em><span
class="cmti-12">Openstack</span></em> [<a id="page.69"></a><a
href="mainli5.html#X0-openstack" >26</a>] mainly for cloud management and
software deﬁned networking along with live-migration support. For the distributed
key-value store, we use <em><span
class="cmti-12">etcd</span></em> [<a
href="mainli5.html#X0-etcd" >27</a>]. We have conducted several experiments to determine the
eﬀectiveness of our DRS. The experiments relating to monitoring and auto-ballooning
aspect of the DRS have been described below.
<a
id="x10-52001r53"></a>
</p>

<h3 class="sectionHead"><span class="titlemark">5.1   </span> <a
href="main.html#QQ2-10-59" id="x10-530001">Experimental Setup</a></h3>
<p>Our experimental setup consists of six physical machines. Each physical machine has an 8
core Intel Core i7 3.5GHz CPU, 16 GB memory, 1Gbps NIC and runs the 64 bit Ubuntu
14.04 with Linux kernel version 3.19 . We have installed Openstack on these 6 nodes such
that one of the nodes is the controller node(runs the Openstack management services)
and the other ﬁve are the compute nodes (run the actual VMs). The nodes are
connected to a gigabit local network, which they use for transferring data while live
migration and for communicating with the outside network. The disks of all the
VMs reside on a shared NFS server, and hence, live migration needs to just
transfer the memory of the VM between the hosts, and not the disks. Each of the
compute nodes also run the DRS software created by us. The controller and
two separate nodes (separate from the controller and the compute nodes) run
etcd, with a cluster size of three, which provides a fault tolerance of degree one
[<a
href="mainli5.html#X0-etcd-ad" >28</a>]. All the VMs that we use run 64 bit Ubuntu 12.04 cloud image. The VMs
can be of two sizes - 1 vCPU with 2 GB RAM(small) or 2 vCPU with 4 GB
RAM(large).
</p>
<p>   There are three types of workloads run by these VMs. One workload is memory
intensive, one is CPU intensive and one of the workloads is a mix of the two. The


memory intensive workload is a program written by us which consumes a total
of 1800MB. The program runs in two phases - the allocation phase and the
retention phase. The allocation phase starts when the program starts. In the
allocation phase, the program tries to allocate 100MB memory using <em><span
class="cmti-12">malloc</span></em> and
then sleeps for two seconds. This step is performed iteratively till the allocated
memory has reached 1800MB. Notably, it may take more than 18 iterations for the
allocation to reach 1800MB because malloc will return <em><span
class="cmti-12">null</span></em> if it cannot allocate
memory due to shortage of memory and the program will sleep for two more
seconds. So, the length of the allocation phase depends on the availability of
memory. After the allocation phase, retention phase starts, where the program
retains the allocated memory for 300 seconds, and does no more allocations.
After the retention phase, the program ends. We will refer to this workload as
<em><span
class="cmti-12">usemem</span></em>.
</p>
<p>   For the other two types of workloads, we have chosen two SPEC CPU 2006 V1.2
benchmarks [<a id="page.70"></a><a
href="mainli5.html#X0-Henning:2006:SCB:1186736.1186737" >29</a>]. For CPU intensive workload, we run the <em><span
class="cmti-12">libquantum</span></em> benchmark. The
libquantum benchmark tries to solve certain computationally hard problems using the
simulation of a quantum computer. At runtime, the benchmark consumes 100% of a
vCPU and about 50MB memory. For the CPU and memory intensive workload, we
use the <em><span
class="cmti-12">mcf </span></em> benchmark. The mcf benchmark solves the single-depot vehicle
scheduling problem in the planning process of public transportation companies
using the network simplex algorithm accelerated with a column generation. At
runtime, the mcf benchmark consumes 100% of a vCPU and about 1800MB
memory.
</p>
<p>   Large VMs run two workloads in two separate threads simultaneously, while the
small VMs runs only one workload in a single thread. Each thread randomly
chooses a workload from the three workloads described above, runs it and
calculates the time it took to run the workload, sleeps for a randomly chosen
time between 0 and 90 seconds, and then repeats this process. Each compute
node has four large VMs and three small VMs. Keeping aside one core and


two GB memory for the hypervisor, this gives us an overcommitment ratio of
<!--l. 12--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mi>1.57</mi></math> for
both CPU and memory.
<a
id="x10-53001r59"></a>
</p>

<h3 class="sectionHead"><span class="titlemark">5.2   </span> <a
href="main.html#QQ2-10-60" id="x10-540002">Results</a></h3>
<p>The experiment that we ran was to determine the eﬀectiveness of autoballooning in
memory overcommitment. For this, we monitored two hosts named compute2 and
compute3 respectively, running seven VMs as described in the previous section. We
disabled live-migration in the DRS on the both hosts. On compute3, autoballoning was
also disabled. We compare the results obtained after running the experiment for about 17
hours.
<a
id="x10-54001r55"></a>
</p>

<h4 class="subsectionHead"><span class="titlemark">5.2.1   </span> <a
href="mainli2.html#QQ2-10-61" id="x10-550001">Analyzing Auto-Ballooning</a></h4>
<p>Figure <a
href="#x10-55001r1">5.1<!--tex4ht:ref: fig:mem --></a> shows the diﬀerent memory metrics of compute2 and compute3 hosts
plotted against time. From these graphs, the most remarkable diﬀerence that we
can see is in the swap memory on both the machines. On compute3, the swap
memory rose to very high levels of about 7GB, which is equal to the amount of
memory we have overcommited. On compute 2, the swap memory remained
very low throughout the experiment, remaining below 1GB most of the time
and never going above 1.5GB. On compute3, the total used memory remains
almost constant and equal to maximum memory, while it keeps on ﬂuctuating
on compute2. This is because memory, once allocated, cannot be reclaimed on
compute3.


</p>
<p>   <a
id="x10-55001r1"></a></p>
<hr class="float" /><div class="float"
>



<img
src="/images/thesis/mem-compute3.png" alt="PIC"  
/> <img
src="/images/thesis/mem-compute2.png" alt="PIC"  
/>
<a
id="x10-55002"></a>
<br /> <div class="caption"
><span class="id">Figure 5.1: </span><span  
class="content">Graphs showing diﬀerent memory metrics of compute3 (autoballooning
disabled) and compute 2 (autoballooning enabled). X-axis represents the time at
which the value was recorded, Y-axis shows the value.</span></div><!--tex4ht:label?: x10-55001r5 -->


</div><hr class="endfloat" />
<p>   Figure <a
href="#x10-55003r2">5.2<!--tex4ht:ref: fig:cpu --></a> shows the CPU usage of compute2 and compute3 plotted against
time. In the graph, a few hours after starting the experiment, the CPU usage of
compute3 is consistently low, while compute2 makes better use of the CPU. This is
because of high levels of swap on compute 3. The mcf and usemem workloads
spend more time in performing I/O and hence are not able to utilize the CPU
eﬃciently.


</p>
<p>   <a
id="x10-55003r2"></a></p>
<hr class="float" /><div class="float"
>



<img
src="/images/thesis/cpu-compute3.png" alt="PIC"  
/> <img
src="/images/thesis/cpu-compute2.png" alt="PIC"  
/>
<a
id="x10-55004"></a>
<br /> <div class="caption"
><span class="id">Figure 5.2: </span><span  
class="content">Graphs showing cpu usage of compute3 (autoballooning disabled) and
compute2 (autoballooning enabled). X-axis represents the time at which the value
was recorded, Y-axis shows the value.</span></div><!--tex4ht:label?: x10-55003r5 -->


</div><hr class="endfloat" />
<p>   Table <a
href="#x10-55005r1">5.1<!--tex4ht:ref: tab:count --></a> lists the number of time each workload ran on both the machines.
Libquantum and mcf are CPU intensive. On compute2, CPU intensive workloads ran 303
times compared to 287 times on compute3 in the same time interval. Mcf and usemem are
memory intensive. On compute2, memory intensive workloads ran 357 times compared to
289 times on compute 3 in the same time interval. It clearly shows that there was better
utilization of the CPU and memory resources on compute2 resulting in a better overall
throughput.
</p>
<div class="table">


<p>   <a
id="x10-55005r1"></a></p>
<hr class="float" /><div class="float"
>


<a
id="x10-55006"></a>
<div class="caption"
><span class="id">Table 5.1: </span><span  
class="content">Number of times each workload ran during the experiment</span></div><!--tex4ht:label?: x10-55005r5 -->
<div class="center"
>
<p>
<table id="TBL-11" class="tabular"
cellspacing="0" cellpadding="0"  
><colgroup id="TBL-11-1g"><col
id="TBL-11-1" /><col
id="TBL-11-2" /><col
id="TBL-11-3" /></colgroup><tr
class="hline"><td><hr /></td><td><hr /></td><td><hr /></td></tr><tr  
style="vertical-align:baseline;" id="TBL-11-1-"><td  style="text-align:left;" id="TBL-11-1-1"  
class="td11">                   </td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-11-2-"><td  style="text-align:left;" id="TBL-11-2-1"  
class="td11"> <p>                  </p>
</td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-11-3-"><td  style="text-align:left;" id="TBL-11-3-1"  
class="td11"> <p>Workload                </p>
</td><td  style="text-align:left;" id="TBL-11-3-2"  
class="td11"> <p>Count-compute3      </p>
</td><td  style="text-align:left;" id="TBL-11-3-3"  
class="td11"> <p>Count-compute2      </p>
</td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-11-4-"><td  style="text-align:left;" id="TBL-11-4-1"  
class="td11">                   </td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-11-5-"><td  style="text-align:left;" id="TBL-11-5-1"  
class="td11"> <p>                  </p>
</td>
</tr><tr
class="hline"><td><hr /></td><td><hr /></td><td><hr /></td></tr><tr  
style="vertical-align:baseline;" id="TBL-11-6-"><td  style="text-align:left;" id="TBL-11-6-1"  
class="td11">                   </td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-11-7-"><td  style="text-align:left;" id="TBL-11-7-1"  
class="td11"> <p>                  </p>
</td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-11-8-"><td  style="text-align:left;" id="TBL-11-8-1"  
class="td11"> <p>libquantum              </p>
</td><td  style="text-align:left;" id="TBL-11-8-2"  
class="td11"> <p>153                        </p>
</td><td  style="text-align:left;" id="TBL-11-8-3"  
class="td11"> <p>198                        </p>
</td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-11-9-"><td  style="text-align:left;" id="TBL-11-9-1"  
class="td11"> <p>mcf                        </p>
</td><td  style="text-align:left;" id="TBL-11-9-2"  
class="td11"> <p>134                        </p>
</td><td  style="text-align:left;" id="TBL-11-9-3"  
class="td11"> <p>105                        </p>
</td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-11-10-"><td  style="text-align:left;" id="TBL-11-10-1"  
class="td11"> <p>usemem                   </p>
</td><td  style="text-align:left;" id="TBL-11-10-2"  
class="td11"> <p>155                        </p>
</td><td  style="text-align:left;" id="TBL-11-10-3"  
class="td11"> <p>252                        </p>
</td>
</tr><tr
class="hline"><td><hr /></td><td><hr /></td><td><hr /></td></tr><tr  
style="vertical-align:baseline;" id="TBL-11-11-"><td  style="text-align:left;" id="TBL-11-11-1"  
class="td11"> <p>                  </p>
</td></tr></table></div>


</div><hr class="endfloat" />
</div>


<p>   <a
id="x10-55007r3"></a></p>
<hr class="float" /><div class="float"
>



<img
src="/images/thesis/time-compute3.png" alt="PIC"  
/> <img
src="/images/thesis/time-compute2.png" alt="PIC"  
/>
<a
id="x10-55008"></a>
<br /> <div class="caption"
><span class="id">Figure 5.3: </span><span  
class="content">Graphs showing the time taken by diﬀerent workloads on the two hosts.
The values greater than two hours have been ﬁltered out from the compute3 graph
for the sake of visibility. X-axis represents the time at which the value was recorded,
Y-axis shows the value.</span></div><!--tex4ht:label?: x10-55007r5 -->


</div><hr class="endfloat" />
<div class="table">


<p>   <a
id="x10-55009r2"></a></p>
<hr class="float" /><div class="float"
>


<a
id="x10-55010"></a>
<div class="caption"
><span class="id">Table 5.2: </span><span  
class="content">Mean time taken by each workload to run</span></div><!--tex4ht:label?: x10-55009r5 -->
<div class="center"
>
<p>
<table id="TBL-15" class="tabular"
cellspacing="0" cellpadding="0"  
><colgroup id="TBL-15-1g"><col
id="TBL-15-1" /><col
id="TBL-15-2" /><col
id="TBL-15-3" /></colgroup><tr
class="hline"><td><hr /></td><td><hr /></td><td><hr /></td></tr><tr  
style="vertical-align:baseline;" id="TBL-15-1-"><td  style="text-align:left;" id="TBL-15-1-1"  
class="td11">                   </td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-15-2-"><td  style="text-align:left;" id="TBL-15-2-1"  
class="td11"> <p>                  </p>
</td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-15-3-"><td  style="text-align:left;" id="TBL-15-3-1"  
class="td11"> <p>Workload                </p>
</td><td  style="text-align:left;" id="TBL-15-3-2"  
class="td11"> <p>Mean-compute3       </p>
</td><td  style="text-align:left;" id="TBL-15-3-3"  
class="td11"> <p>Mean-compute2       </p>
</td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-15-4-"><td  style="text-align:left;" id="TBL-15-4-1"  
class="td11">                   </td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-15-5-"><td  style="text-align:left;" id="TBL-15-5-1"  
class="td11"> <p>                  </p>
</td>
</tr><tr
class="hline"><td><hr /></td><td><hr /></td><td><hr /></td></tr><tr  
style="vertical-align:baseline;" id="TBL-15-6-"><td  style="text-align:left;" id="TBL-15-6-1"  
class="td11">                   </td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-15-7-"><td  style="text-align:left;" id="TBL-15-7-1"  
class="td11"> <p>                  </p>
</td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-15-8-"><td  style="text-align:left;" id="TBL-15-8-1"  
class="td11"> <p>libquantum              </p>
</td><td  style="text-align:left;" id="TBL-15-8-2"  
class="td11"> <p>14.5 min                 </p>
</td><td  style="text-align:left;" id="TBL-15-8-3"  
class="td11"> <p>23.7 min                 </p>
</td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-15-9-"><td  style="text-align:left;" id="TBL-15-9-1"  
class="td11"> <p>mcf                        </p>
</td><td  style="text-align:left;" id="TBL-15-9-2"  
class="td11"> <p>39.6 min                 </p>
</td><td  style="text-align:left;" id="TBL-15-9-3"  
class="td11"> <p>20.3 min                 </p>
</td>
</tr><tr  
style="vertical-align:baseline;" id="TBL-15-10-"><td  style="text-align:left;" id="TBL-15-10-1"  
class="td11"> <p>usemem                   </p>
</td><td  style="text-align:left;" id="TBL-15-10-2"  
class="td11"> <p>12.8 min                 </p>
</td><td  style="text-align:left;" id="TBL-15-10-3"  
class="td11"> <p>7.9 min                  </p>
</td>
</tr><tr
class="hline"><td><hr /></td><td><hr /></td><td><hr /></td></tr><tr  
style="vertical-align:baseline;" id="TBL-15-11-"><td  style="text-align:left;" id="TBL-15-11-1"  
class="td11"> <p>                  </p>
</td></tr></table></div>


</div><hr class="endfloat" />
</div>
<p>   The graphs in Figure <a
href="#x10-55007r3">5.3<!--tex4ht:ref: fig:time --></a> show the time it took for the workloads to run plotted
against the time at which the workload completed. In the graph for compute3, we can see
that the time to complete the memory intensive workloads - mcf and usemem can grow to
more than two hours while it always remains below 33 minutes on compute2.
On top of this, some of the very large values have been ﬁltered out from the
graph of compute3. There were three such values for the mcf benchmark, which
were greater than 9 hours. On the contrary, the libquantum workload performs
better on compute3. This is because libquantum does not require much memory
and the CPU on compute3 is relatively free because the other workloads do
not utilize it well. Table <a
href="#x10-55009r2">5.2<!--tex4ht:ref: tab:mean --></a> shows the mean time it took for the workloads
to run. As expected, the libquantum performs better on compute3 while the
other workloads perform better on compute2. But mean time is not a good
metric to compare the eﬀectiveness of autoballooning. Throughput is a better
metric.
<a
id="x10-55011r61"></a>
</p>

<h4 class="subsectionHead"><span class="titlemark">5.2.2   </span> <a
href="mainli2.html#QQ2-10-62" id="x10-560002">Analyzing CPU Hotspot Detection</a></h4>


<p>   <a
id="x10-56001r4"></a></p>
<hr class="float" /><div class="float"
>



<img
src="/images/thesis/cpu-migration.png" alt="PIC"  
/>
<a
id="x10-56002"></a>
<br /> <div class="caption"
><span class="id">Figure 5.4: </span><span  
class="content">CPU usage of compute2 during one hour of the experiment. Blue dots
show the points at which migration was triggered. X-axis represents the time at
which the value was recorded, Y-axis shows the value in %</span></div><!--tex4ht:label?: x10-56001r5 -->


</div><hr class="endfloat" />
<p>The graph in Figure <a
href="#x10-56001r4">5.4<!--tex4ht:ref: fig:cpu-mig --></a> shows the CPU usage of compute2 for one hour during the
experiment. The blue dots represent the instances at which migration was triggered.
Migration was disabled for this experiment, so no machines was actually migrated out of
this host. In the graph, the CPU usage is 100% in two intervals. First interval is from
around time 3:47 to 4:06 (interval1) and the second interval is from around 4:21 to 4:38
(interval2). However, migration is triggered only during interval1. This implies that
there was a hotspot due to CPU utilization only during interval1 and not during
interval2.
</p>
<hr class="figure" /><div class="figure"
>


<a
id="x10-56003r5"></a>



<p><img
src="/images/thesis/cpu-big1.png" alt="PIC"  
/> <img
src="/images/thesis/cpu-big2.png" alt="PIC"  
/> <img
src="/images/thesis/cpu-big3.png" alt="PIC"  
/> <img
src="/images/thesis/cpu-big4.png" alt="PIC"  
/>
<a
id="x10-56004"></a>
<br /> </p>
<div class="caption"
><span class="id">Figure 5.5: </span><span  
class="content">CPU usage of all the large VMs on compute2 during 1 hour of the
experiment. X-axis represents the time at which the value was recorded, Y-axis
shows the value in %</span></div><!--tex4ht:label?: x10-56003r5 -->


</div><hr class="endfigure" />
<hr class="figure" /><div class="figure"
>


<a
id="x10-56005r6"></a>



<p><img
src="/images/thesis/cpu-small1.png" alt="PIC"  
/> <img
src="/images/thesis/cpu-small2.png" alt="PIC"  
/> <img
src="/images/thesis/cpu-small3.png" alt="PIC"  
/>
<a
id="x10-56006"></a>
<br /> </p>
<div class="caption"
><span class="id">Figure 5.6: </span><span  
class="content">CPU usage of all the small VMs on compute2 during 1 hour of the
experiment. X-axis represents the time at which the value was recorded, Y-axis
shows the value in %</span></div><!--tex4ht:label?: x10-56005r5 -->


</div><hr class="endfigure" />
<p>   The graphs in Figure <a
href="#x10-56003r5">5.5<!--tex4ht:ref: fig:cpu-large --></a> and <a
href="#x10-56005r6">5.6<!--tex4ht:ref: fig:cpu-small --></a> show the CPU usage and steal time of individual
virtual machines on compute2 during that one hour. For a large virtual machines, 100%
CPU usage implies that it is using two vCPUs completely and 50% CPU usage implies
that it is using only one of its two vCPUs. For a small virtual machines, 100% CPU
usage means that it is using its only vCPU completely. From the graphs, we
can see that during interval1, large virtual machines are trying to use 6 vCPUs
and the small virtual machines are trying to use 3 vCPUs, which is a total of 9
vCPUs. The host has only 8 physical CPUs and hence, it is overloaded. This
also reﬂected in the steal time of the individual VMs which is higher than 10%
during interval1. During interval2, a total of 8 vCPUs are being used, which
is equal to the number of the physical CPUs, and hence there is no overload.
The steal time of the VMs are also low and migration is not triggered. These
observations show that steal time is an appropriate metric for determining CPU
hotspots.
<a
id="x10-56007r62"></a>
</p>

<h4 class="subsectionHead"><span class="titlemark">5.2.3   </span> <a
href="#x10-570003" id="x10-570003">Analyzing the CUSUM algorithm</a></h4>
<p>Figure <a
href="#x10-57001r7">5.7<!--tex4ht:ref: fig:etcd --></a> shows a graph of one hour time duration of the experiment. The red dots in
the graph mark the diﬀerent points at which etcd was updated with the value of used
memory for the host compute2. It is evident from the graph that the algorithm is
successful in ﬁltering sudden changes in the value of load memory and only updates etcd
when the load proﬁle has changed. In the one hour time duration, etcd was updated 13
times. Without the ﬁltering algorithm, etcd would have been updated once in every 10
seconds i.e. 360 time in an hour, which is about 28 times more than the ﬁltering
algorithm. Overall during the 17 hour run of the experiment, etcd was updated just 266
times.


</p>
<p>   <a
id="x10-57001r7"></a></p>
<hr class="float" /><div class="float"
>



<img
src="/images/thesis/etcd-mem.png" alt="PIC"  
/>
<a
id="x10-57002"></a>
<br /> <div class="caption"
><span class="id">Figure 5.7: </span><span  
class="content">Graphs showing the points at which etcd is updated. X-axis represents
the time at which the value was recorded, Y-axis shows the value.</span></div><!--tex4ht:label?: x10-57001r5 -->


</div><hr class="endfloat" />
<a
id="x10-57003r63"></a>
<h4 class="subsectionHead"><span class="titlemark">5.2.4   </span> <a
href="#x10-580004" id="x10-580004">Memory and CPU Footprint of DRS</a></h4>
<hr class="figure" /><div class="figure"
>


<a
id="x10-58001r8"></a>



<p><img
src="/images/thesis/mem-self.png" alt="PIC"  
/> <img
src="/images/thesis/cpu-self.png" alt="PIC"  
/>
<a
id="x10-58002"></a>
<br /> </p>
<div class="caption"
><span class="id">Figure 5.8: </span><span  
class="content">Graphs showing the CPU and memory used by the DRS. The memory
usage is shown in MBs abd CPU usage in %</span></div><!--tex4ht:label?: x10-58001r5 -->


</div><hr class="endfigure" />
<p>   Graphs in Figure <a
href="#x10-58001r8">5.8<!--tex4ht:ref: fig:self --></a> show the resource usage statistics of the DRS algorithm. As we
can see, the CPU usage is always less then 0.35% which is almost negligible. The memory
used by the DRS algorithm is 50MB with the entire virtual memory size of the software
being just 350MB.
<a
id="Q1-10-65"></a>
</p>

<h3 class="likesectionHead"><a
href="#x10-590004" id="x10-590004">Summary</a></h3>
<p>In this chapter, we described our experimental setup and compared the resource
utilization without autoballooning with the case when autoballooning was enabled. We
looked at the eﬀectiveness of steal time in identifying CPU hotspots. We also saw the
performance of the CUSUM algorithm which is used to ﬁlter sudden changes
in the resource usage from aﬀecting decision making of the DRS algorithm.
We also looked at the resources consumed by our implementation of the DRS
algorithm.


</p>
<!--l. 1--><div class="crosslinks"><p class="noindent">[<a
href="mainch6.html" >next</a>] [<a
href="mainch4.html" >prev</a>] [<a
href="mainch4.html#tailmainch4.html" >prev-tail</a>] [<a
href="mainch5.html" >front</a>] [<a
href="main.html#mainch5.html" >up</a>] </p></div>
<p>   <a
id="tailmainch5.html"></a>        </p>

{{< /raw >}}
