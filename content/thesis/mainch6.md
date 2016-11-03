+++
title = "Conclusions"
menu = "main"
url = "thesis/mainch6.html"
+++
{{< raw >}}
<div class="crosslinks"><p class="noindent">[<a
href="mainli5.html" >next</a>] [<a
href="mainch5.html" >prev</a>] [<a
href="mainch5.html#tailmainch5.html" >prev-tail</a>] [<a
href="#tailmainch6.html">tail</a>] [<a
href="main.html#mainch6.html" >up</a>] </p></div>
<h2 class="chapterHead"><span class="titlemark">Chapter 6</span><br /><a
href="main.html#QQ2-10-62" id="x11-600006">Conclusions</a></h2>
<a
id="x11-60001r60"></a>
<h3 class="sectionHead"><span class="titlemark">6.1   </span> <a
href="main.html#QQ2-10-63" id="x11-610001">Summary</a></h3>
<p>Resource overcommitment is an essential technique for eﬃcient use of resources in a cloud
infrastructure. There are several hurdles in overcommitment which have not been
addressed completely yet. Most of the studies in this ﬁeld focus on just a part of the
problem, but do not solve it as a whole. The subproblems that have attracted most
attention is the optimal placement of virtual machines depending on their demands. But
many of these studies do not take the dynamic nature of resource demand of the VMs or
the performance degradation during live migration into account. Through this thesis, we
have tried to solve this problem by taking the whole picture into account, and not just a
part of it.
</p>
<p>   In this thesis, we have identiﬁed some of the major problems faced in overcommitment
of CPU and memory resources in a virtualized environment. We have described some of
the relevant work done in this area and their limitations. We have proposed an approach
and architecture to build a decentralized distributed resource scheduler to manage the
cloud resources under overcommitment. The DRS is designed to be horizontally scalable
and has no single point of failure. We have also implemented a complete DRS system for
QEMU-KVM virtualization platform. The implementation of the autoballooning and
monitoring component have been described in detail and their performance has been
analyzed by experimentation.
</p>
<p>   Our implementation is open-source and the code for it can be found here:
</p>

<div class="center"
>
<p>
</p>
<p><a
href="https://github.com/shivanshuag/thesis-code" class="url" ><span
class="cmtt-12">https://github.com/shivanshuag/thesis-code</span></a></p>
</div>


<a
id="x11-61001r68"></a>
<h3 class="sectionHead"><span class="titlemark">6.2   </span> <a
href="main.html#QQ2-10-64" id="x11-620002">Future Work</a></h3>
<p>Although we have demonstrated that our technique for autoballooning is eﬃcient, there
are still some limitations to this approach which need to be addressed.
<a
id="x11-62001r64"></a>
</p>

<h4 class="subsectionHead"><span class="titlemark">6.2.1   </span> <a
href="#x11-630001" id="x11-630001">Limitation of Memory Ballooning</a></h4>
<p>The balloon driver is used to modify the amount of memory a VM has. This might have
some unintended aﬀects on the processes running inside the VM. A new process starting
on the guest may require more memory than is present(because some amount of memory
has been ballooned out) and crash if it does not get that memory. In some cases, if no
available memory is left inside the guest, and the guest does not have swap space, the
Linux OOM killer may be activated to kill some of the processes and reclaim memory.
This is a rare situation and requires that either there is no swap device, or swap memory
is exhausted.
</p>
<p>   Ideally, the memory ballooning process should be totally transparent to the guest VM.
Doing this is a non-trivial task which might require exposing the memory allocation
requests inside the guest VM to the host.
<a
id="x11-63001r70"></a>
</p>

<h4 class="subsectionHead"><span class="titlemark">6.2.2   </span> <a
href="#x11-640002" id="x11-640002">Supporting More Resource Types</a></h4>
<p>The DRS we have built right now supports balancing of only CPU and memory resources.
Other resources like the network bandwidth and disk i/o can also be overcommited and
can be taken into account for balancing and migration.
<a
id="x11-64001r71"></a>
</p>

<h4 class="subsectionHead"><span class="titlemark">6.2.3   </span> <a
href="#x11-650003" id="x11-650003">Multiple Migrations</a></h4>
<p>In our DRS, the migration service considers only single migrations to resolve


hotspots. Sometimes, multiple migrations may be required to resolve a single
hotspot. This will involve coordination between more than two machines for
migration. Strategies to make this process eﬀective and beneﬁcial need to be
explored.


</p>
<p>




</p>
<!--l. 123--><div class="crosslinks"><p class="noindent">[<a
href="mainli5.html" >next</a>] [<a
href="mainch5.html" >prev</a>] [<a
href="mainch5.html#tailmainch5.html" >prev-tail</a>] [<a
href="mainch6.html" >front</a>] [<a
href="main.html#mainch6.html" >up</a>] </p></div>
<p>   <a
id="tailmainch6.html"></a>      </p>
{{< /raw >}}
