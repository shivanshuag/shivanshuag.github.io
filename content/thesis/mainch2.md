+++
title = "Related Work"
menu = "main"
url = "thesis/mainch2.html"
+++

{{< raw >}}
<div class="crosslinks"><p class="noindent">[<a
href="mainch3.html" >next</a>] [<a
href="mainch1.html" >prev</a>] [<a
href="mainch1.html#tailmainch1.html" >prev-tail</a>] [<a
href="#tailmainch2.html">tail</a>] [<a
href="main.html#mainch2.html" >up</a>] </p></div>
<h2 class="chapterHead"><span class="titlemark">Chapter 2</span><br /><a
href="main.html#QQ2-7-16" id="x7-130002">Related Work</a></h2>
<a
id="x7-13001r13"></a>
<h3 class="sectionHead"><span class="titlemark">2.1   </span> <a
href="main.html#QQ2-7-17" id="x7-140001">Auto-Ballooning in Xen</a></h3>
<p>Xen is a popular open-source, bare-metal hypervisor which was developed by University
of Cambridge Computer Laboratory in 2003 and was the ﬁrst hypervisor to support
paravirtualization. Support for full vritualization was later added to it. Xen has
autoballooning feature which works via the autoballoon driver that exists in the Linux
kernel. Autoballooning implementation in our work has some key diﬀerences
to autoballooning in Xen, which have been discussed later. It is important to
understand both techniques for comparison. To understand Xen hypervisor&#x2019;s method
of autoballooning, it is ﬁrst essential to understand transcendent memory in
Linux.
<a
id="x7-14001r10"></a>
</p>

<h4 class="subsectionHead"><span class="titlemark">2.1.1   </span> <a
href="mainli2.html#QQ2-7-18" id="x7-150001">Transcendent Memory (tmem)</a></h4>
<p>Transcendent memory (tmem) [<a id="page.15"></a><a
href="mainli5.html#X0-magenheimer2009transcendent" >13</a>] is a type of memory which the linux kernel cannot
directly enumerate, track or directly address, but helps in more eﬃcient utilization of
memory by a single kernel or load-balancing of memory between multiple kernels in a
virtualized environment.
</p>
<p>   The implementation of tmem is divided into two parts - frontend and backend.
Frontend provides the interfaces for diﬀerent types of data which can be stored
provided by tmem to the kernel, while backend is the underlying implementation of
storage/retrieval methods. Two basic operations provided by the frontend are &#x2019;put&#x2019; and
&#x2019;get&#x2019;. If the kernel wants to save some data into tmem, it uses the &#x2019;put&#x2019; operation while
&#x2019;get&#x2019; is used to retrieve the data.
</p>



<h5 class="subsubsectionHead"><a
href="#x7-160001" id="x7-160001">Tmem Frontends</a></h5>
<p>There are two tmem frontends in the Linux kernel which cover two major types of kernel
memory - <em><span
class="cmti-12">anonymous pages</span></em> and <em><span
class="cmti-12">ﬁle backed pages</span></em>.
   </p>
<dl class="enumerate"><dt class="enumerate">
1. </dt><dd
class="enumerate"><strong><span
class="cmbx-12">Cleancache:</span></strong> Cleancache is used for storing pages which are backed by ﬁles
   on a disk. A kernel can choose to reclaim such pages at the times of memory
   pressure. These pages are evicted from memory. If the same page is to be used
   again, a page fault happens and it is fetched from the disk. Before evicting
   such a page, the kernel can choose to store it in cleancache. If the operation
   succeeds, the page can possibly be reclaimed from tmem at any later time,
   otherwise it has to be reclaimed from disk, if needed
   <p>Cleancache data in tmem is ephemeral i.e. tmem can choose to discard any
   cleancache data at any point of time. Later, when a kernel wants its data back
   from tmem, if tmem has already discarded that data, the kernel can fetch it
   from the disk.
   </p>
</dd><dt class="enumerate">
2. </dt><dd
class="enumerate"><strong><span
class="cmbx-12">Frontswap:</span></strong> Frontswap is used for storing swap pages. Linux swap subsystem
   stores  anonymous  pages  in  a  swap  device  when  it  needs  to  evict  them.
   Whenever the swap subsystem of the Linux kernel wants to swap a page to
   swap device, it can send the page to tmem instead. If the operation succeeds,
   the page is written to tmem, otherwise it goes to the swap device. Frontswap
   provides a guarantee that a page stored in it can be reclaimed at any later
   point of time, and it will not be discarded.</dd></dl>
<h5 class="subsubsectionHead"><a
href="#x7-170001" id="x7-170001">Tmem Backends</a></h5>
<p>There are multiple backends for tmem - <em><span
class="cmti-12">zcache</span></em>, <em><span
class="cmti-12">transcendent memory for Xen</span></em>, and
<em><span
class="cmti-12">RAMster</span></em>. Tmem was originally developed for Xen with Xen backend. The other backends
were created later. For autoballooning, we are only concerned with the Xen
backend.


</p>
<p>   Tmem backend for Xen works in a virtualized environemnt under the Xen hypervisor
to share the spare hypervisor memory amongst the running VMs. In a virtualized
environment, the memory is allocated to the running VMs on demand. Some memory is
also reserved for the hypervisor to run. Thus, it may have some free memory which is not
used by anyone nd can be used for Tmem. This will result in faster swap out/in and page
reclamation for the guests under memory pressure. This plays an important part in
autoballooning of VMs and diﬀerentiates autoballooning in KVM from Xen, as we will see
in the later sections.
</p>

<h5 class="subsubsectionHead"><a
href="#x7-180001" id="x7-180001">Frontswap Self Shrinking</a></h5>
<p>When kernel swaps out a page, it assumes that the page will go to disk and may remain
there for long time even if it is not used again as kernel assumes disk space is less costly
and abundant. But if the page has gone to frontswap, it is taking up valuable memory
which might be better used elsewhere. To resolve this problem, <em><span
class="cmti-12">frontswap-self-shrinking</span></em> is
used. When a guest kernel is under normal memory pressure, it uses partial
swapoﬀ interface to bring the pages which were swapped to tmem back to guest&#x2019;s
memory. This frees tmem for use by other guests which might be under memory
pressure.
<a
id="x7-18001r18"></a>
</p>

<h4 class="subsectionHead"><span class="titlemark">2.1.2   </span> <a
href="mainli2.html#QQ2-7-22" id="x7-190002">Auto-Balloon Mechanism</a></h4>
<p>Autoballooning in Xen requires transcendent memory. In each guest, a autoballoon driver
is present. A thread of the driver runs periodically in some ﬁxed (conﬁgurable) time
interval which sets the target size of the guest.
</p>
<p>


<!--tex4ht:inline--></p>
<!--l. 35--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="block" >
   <mi
>T</mi><mi
>a</mi><mi
>r</mi><mi
>g</mi><mi
>e</mi><mi
>t</mi> <mo
class="MathClass-rel">=</mo> <mi
>C</mi><mi
>o</mi><mi
>m</mi><mi
>m</mi><mi
>i</mi><mi
>t</mi><mi
>t</mi><mi
>e</mi><mi
>d</mi><mspace width="1em" class="nbsp" /><mi
>p</mi><mi
>a</mi><mi
>g</mi><mi
>e</mi><mi
>s</mi> <mo
class="MathClass-bin">+</mo> <mi
>R</mi><mi
>e</mi><mi
>s</mi><mi
>e</mi><mi
>r</mi><mi
>v</mi><mi
>e</mi><mi
>d</mi><mspace width="1em" class="nbsp" /><mi
>p</mi><mi
>a</mi><mi
>g</mi><mi
>e</mi><mi
>s</mi> <mo
class="MathClass-bin">+</mo> <mi
>B</mi><mi
>a</mi><mi
>l</mi><mi
>l</mi><mi
>o</mi><mi
>o</mi><mi
>n</mi><mspace width="1em" class="nbsp" /><mi
>r</mi><mi
>e</mi><mi
>s</mi><mi
>e</mi><mi
>r</mi><mi
>v</mi><mi
>e</mi><mi
>d</mi><mspace width="1em" class="nbsp" /><mi
>p</mi><mi
>a</mi><mi
>g</mi><mi
>e</mi><mi
>s</mi>
</math>
<p>
</p>
<p>   <em><span
class="cmti-12">Committed pages</span></em> are the pages which are used by some process and may reside
either in memory or swap. <em><span
class="cmti-12">Reserved pages</span></em> are the pages reserved from normal
usage by the kernel. <em><span
class="cmti-12">Balloon reserved pages</span></em> form the memory reserved by the
balloon driver to provide some extra memory for the kernel caches and some
room to grow for any process that may demand more memory in the future. It
defaults to 10% of the total memory size of the guest. If the target is less than the
current RAM, the guest is ballooned down, else it is ballooned up. There is
a hysteresis counter which represents the number of iterations it will take for
the machine to balloon down to target. So, each time self-ballooning process
runs,
<!--tex4ht:inline--></p>
<!--l. 39--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="block" >
                 <mi
>M</mi><mi
>e</mi><mi
>m</mi><mi
>o</mi><mi
>r</mi><mi
>y</mi> <mo
class="MathClass-rel">=</mo> <mi
>C</mi><mi
>u</mi><mi
>r</mi><mi
>r</mi><mi
>e</mi><mi
>n</mi><mi
>t</mi> <mo
class="MathClass-bin">−</mo><mfrac><mrow
><mi
>C</mi><mi
>u</mi><mi
>r</mi><mi
>r</mi><mi
>e</mi><mi
>n</mi><mi
>t</mi> <mo
class="MathClass-bin">−</mo> <mi
>T</mi><mi
>a</mi><mi
>r</mi><mi
>g</mi><mi
>e</mi><mi
>t</mi></mrow>
<mrow
><mi
>h</mi><mi
>y</mi><mi
>s</mi><mi
>t</mi><mi
>e</mi><mi
>r</mi><mi
>e</mi><mi
>s</mi><mi
>i</mi><mi
>s</mi><mspace width="1em" class="nbsp" /><mi
>c</mi><mi
>o</mi><mi
>u</mi><mi
>n</mi><mi
>t</mi><mi
>e</mi><mi
>r</mi></mrow></mfrac>
</math>
<p>
</p>
<p>   Hysteresis counter is diﬀerent for ballooning up and down. There is also a<em><span
class="cmti-12">min usable</span>
<span
class="cmti-12">MB</span></em> parameter, below which machines cannot be ballooned.
</p>
<p>   It is important to note here that Xen autoballooning is proactive in nature. Even if
none of the guests is under memory pressure, it reclaims free/idle memory from the


guests. This may seem wasteful as the memory reclaimed when there is no memory
pressure on the host may have been better used by the original guest for page caches. But
this is compensated by the fact that the reclaimed memory, if not given to any other host,
will go to tmem, which will lead to faster swap in/out for all the guests. So, the
reclaimed memory is shared by all the guests and can be used by them depending
on the need. This kind of ballooning is not possible for KVM because there
is no tmem backend for QEMU-KVM. Thus, QEMU-KVM requires reactive
ballooning.
<a
id="x7-19001r17"></a>
</p>

<h3 class="sectionHead"><span class="titlemark">2.2   </span> <a
href="main.html#QQ2-7-23" id="x7-200002">Memory Management in VMware ESX</a></h3>
<p>VMware was founded in 1999 when it launched its proprietary hypervisor which used
binary translation techniques for x86 virtualization. They later released an enterprise class
bare metal hypervisor called VMware ESXi. ESX has very robust memory management
and it introduced several novel techniques as early as 2003 which are still being used and
have been implemented in other platforms. The techniques used by ESX to manage
memory [<a id="page.19"></a><a
href="mainli5.html#X0-waldspurger2002memory" >14</a>] of its guests and facilitate memory overcommitment have been very brieﬂy
described here.
<a
id="x7-20001r22"></a>
</p>

<h4 class="subsectionHead"><span class="titlemark">2.2.1   </span> <a
href="mainli2.html#QQ2-7-24" id="x7-210001">Memory Reclamation Techniques</a></h4>
<p>ESX uses several memory reclamation techniques.
   </p>
<dl class="enumerate"><dt class="enumerate">
1. </dt><dd
class="enumerate"><strong><span
class="cmbx-12">Content-Based Page Sharing:</span></strong> In a virtualized environment, several guests
   might be running common OS and some common applications which means
   that  there  would  be  lots  of  pages  having  the  same  content.  Instead  of
   keeping  separate  copies  of  these  pages  for  separate  guests,  the  duplicate
   pages are deleted and only one copy of the page is kept which is marked


   Copy-On-Write(COW). When any of the guests attempts to write to that
   page, a new copy of the page is created by the kernel which is then modiﬁed.
   <p>Linux kernel has KSM(Kernel Same-Page Merging) [<a id="page.20"></a><a
href="mainli5.html#X0-ksm" >15</a>] which performs the
   same task and is used in virtualization by QEMU-KVM.
   </p>
</dd><dt class="enumerate">
2. </dt><dd
class="enumerate"><strong><span
class="cmbx-12">Memory Ballooning:</span></strong> Memory Ballooning has been explained in Section
   <a
href="mainch1.html#x6-70001">1.1.1<!--tex4ht:ref: ballooning --></a>
   </dd><dt class="enumerate">
3. </dt><dd
class="enumerate"><strong><span
class="cmbx-12">Page compression:</span></strong> In this case, the content of the page is compressed and
   stored. When the page is accessed, its content is decompressed.
   </dd><dt class="enumerate">
4. </dt><dd
class="enumerate"><strong><span
class="cmbx-12">Demand Paging:</span></strong> ESX server has a swap daemon which handles hypervisor
   level swapping. The swap daemon gets the target swap levels of each VM from
   the swapping policy and selects the pages that need to be swapped.
   </dd></dl>
<a
id="x7-21005r24"></a>
<h4 class="subsectionHead"><span class="titlemark">2.2.2   </span> <a
href="mainli2.html#QQ2-7-25" id="x7-220002">Memory Reclamation Policies</a></h4>
<p>The ESX server deﬁnes four memory states depending upon which it employs the memory
reclamation techniques.
   </p>
<dl class="enumerate"><dt class="enumerate">
1. </dt><dd
class="enumerate"><strong><span
class="cmbx-12">High:</span></strong> More than 6% of the total memory of the hypervisor is free at this
   point. Only page sharing is employed in this state.
   </dd><dt class="enumerate">
2. </dt><dd
class="enumerate"><strong><span
class="cmbx-12">Soft:</span></strong> Free memory is between 6% and 4%. Page sharing is active. Balloon
   driver also starts reclaiming idle memory.
   </dd><dt class="enumerate">


3. </dt><dd
class="enumerate"><strong><span
class="cmbx-12">Hard:</span></strong> Free memory is between 4% and 2%. The hypervisor now aggressively
   starts reclaiming memory through swapping. Page compression also becomes
   active.  So,  on  page  reclamation,  if  it  is  compressible  or  shareable,  it  is
   compressed/shared otherwise it is swapped out.
   </dd><dt class="enumerate">
4. </dt><dd
class="enumerate"><strong><span
class="cmbx-12">Low:</span></strong> Below  1%  free  memory.  Along  with  memory  reclamation,  ESX  also
   blocks any new memory allocation by any guest.</dd></dl>
<p>   One important point of contention here is which pages and how many pages to reclaim
from each guest. For determining how much memory to reclaim from each guest, ESX
uses a share based allocation technique. After getting the amount of memory to reclaim,
it chooses the pages to reclaim randomly.
</p>

<h5 class="subsubsectionHead"><a
href="#x7-230002" id="x7-230002">Share-Based Allocation</a></h5>
<p>Under share-based allocation, each guest is allocated some number of shares for
memory. A machine is guaranteed a minimum resource fraction equal to its
fraction of the total shares in the system. So, in case of memory pressure, memory
should be revoked from the guest which has the fewest shares per allocated page.
This may lead to idle clients having large number of shares hoarding memory
unproductively while active clients having fewer shares suﬀer under memory
pressure.
</p>
<p>   To resolve this situation, ESX imposes a tax on idle memory of guests. Thus, for a client with
<!--l. 75--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mi
>S</mi></math> shares and
<!--l. 75--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mi
>P</mi></math> allocated pages out of
which a fraction <!--l. 75--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mi
>f</mi></math> are active,
the shares-per-page ratio <!--l. 75--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mi
>R</mi></math>
is


<!--tex4ht:inline--></p>
<!--l. 76--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="block" >
                           <mi
>R</mi> <mo
class="MathClass-rel">=</mo>           <mfrac><mrow
><mi
>S</mi></mrow>
<mrow
><mi
>P</mi><mrow ><mo
class="MathClass-open">(</mo><mrow><mi
>f</mi> <mo
class="MathClass-bin">+</mo> <mi
>k</mi><mrow ><mo
class="MathClass-open">(</mo><mrow><mn>1</mn> <mo
class="MathClass-bin">−</mo> <mi
>f</mi></mrow><mo
class="MathClass-close">)</mo></mrow></mrow><mo
class="MathClass-close">)</mo></mrow></mrow></mfrac>
</math>
<p> where the idle page cost is <!--l. 77--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mi
>k</mi> <mo
class="MathClass-rel">=</mo> <mn>1</mn><mo
class="MathClass-bin">∕</mo><mrow ><mo
class="MathClass-open">(</mo><mrow><mn>1</mn> <mo
class="MathClass-bin">−</mo> <mi
>τ</mi></mrow><mo
class="MathClass-close">)</mo></mrow></math>
and tax rate <!--l. 77--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mi
>τ</mi></math>
is between 0 and 1. Tax rate controls the maximum fraction of
idle pages that can be reclaimed from a VM. The default value of
<!--l. 77--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mi
>τ</mi></math> is
<!--l. 77--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mn>0</mn><mo
class="MathClass-punc">.</mo><mn>7</mn><mn>5</mn></math>.
</p>

<h5 class="subsubsectionHead"><a
href="#x7-240002" id="x7-240002">Idle Memory Calculation</a></h5>
<p>For calculating share per page ratio eﬀectively, idle memory calculation for each
guest is required. ESX uses statistical sampling to estimate the working
set size of each VM. At the start of the sampling period, a small number
<!--l. 79--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mi
>n</mi></math>
of a VM&#x2019;s pages are selected randomly using uniform distribution. Accesses
to these pages are tracked by the hypervisor by incrementing a counter
<!--l. 79--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mi
>t</mi></math> on every
page access. At the end of the sampling period, the fraction of working set is estimated as
<!--l. 79--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mi
>f</mi> <mo
class="MathClass-rel">=</mo> <mi
>t</mi><mo
class="MathClass-bin">∕</mo><mi
>n</mi></math>.
<a
id="x7-24001r23"></a>
</p>

<h3 class="sectionHead"><span class="titlemark">2.3   </span> <a
href="main.html#QQ2-7-28" id="x7-250003">VMware Distributed Resource Management</a></h3>
<p>VMware created DRS [<a id="page.22"></a><a
href="mainli5.html#X0-gulati2012vmware" >16</a>] for their hypervisor which is responsible for allocation of the
physical resources to a set of virtual machines deployed in a cluster of hosts. The key
capabilities of their DRS are a hierarchical resource pool abstraction, automatic initial


placement of VM, and dynamic load balancing of VMs based on their ﬂuctuating
demands. Some of our work is inspired by VMware&#x2019;s DRS and aimed at ﬁxing some of its
limitations. Hence, this section provides an appropriate background for the next
chapter.
<a
id="x7-25001r25"></a>
</p>

<h4 class="subsectionHead"><span class="titlemark">2.3.1   </span> <a
href="mainli2.html#QQ2-7-29" id="x7-260001">Resource Model</a></h4>
<p>Each VM in VMware DRS has three resource control parameters. <em><span
class="cmti-12">Reservation</span></em> speciﬁes
a minimum guaranteed amount of a particular resource which will always be
available to the VM. <em><span
class="cmti-12">Limit</span></em> speciﬁes the maximum amount of a resource that
can be allocated to a VM. <em><span
class="cmti-12">Shares</span></em> specify the relative importance of the VMs
similar to the shares in memory allocation of ESX described in the previous
section.
</p>
<p>   The resources in a cluster are divided into resource pools which are used for dividing
the aggregate capacity of the cluster into a group of VMs. The resource pool forms a
hierarchical tree structure with leaves as the actual VMS. The resource pool can represent
the hierarchical structure of the oraganization.
<a
id="x7-26001r29"></a>
</p>

<h4 class="subsectionHead"><span class="titlemark">2.3.2   </span> <a
href="mainli2.html#QQ2-7-30" id="x7-270002">DRS Algorithm</a></h4>
<p>The DRS algorithm runs every ﬁve minutes. It ﬁrst divides the resources of the cluster
amongst the resource pool in a process called <em><span
class="cmti-12">divvying</span></em>. It computes the reservation,
limit, and shares of each pool. To distribute the resources, it is important to
calculate the demand of each VM, which can be used to calculate the total
load on the cluster. The working set of a VM is used as it&#x2019;s memory demand.
The working set is calculated in the way explained in Section <a
href="#x7-240002">2.2.2<!--tex4ht:ref: working-set --></a>. The CPU
demand of a VM is computed as its actual CPU consumption plus a scaled
portion of the time it was ready to execute, but it spent in the ready queue due to
contention.


</p>
<p>
<!--tex4ht:inline--></p>
<!--l. 92--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="block" >
         <mi
>C</mi><mi
>P</mi><msub><mrow
><mi
>U</mi></mrow><mrow
><mi
>d</mi><mi
>e</mi><mi
>m</mi><mi
>a</mi><mi
>n</mi><mi
>d</mi></mrow></msub
> <mo
class="MathClass-rel">=</mo> <mi
>C</mi><mi
>P</mi><msub><mrow
><mi
>U</mi></mrow><mrow
><mi
>u</mi><mi
>s</mi><mi
>e</mi><mi
>d</mi></mrow></msub
> <mo
class="MathClass-bin">+</mo>         <mfrac><mrow
><mi
>C</mi><mi
>P</mi><msub><mrow
><mi
>U</mi></mrow><mrow
><mi
>r</mi><mi
>u</mi><mi
>n</mi></mrow></msub
></mrow>
<mrow
><mi
>C</mi><mi
>P</mi><msub><mrow
><mi
>U</mi></mrow><mrow
><mi
>r</mi><mi
>u</mi><mi
>n</mi></mrow></msub
> <mo
class="MathClass-bin">+</mo> <mi
>C</mi><mi
>P</mi><msub><mrow
><mi
>U</mi></mrow><mrow
><mi
>s</mi><mi
>l</mi><mi
>e</mi><mi
>e</mi><mi
>p</mi></mrow></msub
></mrow></mfrac> <mo
class="MathClass-bin">∗</mo> <mi
>C</mi><mi
>P</mi><msub><mrow
><mi
>U</mi></mrow><mrow
><mi
>r</mi><mi
>e</mi><mi
>a</mi><mi
>d</mi><mi
>y</mi></mrow></msub
>
</math>
<p>
</p>
<p>   The divvying algorithm works by dividing the resource at the parent resource pool
amongst the children. It executes in two phases. In the initial phase, it aggregates demand
values of the VMs from leaves up to the root. At each step, the demand values are
updated to be not less than the reservation and not more than the limit. The second
phase proceeds in a top-down manner and resources (limit and reservation) of
parents at each level are divided such that they are in proportion to shares of the
siblings.
</p>
<p>   After divvying, the DRS load-balances the cluster based on a metric called
dynamic entitlement. Dynamic entitlement is equal to the demand of the VMs if
demands of each VM in the cluster can be met, otherwise it is a scaled down
value of a VMs demand based on its shares. It is computed by running the
divvying algorithm at the resource pool tree using the cluster capacity as the
resource. Then, normalized entitlement is calculated for each host. For a host
<!--l. 96--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mi
>h</mi></math>, normalized
entitlement <!--l. 96--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><msub><mrow
><mi
>N</mi></mrow><mrow
><mi
>h</mi></mrow></msub
></math>
is deﬁned as the sum of per VM entitlement of all VMs running on
<!--l. 97--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mi
>h</mi></math>
divided by the host capacity.


<!--tex4ht:inline--></p>
<!--l. 98--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="block" >
                                <msub><mrow
><mi
>N</mi></mrow><mrow
><mi
>h</mi></mrow></msub
> <mo
class="MathClass-rel">=</mo> <mi
>Σ</mi> <mfrac><mrow
><msub><mrow
><mi
>E</mi></mrow><mrow
><mi
>i</mi></mrow></msub
></mrow>
<mrow
><msub><mrow
><mi
>C</mi></mrow><mrow
><mi
>h</mi></mrow></msub
></mrow></mfrac>
</math>
<p> <!--l. 99--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><msub><mrow
><mi
>N</mi></mrow><mrow
><mi
>h</mi></mrow></msub
></math> values are
per-resource. <!--l. 99--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><msub><mrow
><mi
>N</mi></mrow><mrow
><mi
>h</mi></mrow></msub
> <mo
class="MathClass-rel">&#x003C;</mo> <mn>1</mn></math>
signiﬁes that the host has some unused capacity for that resource, while
<!--l. 99--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><msub><mrow
><mi
>N</mi></mrow><mrow
><mi
>h</mi></mrow></msub
> <mo
class="MathClass-rel">&#x003E;</mo> <mn>1</mn></math> means
that it is overloaded for that resource. The DRS then calculates the cluster wide imbalance
<!--l. 99--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><msub><mrow
><mi
>I</mi></mrow><mrow
><mi
>c</mi></mrow></msub
></math>
as the weighted sum of the standard deviation of all
<!--l. 99--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><msub><mrow
><mi
>N</mi></mrow><mrow
><mi
>h</mi></mrow></msub
></math> values
for CPU and memory resources. If memory is highly contested, it&#x2019;s weight is
more. If CPU is highly contested, CPU&#x2019;s weight is more. If none are highly
contested, both have equal weights. The higher weighted resource gets a weight of
<!--l. 99--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mi
>t</mi><mi
>h</mi><mi
>r</mi><mi
>e</mi><mi
>e</mi></math> while the lower weighted
resource gets a weight of <!--l. 99--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mi
>o</mi><mi
>n</mi><mi
>e</mi></math>.
These values were decided by experimentation.
</p>
<p>   The DRS then tries to minimize the cluster wide imbalance
<!--l. 101--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><msub><mrow
><mi
>I</mi></mrow><mrow
><mi
>c</mi></mrow></msub
></math>
by considering all possible VM migrations (trying out all possible guest VM ,
destination host pairs). The best VM migration is selected, applied to clusters state,
and another move is selected. This process continues till no beneﬁcial moves
remain, or enough moves have been selected for this pass, or the cluster imbalance
<!--l. 101--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><msub><mrow
><mi
>I</mi></mrow><mrow
><mi
>c</mi></mrow></msub
></math> is
below a threshold. Additionally, DRS also does a cost-beneﬁt analysis of each move where


it ﬁlters out the moves with higher migration cost than beneﬁt in terms of decreasing load
imbalance.
<a
id="x7-27001r30"></a>
</p>

<h4 class="subsectionHead"><span class="titlemark">2.3.3   </span> <a
href="mainli2.html#QQ2-7-31" id="x7-280003">Limitations of VMware DRS</a></h4>
<p>The major limitation of VMware DRS is scaling, which is due to several reasons. There is
a central controller for the cluster which monitors all the hosts and virtual machines.
In a cloud scale, where the hosts can grow to thousands in number and VMs
to tens of thousand, a central machine monitoring all of the components and
taking all the decisions is not possible. Apart from this, in making a decision, the
DRS considers all possible VM migrations. The time for running this algorithm
grows exponentially with the number of VMs and hence cannot scale. On top of
all this, the controller is a single point of failure for the whole cluster in this
design.
</p>
<p>   Another limitation of VMware DRS is that it manages only CPU and memory
resources. Other resources like network bandwidth, and disk I/O, which can aﬀect the
performance of a VM,can also be considered.
<a
id="x7-28001r28"></a>
</p>

<h3 class="sectionHead"><span class="titlemark">2.4   </span> <a
href="main.html#QQ2-7-32" id="x7-290004">Other Works</a></h3>
<p>Sandpiper [<a id="page.26"></a><a
href="mainli5.html#X0-wood2009sandpiper" >17</a>] is a system for automating the task of detecting hotspots and migrating VMs in
response to hotspots. For VM migration, Sandpiper takes three resources into consideration -
CPU, memory and network bandwidth. The empty volume of each host is calculated as
<!--l. 110--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mi
>e</mi><mi
>m</mi><mi
>p</mi><mi
>t</mi><mi
>y</mi><mstyle
class="text"><mtext  >_</mtext></mstyle><mi
>v</mi><mi
>o</mi><mi
>l</mi> <mo
class="MathClass-rel">=</mo> <mrow ><mo
class="MathClass-open">(</mo><mrow><mn>1</mn> <mo
class="MathClass-bin">−</mo> <mi
>c</mi><mi
>p</mi><mi
>u</mi></mrow><mo
class="MathClass-close">)</mo></mrow><mrow ><mo
class="MathClass-open">(</mo><mrow><mn>1</mn> <mo
class="MathClass-bin">−</mo> <mi
>n</mi><mi
>e</mi><mi
>t</mi></mrow><mo
class="MathClass-close">)</mo></mrow><mrow ><mo
class="MathClass-open">(</mo><mrow><mn>1</mn> <mo
class="MathClass-bin">−</mo> <mi
>m</mi><mi
>e</mi><mi
>m</mi></mrow><mo
class="MathClass-close">)</mo></mrow></math>. It
calculates <!--l. 110--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mi
>v</mi><mi
>o</mi><mi
>l</mi><mi
>u</mi><mi
>m</mi><mi
>e</mi></math>
occupied by a VM as


<!--tex4ht:inline--></p>
<!--l. 110--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="block" >
                    <mi
>v</mi><mi
>o</mi><mi
>l</mi> <mo
class="MathClass-rel">=</mo>     <mfrac><mrow
><mn>1</mn></mrow>
<mrow
><mn>1</mn> <mo
class="MathClass-bin">−</mo> <mi
>c</mi><mi
>p</mi><mi
>u</mi></mrow></mfrac> <mo
class="MathClass-bin">∗</mo>  <mfrac><mrow
><mn>1</mn></mrow>
<mrow
><mn>1</mn> <mo
class="MathClass-bin">−</mo> <mi
>n</mi><mi
>e</mi><mi
>t</mi></mrow></mfrac> <mo
class="MathClass-bin">∗</mo>   <mfrac><mrow
><mn>1</mn></mrow>
<mrow
><mn>1</mn> <mo
class="MathClass-bin">−</mo> <mi
>m</mi><mi
>e</mi><mi
>m</mi></mrow></mfrac>
</math>
<p>. It then calculates volume to size ratio (VSR) of each VM, where size is a VMs
memory footprint. The VM with the maximum VSR from the host having the least
<!--l. 110--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mi
>e</mi><mi
>m</mi><mi
>p</mi><mi
>t</mi><mi
>y</mi><mstyle
class="text"><mtext  >_</mtext></mstyle><mi
>v</mi><mi
>o</mi><mi
>l</mi></math> is migrated to the host
having the maximum <!--l. 110--><math
xmlns="http://www.w3.org/1998/Math/MathML"  
display="inline" ><mi
>e</mi><mi
>m</mi><mi
>p</mi><mi
>t</mi><mi
>y</mi><mstyle
class="text"><mtext  >_</mtext></mstyle><mi
>v</mi><mi
>o</mi><mi
>l</mi></math>.
Some of the problems and anomalies in this approach have been explored [<a id="page.27"></a><a
href="mainli5.html#X0-mishra2011theory" >18</a>].
</p>
<p>   CloudScale [<a
href="mainli5.html#X0-shen2011cloudscale" >19</a>] uses long-term resource demand prediction models to predict the
resource demands of virtual machines in future based on the previous resource usage data.
It handle conﬂicts by proactive migration of virtual machines based on the predicted
demands. Tan et al. [<a
href="mainli5.html#X0-tan2011exploiting" >20</a>] propose a similar approach where they use a Principal
Component Analysis (PCA) based approach to predict the long-term resource usage
proﬁles of the VMs using the measurements obtained from a commercial data center.
They formulate the VM placement problem as a bin-packing problem and propose
to place VMs using variance reduction. But in doing so, they assume that the
resource usages of the VMs remains constant, which may not be the case in
reality.
</p>
<p>   Vector Dot [<a
href="mainli5.html#X0-singh2008server" >21</a>] is a load balancing algorithm for handling multi-dimensional resource
constraints in a virtualized infrastructure. It expresses the normalized resource
requirements of machines as multi-dimensional vectors. It then uses the dot product
of the resource usage vector (RUV) of the host and the resource requirement
vector (RRV) of the guest to choose an appropriate host-VM mapping. To ensure
balanced usage of resources, the RRV of the guest should be complementary to the
RUV of the host. The anomalies with this approach have been discussed and
improvements have been proposed [<a
href="mainli5.html#X0-mishra2011theory" >18</a>]. But these studies do not take scaling
into consideration and are not distributed enough to handle the cloud scale.


They also do not consider the performance degradation that happens during live
migration.
<a
id="Q1-7-33"></a>
</p>

<h4 class="likesubsectionHead"><a
href="#x7-300004" id="x7-300004">Summary</a></h4>
<p>In this chapter, we have discussed the autoballooning mechanism implemented in the Xen
hypervisor and how it is diﬀerent from the autoballooning in KVM. We have discussed
the techniques which VMware ESX implemens for memory management and distributed
resource scheduling. We have also explored some other important works in this area and
their drawbacks.


</p>
<!--l. 1--><div class="crosslinks"><p class="noindent">[<a
href="mainch3.html" >next</a>] [<a
href="mainch1.html" >prev</a>] [<a
href="mainch1.html#tailmainch1.html" >prev-tail</a>] [<a
href="mainch2.html" >front</a>] [<a
href="main.html#mainch2.html" >up</a>] </p></div>
<p>   <a
id="tailmainch2.html"></a>                  </p>
{{< /raw >}}
