+++
title = "Distributed Memory and CPU Management in Cloud Computing Environments"
menu = "main"
url = "thesis/main.html"
+++

{{< raw >}}

<div class="maketitle">


<br />
<h2 class="titleHead">Distributed Memory and CPU Management in
Cloud Computing Environments</h2><br /><br />
<em><span
class="cmti-12x-x-120">A thesis</span><span
class="cmti-12x-x-120"> submitted</span></em><br />
<span
class="cmr-12x-x-120">in Partial Fulﬁllment of the Requirements</span><br />
<span
class="cmr-12x-x-120">for the Degree of</span><br /><br />
<span
class="cmr-12x-x-120">Master of Technology</span><br /><br />
<span
class="cmr-12x-x-120">by</span><br /><br />
<div class="author" ><span
class="cmbx-12x-x-120">Shivanshu Agrawal</span></div><br /><br />
<em><span
class="cmti-12x-x-120">to the</span></em><br />
<span
class="cmr-12x-x-120">DEPARTMENT OF COMPUTER SCIENCE &#x0026; ENGINEERING</span><br />
<span
class="cmr-12x-x-120">INDIAN INSTITUTE OF TECHNOLOGY KANPUR</span><br />
<div class="date" ><span
class="cmr-12x-x-120">May, 2016</span></div>


</div>
<div
class="abstract"
>


<div class="center"
>
<p>
</p>
<p><span
class="cmbx-12x-x-120">ABSTRACT</span></p>
</div>
<p>To eﬃciently utilize the resources in a virtualized environment, they need to be
overcommited. Overcommitment is the process of allocating more resources to a virtual
machine, or a group of virtual machines than are physically present on the host. It is
based on the premise that most of the Virtual Machines will use only a small percentage
of the resources allocated to them at a given time. However, situations may arise when
the resources required by the virtual machines are more than the physical resources
present on the machine. The performance of the Virtual Machines will be severely
impacted in these situations. Most cloud providers generally have Service Level
Agreements (SLAs) with their clients and hence cannot allow virtual machines to deliver
a poor quality of service.
</p>
<p>   In this thesis, we explore the problem of overcommitment of the CPU and memory
resources. We propose a distributed resource scheduler (DRS) which uses techniques such
as memory ballooning and virtual machine live nigration to solve the problems in CPU
and memory overcommitment. We design an architecture for DRS which is horizontally
scalable and describe the techniques involved in monitoring and memory ballooning
aspects of the DRS.


</p>
</div>


<div class="tableofcontents">
<span class="likechapterToc" ><a
href="mainli1.html#x2-1000" id="QQ2-2-1">Acknowledgements</a></span>
<br />   <span class="likechapterToc" ><a
href="mainli2.html#x3-2000" id="QQ2-3-2">Contents</a></span>
<br />  <span class="chapterToc" >1 <a
href="mainch1.html#x6-50001" id="QQ2-6-7">Introduction: Virtualization and Resource Management in Cloud</a></span>
<br />    <span class="sectionToc" >1.1 <a
href="mainch1.html#x6-60001" id="QQ2-6-8">Virtualization</a></span>
<br />    <span class="sectionToc" >1.2 <a
href="mainch1.html#x6-90002" id="QQ2-6-11">Virtual Machine Live Migration</a></span>
<br />    <span class="sectionToc" >1.3 <a
href="mainch1.html#x6-100003" id="QQ2-6-12">Resource Management in Cloud</a></span>
<br />    <span class="sectionToc" >1.4 <a
href="mainch1.html#x6-110004" id="QQ2-6-13">Organization of this Thesis </a></span>
<br />   <span class="chapterToc" >2 <a
href="mainch2.html#x7-130002" id="QQ2-7-16">Related Work</a></span>
<br />    <span class="sectionToc" >2.1 <a
href="mainch2.html#x7-140001" id="QQ2-7-17">Auto-Ballooning in Xen</a></span>
<br />    <span class="sectionToc" >2.2 <a
href="mainch2.html#x7-200002" id="QQ2-7-23">Memory Management in VMware ESX</a></span>
<br />    <span class="sectionToc" >2.3 <a
href="mainch2.html#x7-250003" id="QQ2-7-28">VMware Distributed Resource Management</a></span>
<br />    <span class="sectionToc" >2.4 <a
href="mainch2.html#x7-290004" id="QQ2-7-32">Other Works</a></span>
<br />   <span class="chapterToc" >3 <a
href="mainch3.html#x8-310003" id="QQ2-8-35">Architecture and Design of DRS</a></span>
<br />    <span class="sectionToc" >3.1 <a
href="mainch3.html#x8-320001" id="QQ2-8-36">Functions of DRS</a></span>
<br />    <span class="sectionToc" >3.2 <a
href="mainch3.html#x8-330002" id="QQ2-8-37">Goals and Non-Goals</a></span>
<br />    <span class="sectionToc" >3.3 <a
href="mainch3.html#x8-340003" id="QQ2-8-38">Architecture Overview</a></span>
<br />   <span class="chapterToc" >4 <a
href="mainch4.html#x9-390004" id="QQ2-9-44">Implementation of Monitoring and Auto-Ballooning</a></span>
<br />    <span class="sectionToc" >4.1 <a
href="mainch4.html#x9-400001" id="QQ2-9-45">Monitoring</a></span>
<br />    <span class="sectionToc" >4.2 <a
href="mainch4.html#x9-480002" id="QQ2-9-53">Auto-Ballooning</a></span>
<br />   <span class="chapterToc" >5 <a
href="mainch5.html#x10-520005" id="QQ2-10-58">Experimental Evaluation</a></span>


<br />    <span class="sectionToc" >5.1 <a
href="mainch5.html#x10-530001" id="QQ2-10-59">Experimental Setup</a></span>
<br />    <span class="sectionToc" >5.2 <a
href="mainch5.html#x10-540002" id="QQ2-10-60">Results</a></span>
<br />
<span class="chapterToc" >6 <a
href="mainch6.html#x11-600006" id="QQ2-10-62">Conclusions</a></span>


<br />    <span class="sectionToc" >6.1 <a
href="mainch6.html#x11-610001" id="QQ2-10-63">Summary</a></span>
<br />    <span class="sectionToc" >6.2 <a
href="mainch6.html#x11-620002" id="QQ2-10-64">Future Work</a></span>
<br />
<span class="likechapterToc" ><a
href="mainli5.html#x12-660003" id="QQ2-2-61">References</a></span>
<br />

</div>


{{< /raw >}}
