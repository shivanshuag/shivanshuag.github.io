+++
title = "Contents"
menu = "main"
url = "thesis/mainli2.html"
+++

{{< raw >}}
<div class="crosslinks"><p class="noindent">[<a
href="mainli3.html" >next</a>] [<a
href="mainli1.html" >prev</a>] [<a
href="mainli1.html#tailmainli1.html" >prev-tail</a>] [<a
href="#tailmainli2.html">tail</a>] [<a
href="main.html#mainli2.html" >up</a>] </p></div>
<h2 class="likechapterHead"><a
href="main.html#QQ2-3-2" id="x3-2000">Contents</a></h2> <div class="tableofcontents">
<span class="chapterToc" >1 <a
href="mainch1.html#x6-50001">Introduction: Virtualization and Resource Management in Cloud</a></span>
<br />    <span class="sectionToc" >1.1 <a
href="mainch1.html#x6-60001">Virtualization</a></span>
<br />     <span class="subsectionToc" >1.1.1 <a
href="mainch1.html#x6-70001" id="QQ2-6-9">Memory Overcommitment and Ballooning</a></span>
<br />     <span class="subsectionToc" >1.1.2 <a
href="mainch1.html#x6-80002" id="QQ2-6-10">CPU Overcommitment</a></span>
<br />    <span class="sectionToc" >1.2 <a
href="mainch1.html#x6-90002">Virtual Machine Live Migration</a></span>
<br />    <span class="sectionToc" >1.3 <a
href="mainch1.html#x6-100003">Resource Management in Cloud</a></span>
<br />    <span class="sectionToc" >1.4 <a
href="mainch1.html#x6-110004">Organization of this Thesis </a></span>
<br />   <span class="chapterToc" >2 <a
href="mainch2.html#x7-130002">Related Work</a></span>
<br />    <span class="sectionToc" >2.1 <a
href="mainch2.html#x7-140001">Auto-Ballooning in Xen</a></span>
<br />     <span class="subsectionToc" >2.1.1 <a
href="mainch2.html#x7-150001" id="QQ2-7-18">Transcendent Memory (tmem)</a></span>
<br />     <span class="subsectionToc" >2.1.2 <a
href="mainch2.html#x7-190002" id="QQ2-7-22">Auto-Balloon Mechanism</a></span>
<br />    <span class="sectionToc" >2.2 <a
href="mainch2.html#x7-200002">Memory Management in VMware ESX</a></span>
<br />     <span class="subsectionToc" >2.2.1 <a
href="mainch2.html#x7-210001" id="QQ2-7-24">Memory Reclamation Techniques</a></span>
<br />     <span class="subsectionToc" >2.2.2 <a
href="mainch2.html#x7-220002" id="QQ2-7-25">Memory Reclamation Policies</a></span>
<br />    <span class="sectionToc" >2.3 <a
href="mainch2.html#x7-250003">VMware Distributed Resource Management</a></span>
<br />     <span class="subsectionToc" >2.3.1 <a
href="mainch2.html#x7-260001" id="QQ2-7-29">Resource Model</a></span>
<br />     <span class="subsectionToc" >2.3.2 <a
href="mainch2.html#x7-270002" id="QQ2-7-30">DRS Algorithm</a></span>
<br />     <span class="subsectionToc" >2.3.3 <a
href="mainch2.html#x7-280003" id="QQ2-7-31">Limitations of VMware DRS</a></span>
<br />    <span class="sectionToc" >2.4 <a
href="mainch2.html#x7-290004">Other Works</a></span>
<br />   <span class="chapterToc" >3 <a
href="mainch3.html#x8-310003">Architecture and Design of DRS</a></span>


<br />    <span class="sectionToc" >3.1 <a
href="mainch3.html#x8-320001">Functions of DRS</a></span>
<br />    <span class="sectionToc" >3.2 <a
href="mainch3.html#x8-330002">Goals and Non-Goals</a></span>
<br />    <span class="sectionToc" >3.3 <a
href="mainch3.html#x8-340003">Architecture Overview</a></span>
<br />     <span class="subsectionToc" >3.3.1 <a
href="mainch3.html#x8-350001" id="QQ2-8-39">The Monitoring Service</a></span>
<br />     <span class="subsectionToc" >3.3.2 <a
href="mainch3.html#x8-360002" id="QQ2-8-40">The Memory Balancing Service</a></span>
<br />     <span class="subsectionToc" >3.3.3 <a
href="mainch3.html#x8-370003" id="QQ2-8-41">The Migration Service</a></span>
<br />   <span class="chapterToc" >4 <a
href="mainch4.html#x9-390004">Implementation of Monitoring and Auto-Ballooning</a></span>
<br />    <span class="sectionToc" >4.1 <a
href="mainch4.html#x9-400001">Monitoring</a></span>
<br />     <span class="subsectionToc" >4.1.1 <a
href="mainch4.html#x9-410001" id="QQ2-9-46">Host Monitoring</a></span>
<br />     <span class="subsectionToc" >4.1.2 <a
href="mainch4.html#x9-440002" id="QQ2-9-49">Guest Monitoring</a></span>
<br />     <span class="subsectionToc" >4.1.3 <a
href="mainch4.html#x9-470003" id="QQ2-9-52">Hotspot Detection and Key-Value Store Updation</a></span>
<br />    <span class="sectionToc" >4.2 <a
href="mainch4.html#x9-480002">Auto-Ballooning</a></span>
<br />     <span class="subsectionToc" >4.2.1 <a
href="mainch4.html#x9-490001" id="QQ2-9-54">Hard Ballooning and Soft Ballooning</a></span>
<br />     <span class="subsectionToc" >4.2.2 <a
href="mainch4.html#x9-500002" id="QQ2-9-55">Auto-Ballooning Algorithm</a></span>
<br />   <span class="chapterToc" >5 <a
href="mainch5.html#x10-520005">Experimental Evaluation</a></span>
<br />    <span class="sectionToc" >5.1 <a
href="mainch5.html#x10-530001">Experimental Setup</a></span>
<br />    <span class="sectionToc" >5.2 <a
href="mainch5.html#x10-540002">Results</a></span>
<br />     <span class="subsectionToc" >5.2.1 <a
href="mainch5.html#x10-550001" id="QQ2-10-61">Analyzing Auto-Ballooning</a></span>
<br />     <span class="subsectionToc" >5.2.2 <a
href="mainch5.html#x10-560002" id="QQ2-10-62">Analyzing CPU Hotspot Detection</a></span>

<br />
<span class="chapterToc" >6 <a
href="mainch6.html#x11-600006" id="QQ2-10-63">Conclusions</a></span>

<br />    <span class="sectionToc" >6.1 <a
href="mainch6.html#x11-610001" id="QQ2-10-64">Summary</a></span>
<br />    <span class="sectionToc" >6.2 <a
href="mainch6.html#x11-620002" id="QQ2-10-65">Future Work</a></span>
<br />     <span class="subsectionToc" >6.2.1 <a
href="mainch6.html#x11-630001" id="QQ2-10-66">Limitation of Memory Ballooning</a></span>
<br />     <span class="subsectionToc" >6.2.2 <a
href="mainch6.html#x11-640002" id="QQ2-10-67">Supporting More Resource Types</a></span>
<br />     <span class="subsectionToc" >6.2.3 <a
href="mainch6.html#x11-650003" id="QQ2-10-68">Multiple Migrations</a></span>
<br />
<span class="likechapterToc" ><a
href="mainli5.html#x12-660003" id="QQ2-2-69">References</a></span>
<br />


</div>


<!--l. 92--><div class="crosslinks"><p class="noindent">[<a
href="mainli3.html" >next</a>] [<a
href="mainli1.html" >prev</a>] [<a
href="mainli1.html#tailmainli1.html" >prev-tail</a>] [<a
href="mainli2.html" >front</a>] [<a
href="main.html#mainli2.html" >up</a>] </p></div>
<p>   <a
id="tailmainli2.html"></a> </p>
{{< /raw >}}
