+++
title = "Implementation of Monitoring and Auto-Ballooning"
menu = "main"
url = "thesis/mainch4.html"
+++
 {{< raw >}}
 <div class="crosslinks"><p class="noindent">[<a
 href="mainch5.html" >next</a>] [<a
 href="mainch3.html" >prev</a>] [<a
 href="mainch3.html#tailmainch3.html" >prev-tail</a>] [<a
 href="#tailmainch4.html">tail</a>] [<a
 href="main.html#mainch4.html" >up</a>] </p></div>
 <h2 class="chapterHead"><span class="titlemark">Chapter 4</span><br /><a
 href="main.html#QQ2-9-44" id="x9-390004">Implementation of Monitoring and Auto-Ballooning</a></h2>
 <p>For the purpose of this thesis, we describe the implementation and experimentally
 evaluate only the monitoring and autoballooning parts of the DRS. The following sections
 describe the implementation details of these two components.
 <a
 id="x9-39001r38"></a>
 </p>

 <h3 class="sectionHead"><span class="titlemark">4.1   </span> <a
 href="main.html#QQ2-9-45" id="x9-400001">Monitoring</a></h3>
 <p>As we discussed in Chapter <a
 href="mainch3.html#x8-310003">3<!--tex4ht:ref: chap:design --></a>, the monitoring component runs on each host and monitors
 the host and the guests running on that host to detect hotspots. The next few sections
 discuss which metrics are monitored, why they are monitored, and how they are
 monitored. Our implementation is done in the <em><span
 class="cmti-12">Python programming language</span></em>. The
 monitoring service runs periodically in every ﬁxed time interval which is conﬁgurable. The
 default interval is set to then seconds. We will refer to this time interval as the <em><span
 class="cmti-12">monitor</span>
 <span
 class="cmti-12">interval</span></em> in the rest of this document.
 <a
 id="x9-40001r41"></a>
 </p>

 <h4 class="subsectionHead"><span class="titlemark">4.1.1   </span> <a
 href="mainli2.html#QQ2-9-46" id="x9-410001">Host Monitoring</a></h4>
 <p>Monitoring the host is simple because the Linux kernel exposes much of the needed
 information stored in its internal data structure through procfs [<a id="page.41"></a><a
 href="mainli5.html#X0-procfs" >22</a>]. Procfs is a ﬁlesystem
 in the Linux kernel, and hence can be accessed using the standard ﬁlesystem syscalls or
 through higher level API&#x2019;s that exist in almost all programming languages for this
 purpose. Since the DRS is running on the host, it has unhindered access to all the
 information in procfs.
 </p>

 <h5 class="subsubsectionHead"><a
 href="#x9-420001" id="x9-420001">CPU Monitoring</a></h5>
 <p>For monitoring the CPU usage, we use the information in the <em><span
 class="cmti-12">/proc/stat</span></em> ﬁle. The


 <em><span
 class="cmti-12">/proc/stat</span></em> ﬁle has the following ﬁelds capturing how much time CPU spent doing what
 task [<a
 href="mainli5.html#X0-procfs" >22</a>].
 </p>

    <ul class="itemize1">
    <li class="itemize"><strong><span
 class="cmbx-12">user:</span></strong> time spent in executing normal processes in the user mode
    </li>
    <li class="itemize"><strong><span
 class="cmbx-12">nice:</span></strong> time spent in executing processes with positive nice value in the user
    mode
    </li>
    <li class="itemize"><strong><span
 class="cmbx-12">system:</span></strong> time spent in executing processes in kernel mode
    </li>
    <li class="itemize"><strong><span
 class="cmbx-12">idle:</span></strong> time for which the cpu was idle
    </li>
    <li class="itemize"><strong><span
 class="cmbx-12">iowait:</span></strong> time spent in waiting for I/O to complete
    </li>
    <li class="itemize"><strong><span
 class="cmbx-12">irq:</span></strong> time spent in servicing interrupts
    </li>
    <li class="itemize"><strong><span
 class="cmbx-12">softirq:</span></strong> time spent in servicing software interrupts
    </li>
    <li class="itemize"><strong><span
 class="cmbx-12">steal:</span></strong> time spent waiting involuntarily. This metric is relevant when the kernel
    running under virtualization, and its vCPU thread has to wait in the ready
    queue because the host CPU is overloaded.
    </li>
    <li class="itemize"><strong><span
 class="cmbx-12">guest:</span></strong> time spent running a normal guest (VM)
    </li>
    <li class="itemize"><strong><span
 class="cmbx-12">guest</span><span
 class="cmbx-12">_nice:</span></strong> time spent running a guest (VM) with positive nice value.</li></ul>


 <p>The time is measured in kernel jiﬃes. Jiﬀy is a counter that ticks on every timer interrupt.
 The interrupt frequency is 100Hz in majority of the systems. From the above
 values, we can get the total time that has passed, and time for which the cpu was
 busy.
 <!--tex4ht:inline--></p>
 <!--l. 29--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" >
 <mtable
 class="multline-star">
 <mtr><mtd
 class="multline-star"><mi
 >T</mi><mi
 >o</mi><mi
 >t</mi><mi
 >a</mi><mi
 >l</mi><mi
 >T</mi><mi
 >i</mi><mi
 >m</mi><mi
 >e</mi> <mo
 class="MathClass-rel">=</mo> <mi
 >u</mi><mi
 >s</mi><mi
 >e</mi><mi
 >r</mi> <mo
 class="MathClass-bin">+</mo> <mi
 >n</mi><mi
 >i</mi><mi
 >c</mi><mi
 >e</mi> <mo
 class="MathClass-bin">+</mo> <mi
 >s</mi><mi
 >y</mi><mi
 >s</mi><mi
 >t</mi><mi
 >e</mi><mi
 >m</mi> <mo
 class="MathClass-bin">+</mo> <mi
 >i</mi><mi
 >d</mi><mi
 >l</mi><mi
 >e</mi> <mo
 class="MathClass-bin">+</mo> <mi
 >i</mi><mi
 >o</mi><mi
 >w</mi><mi
 >a</mi><mi
 >i</mi><mi
 >t</mi>
 </mtd></mtr><mtr><mtd
 class="multline-star"> <mo
 class="MathClass-bin">+</mo> <mi
 >i</mi><mi
 >r</mi><mi
 >q</mi> <mo
 class="MathClass-bin">+</mo> <mi
 >i</mi><mi
 >o</mi><mi
 >w</mi><mi
 >a</mi><mi
 >i</mi><mi
 >t</mi> <mo
 class="MathClass-bin">+</mo> <mi
 >i</mi><mi
 >r</mi><mi
 >q</mi> <mo
 class="MathClass-bin">+</mo> <mi
 >s</mi><mi
 >o</mi><mi
 >f</mi><mi
 >t</mi><mi
 >i</mi><mi
 >r</mi><mi
 >q</mi> <mo
 class="MathClass-bin">+</mo> <mi
 >s</mi><mi
 >t</mi><mi
 >e</mi><mi
 >a</mi><mi
 >l</mi> <mo
 class="MathClass-bin">+</mo> <mi
 >g</mi><mi
 >u</mi><mi
 >e</mi><mi
 >s</mi><mi
 >t</mi> <mo
 class="MathClass-bin">+</mo> <mi
 >g</mi><mi
 >u</mi><mi
 >e</mi><mi
 >s</mi><mi
 >t</mi><mstyle
 class="text"><mtext  >_</mtext></mstyle><mi
 >n</mi><mi
 >i</mi><mi
 >c</mi><mi
 >e</mi>                      </mtd></mtr></mtable>
 </math>
 <p>
 <!--tex4ht:inline--></p>
 <!--l. 32--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" >
   <mi
 >B</mi><mi
 >u</mi><mi
 >s</mi><mi
 >y</mi><mi
 >T</mi><mi
 >i</mi><mi
 >m</mi><mi
 >e</mi> <mo
 class="MathClass-rel">=</mo> <mi
 >u</mi><mi
 >s</mi><mi
 >e</mi><mi
 >r</mi> <mo
 class="MathClass-bin">+</mo> <mi
 >n</mi><mi
 >i</mi><mi
 >c</mi><mi
 >e</mi> <mo
 class="MathClass-bin">+</mo> <mi
 >s</mi><mi
 >y</mi><mi
 >s</mi><mi
 >t</mi><mi
 >e</mi><mi
 >m</mi> <mo
 class="MathClass-bin">+</mo> <mi
 >i</mi><mi
 >r</mi><mi
 >q</mi> <mo
 class="MathClass-bin">+</mo> <mi
 >s</mi><mi
 >o</mi><mi
 >f</mi><mi
 >t</mi><mi
 >i</mi><mi
 >r</mi><mi
 >q</mi> <mo
 class="MathClass-bin">+</mo> <mi
 >g</mi><mi
 >u</mi><mi
 >e</mi><mi
 >s</mi><mi
 >t</mi> <mo
 class="MathClass-bin">+</mo> <mi
 >g</mi><mi
 >u</mi><mi
 >e</mi><mi
 >s</mi><mi
 >t</mi><mstyle
 class="text"><mtext  >_</mtext></mstyle><mi
 >n</mi><mi
 >i</mi><mi
 >c</mi><mi
 >e</mi>
 </math>
 <p>
 </p>
 <p>   The <!--l. 34--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi
 >T</mi><mi
 >o</mi><mi
 >t</mi><mi
 >a</mi><mi
 >l</mi><mi
 >T</mi><mi
 >i</mi><mi
 >m</mi><mi
 >e</mi></math>
 calculated here is the total time since the start of the system. Hence this will give the average


 cpu usage since the start of the system, which is not a useful metric for us. The average
 cpu usage in the previous monitor interval is more useful. To calculate that, we keep the
 <!--l. 34--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi
 >T</mi><mi
 >o</mi><mi
 >t</mi><mi
 >a</mi><mi
 >l</mi><mi
 >T</mi><mi
 >i</mi><mi
 >m</mi><mi
 >e</mi></math> and
 <!--l. 34--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi
 >B</mi><mi
 >u</mi><mi
 >s</mi><mi
 >y</mi><mi
 >T</mi><mi
 >i</mi><mi
 >m</mi><mi
 >e</mi></math> at the
 previous monitor interval and subtract those values from the current values.
 So,
 <!--tex4ht:inline--></p>
 <!--l. 35--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" >
       <mi
 >P</mi><mi
 >r</mi><mi
 >e</mi><mi
 >c</mi><mi
 >e</mi><mi
 >n</mi><mi
 >t</mi><mi
 >a</mi><mi
 >g</mi><mi
 >e</mi><mi
 >C</mi><mi
 >p</mi><mi
 >u</mi><mi
 >U</mi><mi
 >s</mi><mi
 >a</mi><mi
 >g</mi><mi
 >e</mi> <mo
 class="MathClass-rel">=</mo> <mfrac><mrow
 ><mi
 >B</mi><mi
 >u</mi><mi
 >s</mi><mi
 >y</mi><mi
 >T</mi><mi
 >i</mi><mi
 >m</mi><mi
 >e</mi> <mo
 class="MathClass-bin">−</mo> <mi
 >P</mi><mi
 >r</mi><mi
 >e</mi><mi
 >v</mi><mi
 >i</mi><mi
 >o</mi><mi
 >u</mi><mi
 >s</mi><mi
 >B</mi><mi
 >u</mi><mi
 >s</mi><mi
 >y</mi><mi
 >T</mi><mi
 >i</mi><mi
 >m</mi><mi
 >e</mi></mrow>
 <mrow
 ><mi
 >T</mi><mi
 >o</mi><mi
 >t</mi><mi
 >a</mi><mi
 >l</mi><mi
 >T</mi><mi
 >i</mi><mi
 >m</mi><mi
 >e</mi> <mo
 class="MathClass-bin">−</mo> <mi
 >P</mi><mi
 >r</mi><mi
 >e</mi><mi
 >v</mi><mi
 >i</mi><mi
 >o</mi><mi
 >u</mi><mi
 >s</mi><mi
 >T</mi><mi
 >o</mi><mi
 >t</mi><mi
 >a</mi><mi
 >l</mi><mi
 >T</mi><mi
 >i</mi><mi
 >m</mi><mi
 >e</mi></mrow></mfrac> <mo
 class="MathClass-bin">∗</mo> <mn>1</mn><mn>0</mn><mn>0</mn>
 </math>
 <p>
 </p>

 <h5 class="subsubsectionHead"><a
 href="#x9-430001" id="x9-430001">Memory Monitoring</a></h5>
 <p>Memory usage information is present in the <em><span
 class="cmti-12">/proc/meminfo</span></em> ﬁle [<a id="page.44"></a><a
 href="mainli5.html#X0-procfs" >22</a>]. For monitoring, we
 need several memory metrics which are described below. </p>

    <ul class="itemize1">
    <li class="itemize"><strong><span
 class="cmbx-12">Total memory:</span></strong> The total usable memory present on the host. Present in the
    ﬁrst line of the <em><span
 class="cmti-12">/proc/meminfo</span></em>.
    </li>
    <li class="itemize"><strong><span
 class="cmbx-12">Used memory:</span></strong> Total memory used by the host. This includes the memory
    used by the VMs, host&#x2019;s own processes and the page caches. This can be
    calculated by subtracting free memory, which is present in the second line of
    <em><span
 class="cmti-12">/proc/meminfo</span></em> ﬁle, from total memory.
    </li>


    <li class="itemize"><strong><span
 class="cmbx-12">Available  memory:</span></strong> This  provides  an  estimate  of  how  much  memory  is
    available to any new application that will be started on the system. This is
    diﬀerent from free memory because used memory also contains page caches,
    which can be reclaimed in case no free memory is left and an application
    requires more memory. Some of the slab memory used by the kernel is also
    reclaimable. The estimate also accounts for the fact that the system needs
    some page cache to function well, and that not all reclaimable slab can be
    reclaimed. It is present in the third line of <em><span
 class="cmti-12">/proc/meminfo</span></em> ﬁle.
    <p>Available memory is an important metric for us, because it basically tells us
    how much memory is available for the new VMs. It is better than free memory,
    because free memory does not take into account page caches which can be
    reclaimed.
    </p>
 </li>
    <li class="itemize"><strong><span
 class="cmbx-12">Hypervisor Load:</span></strong> This is an important metric because we want to have a
    lower bound on how much memory is available for the hypervisor to function.
    We  call  this  memory  <em><span
 class="cmti-12">hypervisor  reserved  memory</span></em>.  While  calculating  the
    amount of memory that is free for VMs to use, we do not want to use the
    memory that is reserved for the hypervisor.
    <p>For calculating the hypervisor load, we ﬁrst calculate the VM load i.e. the
    amount of memory used by the virtual machines on the system. The VM load is
    calculated by looking at the RSS (resident set size) of all the virtual machines
    running on the host. Since QEMU-KVM treats each VM as a separate process,
    we  need  to  get  the  RSS  of  all  the  processes  which  are  running  a  virtual
    machine. The <em><span
 class="cmti-12">/proc/[pid]/statm</span></em> ﬁle has this information for each process. After
    calculating the VMlLoad, the hypervisor load is calculated as


    <!--tex4ht:inline--></p>
 <!--l. 50--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" >
 <mi
 >H</mi><mi
 >y</mi><mi
 >p</mi><mi
 >e</mi><mi
 >r</mi><mi
 >v</mi><mi
 >i</mi><mi
 >s</mi><mi
 >o</mi><mi
 >r</mi><mi
 >L</mi><mi
 >o</mi><mi
 >a</mi><mi
 >d</mi> <mo
 class="MathClass-rel">=</mo> <mi
 >m</mi><mi
 >a</mi><mi
 >x</mi><mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >H</mi><mi
 >y</mi><mi
 >p</mi><mi
 >e</mi><mi
 >r</mi><mi
 >v</mi><mi
 >i</mi><mi
 >s</mi><mi
 >o</mi><mi
 >r</mi><mi
 >R</mi><mi
 >e</mi><mi
 >s</mi><mi
 >e</mi><mi
 >r</mi><mi
 >v</mi><mi
 >e</mi><mi
 >d</mi><mo
 class="MathClass-punc">,</mo><mi
 >U</mi><mi
 >s</mi><mi
 >e</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi><mo
 class="MathClass-bin">−</mo><mi
 >P</mi><mi
 >a</mi><mi
 >g</mi><mi
 >e</mi><mi
 >C</mi><mi
 >a</mi><mi
 >c</mi><mi
 >h</mi><mi
 >e</mi><mi
 >s</mi><mo
 class="MathClass-bin">−</mo><mi
 >V</mi> <mi
 >M</mi><mi
 >L</mi><mi
 >o</mi><mi
 >a</mi><mi
 >d</mi></mrow><mo
 class="MathClass-close">)</mo></mrow>
 </math>
    <p>
    </p>
 </li>
    <li class="itemize"><strong><span
 class="cmbx-12">Load Memory: </span></strong> This metric represents the total memory that is loaded and
    hence, not available for use by any new VM or for existing VMs. It is calculated
    as
    <!--tex4ht:inline--><!--l. 53--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" >
      <mi
 >H</mi><mi
 >y</mi><mi
 >p</mi><mi
 >e</mi><mi
 >r</mi><mi
 >v</mi><mi
 >i</mi><mi
 >s</mi><mi
 >o</mi><mi
 >r</mi><mi
 >E</mi><mi
 >x</mi><mi
 >t</mi><mi
 >r</mi><mi
 >a</mi> <mo
 class="MathClass-rel">=</mo> <mi
 >m</mi><mi
 >a</mi><mi
 >x</mi><mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >H</mi><mi
 >y</mi><mi
 >p</mi><mi
 >e</mi><mi
 >r</mi><mi
 >v</mi><mi
 >i</mi><mi
 >s</mi><mi
 >o</mi><mi
 >r</mi><mi
 >R</mi><mi
 >e</mi><mi
 >s</mi><mi
 >e</mi><mi
 >r</mi><mi
 >v</mi><mi
 >e</mi><mi
 >d</mi> <mo
 class="MathClass-bin">−</mo> <mi
 >H</mi><mi
 >y</mi><mi
 >p</mi><mi
 >e</mi><mi
 >r</mi><mi
 >v</mi><mi
 >i</mi><mi
 >s</mi><mi
 >o</mi><mi
 >r</mi><mi
 >L</mi><mi
 >o</mi><mi
 >a</mi><mi
 >d</mi><mo
 class="MathClass-punc">,</mo> <mn>0</mn></mrow><mo
 class="MathClass-close">)</mo></mrow>
 </math>
    <p>
    <!--tex4ht:inline--></p>
 <!--l. 55--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" >
 <mi
 >L</mi><mi
 >o</mi><mi
 >a</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-rel">=</mo> <mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >T</mi><mi
 >o</mi><mi
 >t</mi><mi
 >a</mi><mi
 >l</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi><mo
 class="MathClass-bin">−</mo><mi
 >A</mi><mi
 >v</mi><mi
 >a</mi><mi
 >i</mi><mi
 >l</mi><mi
 >a</mi><mi
 >b</mi><mi
 >l</mi><mi
 >e</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi><mo
 class="MathClass-bin">−</mo><mi
 >I</mi><mi
 >d</mi><mi
 >l</mi><mi
 >e</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi></mrow><mo
 class="MathClass-close">)</mo></mrow><mo
 class="MathClass-bin">+</mo><mi
 >H</mi><mi
 >y</mi><mi
 >p</mi><mi
 >e</mi><mi
 >r</mi><mi
 >v</mi><mi
 >i</mi><mi
 >s</mi><mi
 >o</mi><mi
 >r</mi><mi
 >E</mi><mi
 >x</mi><mi
 >t</mi><mi
 >r</mi><mi
 >a</mi>
 </math>
    <p>


    </p>
 </li>
    <li class="itemize"><strong><span
 class="cmbx-12">Idle Memory: </span></strong> Idle Memory is the memory which has been consumed by the
    VMs but is sitting idle inside the VM and can be ballooned out. Thus, it can
    be used by the existing VMs or any new VM that is migrated to the host. Its
    calculation is described later in the Section <a
 href="#x9-440002">4.1.2<!--tex4ht:ref: sec:guestmon --></a>.</li></ul>
 <a
 id="x9-43001r46"></a>
 <h4 class="subsectionHead"><span class="titlemark">4.1.2   </span> <a
 href="mainli2.html#QQ2-9-49" id="x9-440002">Guest Monitoring</a></h4>
 <p>Monitoring a VM is more complex than monitoring the host because some of the metrics
 needed are not exposed by the VM to the host in any conventional way. Host operating
 system just gets to see the process abstraction of each virtual machine. So, the host has
 only as much information about each guest as it has about the rest of the processes
 running on it. In the next two sections, along with which metrics have been monitored, we
 have discuss some new techniques which we use to get some of the information from the
 guest VMs.
 </p>

 <h5 class="subsubsectionHead"><a
 href="#x9-450002" id="x9-450002">CPU Monitoring</a></h5>
 <p>CPU monitoring for a guest VM is straightforward because the host OS has the
 scheduling information for each process it is running. This information is exposed by the
 procfs. The metrics related to CPU utilization measured are discussed below.
 </p>

    <ul class="itemize1">
    <li class="itemize"><strong><span
 class="cmbx-12">Busy  Time:</span></strong>  This  is  the  time  for  which  any  of  the  vCPUs  of  the
    VM  was  scheduled  onto  any  of  the  physical  CPUs  of  the  host.  The
    scheduling  information  for  each  thread  of  a  process  is  present  inside  the
    <em><span
 class="cmti-12">/proc/[pid]/task/[tid]/schedstat</span></em> ﬁle [<a id="page.47"></a><a
 href="mainli5.html#X0-procfs" >22</a>]. We can get the busy time for each
    thread in nanoseconds from here. Adding the busy time for all the vCPU
    threads gives us the total busy time. The total time elasped has already been
    calculated in Section <a
 href="#x9-420001">4.1.1<!--tex4ht:ref: sec:cpumon --></a>. We do calculation similar to Section <a
 href="#x9-420001">4.1.1<!--tex4ht:ref: sec:cpumon --></a> to get


    the busy time percentage in previous monitor interval for all the guests.
    </li>
    <li class="itemize"><strong><span
 class="cmbx-12">Steal Time:</span></strong> Steal time [<a id="page.48"></a><a
 href="mainli5.html#X0-ehrhardt2013cpu" >23</a>] is the time for which a vCPU thread waits in the
    ready queue while the CPU is busy executing some other process. Steal time
    has direct correlation to the amount of load on CPU and is useful in predicting
    whether the CPU is overloaded. Steal times for each thread is present inside
    the <em><span
 class="cmti-12">/proc/[pid]/task/[tid]/schedstat</span></em> [<a
 href="mainli5.html#X0-procfs" >22</a>]. For each VM, we add the steal times
    of all its vCPU threads and calculate the average steal time percentage in the
    previous monitor interval.
    </li>
    <li class="itemize"><strong><span
 class="cmbx-12">Estimated CPU Demand:</span></strong> Estimated CPU demand is useful for predicting
    the amount of CPU a VM can consume if the host is not overloaded i.e. the
    steal time is 0. This value will be helpful during migration to check whether the
    CPU demands of the VM can be satisﬁed by the destination. It is calculated
    by adding a scaled value of steal percentage to the current busy percentage.
    <!--tex4ht:inline--><!--l. 69--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" >
     <mi
 >E</mi><mi
 >s</mi><mi
 >t</mi><mi
 >i</mi><mi
 >m</mi><mi
 >a</mi><mi
 >t</mi><mi
 >e</mi><mi
 >d</mi><mi
 >C</mi><mi
 >P</mi><mi
 >U</mi><mi
 >D</mi><mi
 >e</mi><mi
 >m</mi><mi
 >a</mi><mi
 >n</mi><mi
 >d</mi><mi
 >P</mi><mi
 >r</mi><mi
 >e</mi><mi
 >c</mi><mi
 >e</mi><mi
 >n</mi><mi
 >t</mi> <mo
 class="MathClass-rel">=</mo> <mfrac><mrow
 ><mi
 >B</mi><mi
 >u</mi><mi
 >s</mi><mi
 >y</mi><mi
 >T</mi><mi
 >i</mi><mi
 >m</mi><mi
 >e</mi></mrow>
 <mrow
 ><mi
 >T</mi><mi
 >o</mi><mi
 >t</mi><mi
 >a</mi><mi
 >l</mi><mi
 >T</mi><mi
 >i</mi><mi
 >m</mi><mi
 >e</mi></mrow></mfrac> <mo
 class="MathClass-bin">∗</mo> <mrow ><mo
 class="MathClass-open">(</mo><mrow><mn>1</mn> <mo
 class="MathClass-bin">+</mo> <mfrac><mrow
 ><mi
 >S</mi><mi
 >t</mi><mi
 >e</mi><mi
 >a</mi><mi
 >l</mi><mi
 >T</mi><mi
 >i</mi><mi
 >m</mi><mi
 >e</mi></mrow>
 <mrow
 ><mi
 >T</mi><mi
 >o</mi><mi
 >t</mi><mi
 >a</mi><mi
 >l</mi><mi
 >T</mi><mi
 >i</mi><mi
 >m</mi><mi
 >e</mi></mrow></mfrac></mrow><mo
 class="MathClass-close">)</mo></mrow> <mo
 class="MathClass-bin">∗</mo> <mn>1</mn><mn>0</mn><mn>0</mn>
 </math>
    <p></p>
 </li></ul>
 <h5 class="subsubsectionHead"><a
 href="#x9-460002" id="x9-460002">Memory Monitoring</a></h5>
 <p>Getting the memory usage statistics is diﬃcult because the host has no information about
 the memory management of the guest. The host only knows the amount of memory
 allocated by each guest. For calculating the idle memory of each guest, we need the


 amount of <em><span
 class="cmti-12">available memory</span></em> inside the VM.
 </p>
 <p>   One way for a guest VM to expose some information to the host is through a device
 driver. In a virtualized environment, all the devices available to a VM are virtual devices
 which are emulated by the hypervisor. So, a device driver running inside the VM can send
 some information to the device which is emulated by the hypervisor. The virtio balloon
 driver in the Linux kernel has a way of exposing the memory statistics to the host. It
 gives the total memory, free memory, swap in, major page faults, minor page fault
 metrics. We have modiﬁed the balloon driver to expose the available memory metric to
 the host too. We have also modiﬁed the backend virtio balloon hardware in
 QEMU to take the Available Memory metric from the balloon driver inside
 the guest. The following metrics related to the memory usage of the VMs are
 monitored. Figure <a
 href="#x9-46001r1">4.1<!--tex4ht:ref: fig:mem1 --></a> shows the relative sizes of diﬀerent memory metrics calculated.
 </p>
 <hr class="figure" /><div class="figure"
 >


 <a
 id="x9-46001r1"></a>



 <p><img
 src="/images/thesis/mem1.png" alt="PIC"  
 />
 <a
 id="x9-46002"></a>
 <br /> </p>
 <div class="caption"
 ><span class="id">Figure 4.1: </span><span  
 class="content">Relative sizes of diﬀerent types of memory metrics</span></div><!--tex4ht:label?: x9-46001r4 -->


 </div><hr class="endfigure" />
    <ul class="itemize1">
    <li class="itemize"><strong><span
 class="cmbx-12">Maximum Memory:</span></strong> This is the maximum memory that the guest can have.
    The guest cannot be ballooned up after this point. We obtain this metric by
    using the API QEMU provides for this purpose.
    </li>
    <li class="itemize"><strong><span
 class="cmbx-12">Current Memory:</span></strong> This the amount of memory the guest has after taking
    the ballooned memory into account. From the point of view of the guest, this
    is its total memory, which can be increased or decreased by ballooning. The
    maximum limit for the current memory is the maximum memory. The <em><span
 class="cmti-12">total</span>
    <span
 class="cmti-12">memory</span></em> statistic obtained through the virtio balloon driver represents this
    metric.
    </li>
    <li class="itemize"><strong><span
 class="cmbx-12">Used Memory:</span></strong> This is the memory which is being used by the guest. It
    includes all the page caches along with the memory being used by the processes
    inside the guest.
    </li>
    <li class="itemize"><strong><span
 class="cmbx-12">Available Memory:</span></strong> This is the amount of available memory inside the guest.
    The concept of available memory has been discussed in Section <a
 href="#x9-430001">4.1.1<!--tex4ht:ref: impl:hostmemmon --></a>. We have
    modiﬁed the virtio balloon driver to obtain this metric.
    </li>
    <li class="itemize"><strong><span
 class="cmbx-12">Allocated Memory:</span></strong> This is the amount of memory of the guest that is
    backed by physical memory on the host. Allocated memory is diﬀerent from
    current memory because the memory is allocated to the vritual machines on
    demand. This also may not equal to the used memory of the guest. Without
    ballooning, the memory, once allocated to a guest, is not reclaimed from it.
    So, the usage of a guest may keep on changing, but the allocated memory


    will always be equal to the maximum used memory in the guest&#x2019;s lifetime.
    Allocated memory is helpful in calculating the idle memory.
    <p>The allocated memory is calculated by looking at the virtual memory maps
    of the QEMU-KVM process corresponding to each VM. This information is
    present inside the <em><span
 class="cmti-12">/proc/[pid]/smaps</span></em> ﬁle [<a id="page.53"></a><a
 href="mainli5.html#X0-procfs" >22</a>] for each process. The RAM of
    the guest VM is allocated by the QEMU process by doing a malloc of the size
    of maximum memory. Thus, it is one big contiguous chunk of memory in the
    heap memory of the process. From the information about diﬀerent memory
    sections of the process present in this ﬁle, we look for the heap memory chunk
    of the size of VM&#x2019;s maximum memory. The RSS (resident set size) of this
    chunk of memory gives us the amount of memory in the VM that is backed
    by physical storage.
    </p>
 <p>In this chunk of allocated memory, there is some amount of memory that
    QEMU uses to store some metadata about the pages of the guest VM. Hence,
    this entire memory is not available to the guest. This part of memory cannot
    be reclaimed by ballooning. This overhead is constant and depends on the
    maximum memory size of the guest. We call this overhead as QEMU overhead.
    QEMU overhead needs to be deducted from the RSS. We use a piecewise
    continuous function to model the QEMU overhead. The values for the QEMU
    overhead in diﬀerent scenarios were determined experimentally.
 </p>
 <div class="math-display"><!--l. 92--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" ><mrow
 >
 <mi
 >Q</mi><mi
 >E</mi><mi
 >M</mi><mi
 >U</mi><mi
 >O</mi><mi
 >v</mi><mi
 >e</mi><mi
 >r</mi><mi
 >h</mi><mi
 >e</mi><mi
 >a</mi><mi
 >d</mi> <mo
 class="MathClass-rel">=</mo>  <mfenced separators=""
 open="{"  close="" ><mrow> <mtable  align="axis" style=""  
 equalrows="false" columnlines="none" equalcolumns="false" class="array"><mtr><mtd
 class="array"  columnalign="left"><mn>2</mn><mn>0</mn><mn>0</mn><mi
 >M</mi><mi
 >B</mi><mspace width="1em" class="quad"/></mtd><mtd
 class="array"  columnalign="left"><mstyle
 class="text"><mtext  >if </mtext><mstyle
 class="math"><mi
 >m</mi><mi
 >a</mi><mi
 >x</mi><mi
 >m</mi><mi
 >e</mi><mi
 >m</mi> <mo
 class="MathClass-rel">&#x003C;</mo><mo
 class="MathClass-rel">=</mo> <mn>4</mn><mi
 >G</mi><mi
 >B</mi></mstyle><mtext  ></mtext></mstyle>           </mtd></mtr><mtr><mtd
 class="array"  columnalign="left"> <mn>3</mn><mn>0</mn><mn>0</mn><mi
 >M</mi><mi
 >B</mi><mspace width="1em" class="quad"/></mtd> <mtd
 class="array"  columnalign="left"><mstyle
 class="text"><mtext  >if </mtext><mstyle
 class="math"><mn>4</mn><mi
 >G</mi><mi
 >B</mi> <mo
 class="MathClass-rel">&#x003C;</mo> <mi
 >m</mi><mi
 >a</mi><mi
 >x</mi><mi
 >m</mi><mi
 >e</mi><mi
 >m</mi> <mo
 class="MathClass-rel">&#x003C;</mo><mo
 class="MathClass-rel">=</mo> <mn>1</mn><mn>0</mn><mi
 >G</mi><mi
 >B</mi></mstyle><mtext  ></mtext></mstyle></mtd>
 </mtr><mtr><mtd
 class="array"  columnalign="left"> <mn>4</mn><mn>0</mn><mn>0</mn><mi
 >M</mi><mi
 >B</mi><mspace width="1em" class="quad"/></mtd><mtd
 class="array"  columnalign="left"><mstyle
 class="text"><mtext  >if </mtext><mstyle
 class="math"><mn>1</mn><mn>0</mn><mi
 >G</mi><mi
 >B</mi> <mo
 class="MathClass-rel">&#x003C;</mo> <mi
 >m</mi><mi
 >a</mi><mi
 >x</mi><mi
 >m</mi><mi
 >e</mi><mi
 >m</mi> <mo
 class="MathClass-rel">&#x003C;</mo><mo
 class="MathClass-rel">=</mo> <mn>1</mn><mn>8</mn><mi
 >G</mi><mi
 >B</mi></mstyle><mtext  ></mtext></mstyle></mtd></mtr><mtr><mtd
 class="array"  columnalign="left"> <mn>5</mn><mn>0</mn><mn>0</mn><mi
 >M</mi><mi
 >B</mi><mspace width="1em" class="quad"/></mtd> <mtd
 class="array"  columnalign="left"><mstyle
 class="text"><mtext  >if </mtext><mstyle
 class="math"><mi
 >m</mi><mi
 >a</mi><mi
 >x</mi><mi
 >m</mi><mi
 >e</mi><mi
 >m</mi> <mo
 class="MathClass-rel">&#x003E;</mo> <mn>1</mn><mn>8</mn><mi
 >G</mi><mi
 >B</mi></mstyle><mtext  ></mtext></mstyle></mtd>
 </mtr><mtr><mtd
 class="array"  columnalign="left">         <mspace width="1em" class="quad"/></mtd></mtr><!--@{}l@{\quad }l@{}--></mtable>                                                                                   </mrow></mfenced>
 </mrow></math></div>


    <p>
    <!--tex4ht:inline--></p>
 <!--l. 101--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" >
                  <mi
 >A</mi><mi
 >l</mi><mi
 >l</mi><mi
 >o</mi><mi
 >c</mi><mi
 >a</mi><mi
 >t</mi><mi
 >e</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-rel">=</mo> <mi
 >R</mi><mi
 >S</mi><mi
 >S</mi> <mo
 class="MathClass-bin">−</mo> <mi
 >Q</mi><mi
 >E</mi><mi
 >M</mi><mi
 >U</mi><mi
 >O</mi><mi
 >v</mi><mi
 >e</mi><mi
 >r</mi><mi
 >h</mi><mi
 >e</mi><mi
 >a</mi><mi
 >d</mi>
 </math>
    <p>
    </p>
 </li>
    <li class="itemize"><strong><span
 class="cmbx-12">Load Memory:</span></strong> This is the amount of memory that is loaded i.e. is being
    used by the processes of the guest and hence the current memory should not
    go below this point.
    <!--tex4ht:inline--><!--l. 103--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" >
                <mi
 >L</mi><mi
 >o</mi><mi
 >a</mi><mi
 >d</mi><mi
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
 >t</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-bin">−</mo> <mi
 >A</mi><mi
 >v</mi><mi
 >a</mi><mi
 >i</mi><mi
 >l</mi><mi
 >a</mi><mi
 >b</mi><mi
 >l</mi><mi
 >e</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi>
 </math>
    <p>
    </p>
 </li>
    <li class="itemize"><strong><span
 class="cmbx-12">Idle Memory:</span></strong> Idle memory is the amount of memory that is allocated, but
    is not being used by the guest, and hence, can be reclaimed. A lower bound on
    the current memory of the guest has been kept to avoid ballooning so much
    memory out of it that it is unable to function. This bound is called <em><span
 class="cmti-12">guest</span>


    <span
 class="cmti-12">reserved</span></em>. So,
    <!--tex4ht:inline--><!--l. 105--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" >
                  <mi
 >L</mi><mi
 >o</mi><mi
 >w</mi><mi
 >e</mi><mi
 >r</mi><mi
 >B</mi><mi
 >o</mi><mi
 >u</mi><mi
 >n</mi><mi
 >d</mi> <mo
 class="MathClass-rel">=</mo> <mi
 >m</mi><mi
 >a</mi><mi
 >x</mi><mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >L</mi><mi
 >o</mi><mi
 >a</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi><mo
 class="MathClass-punc">,</mo><mi
 >G</mi><mi
 >u</mi><mi
 >e</mi><mi
 >s</mi><mi
 >t</mi><mi
 >R</mi><mi
 >e</mi><mi
 >s</mi><mi
 >e</mi><mi
 >r</mi><mi
 >v</mi><mi
 >e</mi><mi
 >d</mi></mrow><mo
 class="MathClass-close">)</mo></mrow>
 </math>
    <p>
    <!--tex4ht:inline--></p>
 <!--l. 106--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" >
              <mi
 >I</mi><mi
 >d</mi><mi
 >l</mi><mi
 >e</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-rel">=</mo> <mi
 >m</mi><mi
 >a</mi><mi
 >x</mi><mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >A</mi><mi
 >l</mi><mi
 >l</mi><mi
 >o</mi><mi
 >c</mi><mi
 >a</mi><mi
 >t</mi><mi
 >e</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-bin">−</mo> <mi
 >L</mi><mi
 >o</mi><mi
 >w</mi><mi
 >e</mi><mi
 >r</mi><mi
 >B</mi><mi
 >o</mi><mi
 >u</mi><mi
 >n</mi><mi
 >d</mi><mo
 class="MathClass-punc">,</mo> <mn>0</mn></mrow><mo
 class="MathClass-close">)</mo></mrow>
 </math>
    <p> Based on how the idle memory is reclaimed, it can be divided into two types
    - hard idle memory and soft idle memory. Their importance and the diﬀerence
    between them have been explained in Section <a
 href="#x9-490001">4.2.1<!--tex4ht:ref: sec:bal --></a>.</p>
 </li></ul>
 <a
 id="x9-46003r49"></a>
 <h4 class="subsectionHead"><span class="titlemark">4.1.3   </span> <a
 href="mainli2.html#QQ2-9-52" id="x9-470003">Hotspot Detection and Key-Value Store Updation</a></h4>
 <p>The hotspot detection algorithm should be robust enough not to generate false alarms. A
 simple threshold based algorithm which takes absolute or average values into account can
 raise many false alarms. Andreolini et al. [<a id="page.55"></a><a
 href="mainli5.html#X0-andreolini2009dynamic" >24</a>] have shown the drawbacks of such
 algorithms and proposed a more robust statistical model to detect changes in the


 load proﬁle of a machine which is based on the CUSUM (Cumulative Sum)
 algorithm [<a
 href="mainli5.html#X0-page1957estimating" >25</a>]. We have used this algorithm for detecting hotspots and the time to
 update the key-value store. The algorithm is described below for the sake of
 completeness.
 </p>
 <p>   The hotspots due to memory overload are detected using the load memory
 metric of the host. The values for load memory are recorded in ever monitor
 interval to form a time series. Exponential average of the data is calculated
 as
 <!--tex4ht:inline--></p>
 <!--l. 114--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" >
                       <msub><mrow
 ><mi
 >μ</mi></mrow><mrow
 ><mi
 >i</mi></mrow></msub
 > <mo
 class="MathClass-rel">=</mo> <mi
 >α</mi> <mo
 class="MathClass-bin">∗</mo> <mi
 >l</mi><mi
 >o</mi><mi
 >a</mi><mi
 >d</mi><mi
 >m</mi><mi
 >e</mi><msub><mrow
 ><mi
 >m</mi></mrow><mrow
 ><mi
 >i</mi></mrow></msub
 > <mo
 class="MathClass-bin">+</mo> <mrow ><mo
 class="MathClass-open">(</mo><mrow><mn>1</mn> <mo
 class="MathClass-bin">−</mo> <mi
 >α</mi></mrow><mo
 class="MathClass-close">)</mo></mrow><msub><mrow
 ><mi
 >μ</mi></mrow><mrow
 ><mi
 >i</mi><mo
 class="MathClass-bin">−</mo><mn>1</mn></mrow></msub
 >
 </math>
 <p> The value of <!--l. 115--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi
 >α</mi></math> has been
 chosen to be <!--l. 115--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mn>0</mn><mo
 class="MathClass-punc">.</mo><mn>1</mn></math>. The algorithm
 detects abrupt increase in <!--l. 115--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><msub><mrow
 ><mi
 >μ</mi></mrow><mrow
 ><mi
 >i</mi></mrow></msub
 ></math>
 using <!--l. 115--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><msub><mrow
 ><mi
 >d</mi></mrow><mrow
 ><mi
 >i</mi></mrow></msub
 ></math>.
 <!--tex4ht:inline--></p>
 <!--l. 116--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" >
                  <msub><mrow
 ><mi
 >d</mi></mrow><mrow
 ><mn>0</mn></mrow></msub
 > <mo
 class="MathClass-rel">=</mo> <mn>0</mn><mo
 class="MathClass-punc">;</mo> <mspace width="1em" class="nbsp" /><msub><mrow
 ><mi
 >d</mi></mrow><mrow
 ><mi
 >i</mi></mrow></msub
 > <mo
 class="MathClass-rel">=</mo> <msub><mrow
 ><mi
 >d</mi></mrow><mrow
 ><mi
 >i</mi><mo
 class="MathClass-bin">−</mo><mn>1</mn></mrow></msub
 > <mo
 class="MathClass-bin">+</mo> <mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >l</mi><mi
 >o</mi><mi
 >a</mi><mi
 >d</mi><mi
 >m</mi><mi
 >e</mi><msub><mrow
 ><mi
 >m</mi></mrow><mrow
 ><mi
 >i</mi></mrow></msub
 > <mo
 class="MathClass-bin">−</mo> <mrow ><mo
 class="MathClass-open">(</mo><mrow><msub><mrow
 ><mi
 >μ</mi></mrow><mrow
 ><mi
 >i</mi></mrow></msub
 > <mo
 class="MathClass-bin">+</mo> <mi
 >K</mi></mrow><mo
 class="MathClass-close">)</mo></mrow></mrow><mo
 class="MathClass-close">)</mo></mrow>
 </math>
 <p> <!--l. 117--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><msub><mrow
 ><mi
 >d</mi></mrow><mrow
 ><mi
 >i</mi></mrow></msub
 ></math> measures all the


 deviations from <!--l. 117--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><msub><mrow
 ><mi
 >μ</mi></mrow><mrow
 ><mi
 >i</mi></mrow></msub
 ></math> that
 are greater than <!--l. 117--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi
 >K</mi></math>.
 <!--l. 117--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi
 >K</mi> <mo
 class="MathClass-rel">=</mo> <mfrac><mrow
 ><mi
 >Δ</mi></mrow>
 <mrow
 ><mn>2</mn></mrow></mfrac> </math> where
 <!--l. 117--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi
 >Δ</mi></math>
 is the minimum shift to be detected. It has been set to
 (<!--l. 118--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mn>0</mn><mo
 class="MathClass-punc">.</mo><mn>0</mn><mn>0</mn><mn>5</mn> <mo
 class="MathClass-bin">∗</mo> <mi
 >T</mi><mi
 >o</mi><mi
 >t</mi><mi
 >a</mi><mi
 >l</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi></math>) to detect minimum 1%
 change in the value of <!--l. 118--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi
 >l</mi><mi
 >o</mi><mi
 >a</mi><mi
 >d</mi><mi
 >m</mi><mi
 >e</mi><mi
 >m</mi></math>.
 Change in the load proﬁle of the host is triggered when
 <!--l. 118--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><msub><mrow
 ><mi
 >d</mi></mrow><mrow
 ><mi
 >i</mi></mrow></msub
 > <mo
 class="MathClass-rel">&#x003E;</mo> <mi
 >H</mi></math> where
 <!--l. 118--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi
 >H</mi> <mo
 class="MathClass-rel">=</mo> <mi
 >h</mi><mi
 >σ</mi></math>,
 <!--l. 118--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi
 >h</mi></math> being a design
 parameter and <!--l. 118--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi
 >σ</mi></math>
 being the standard deviation of the time series upto that point. We have set
 <!--l. 118--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi
 >h</mi> <mo
 class="MathClass-rel">=</mo> <mn>7</mn></math>,
 which means that the load proﬁle change is detected in an average of 14 samples
 [<a id="page.56"></a><a
 href="mainli5.html#X0-andreolini2009dynamic" >24</a>]. When the load-proﬁle changes, the key-value store is updated with
 <!--l. 119--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi
 >t</mi><mi
 >o</mi><mi
 >t</mi><mi
 >a</mi><mi
 >l</mi><mi
 >m</mi><mi
 >e</mi><mi
 >m</mi> <mo
 class="MathClass-bin">−</mo> <mi
 >l</mi><mi
 >o</mi><mi
 >a</mi><mi
 >d</mi><mi
 >m</mi><mi
 >e</mi><mi
 >m</mi></math>. When the load
 proﬁle changes and <!--l. 119--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mrow ><mo
 class="MathClass-open">(</mo><mrow><msub><mrow
 ><mi
 >μ</mi></mrow><mrow
 ><mi
 >i</mi></mrow></msub
 > <mo
 class="MathClass-rel">&#x003E;</mo> <mn>0</mn><mo
 class="MathClass-punc">.</mo><mn>8</mn> <mo
 class="MathClass-bin">∗</mo> <mi
 >t</mi><mi
 >o</mi><mi
 >t</mi><mi
 >a</mi><mi
 >l</mi><mi
 >m</mi><mi
 >e</mi><mi
 >m</mi></mrow><mo
 class="MathClass-close">)</mo></mrow></math>,
 hotspot is triggered and the migration service becomes active.
 </p>
 <p>   The hotspots in the CPU usage are also calculated in a similar way. The
 only diﬀerence is that the CUSUM algorithm runs on the steal time of all the
 guests, and any guest can trigger hotspot when its load proﬁle changes and its
 <!--l. 121--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi
 >A</mi><mi
 >v</mi><mi
 >e</mi><mi
 >r</mi><mi
 >a</mi><mi
 >g</mi><mi
 >e</mi><mi
 >S</mi><mi
 >t</mi><mi
 >e</mi><mi
 >a</mi><mi
 >l</mi><mi
 >T</mi><mi
 >i</mi><mi
 >m</mi><mi
 >e</mi> <mo
 class="MathClass-rel">&#x003E;</mo> <mn>1</mn><mn>0</mn><mi
 >%</mi></math>.
 Additionally, the CUSUM algorithm runs on the busy time of the host to trigger
 the updation of the key-value store. The value stored in the key-value store is
 <!--l. 121--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >N</mi><mi
 >u</mi><mi
 >m</mi><mi
 >b</mi><mi
 >e</mi><mi
 >r</mi><mi
 >O</mi><mi
 >f</mi><mi
 >C</mi><mi
 >P</mi><mi
 >U</mi><mi
 >s</mi> <mo
 class="MathClass-bin">∗</mo> <mn>1</mn><mn>0</mn><mn>0</mn> <mo
 class="MathClass-bin">−</mo> <mi
 >B</mi><mi
 >u</mi><mi
 >s</mi><mi
 >y</mi><mi
 >T</mi><mi
 >i</mi><mi
 >m</mi><mi
 >e</mi><mi
 >P</mi><mi
 >e</mi><mi
 >r</mi><mi
 >c</mi><mi
 >e</mi><mi
 >n</mi><mi
 >t</mi><mi
 >a</mi><mi
 >g</mi><mi
 >e</mi></mrow><mo
 class="MathClass-close">)</mo></mrow></math>
 <a
 id="x9-47001r45"></a>
 </p>



 <h3 class="sectionHead"><span class="titlemark">4.2   </span> <a
 href="main.html#QQ2-9-53" id="x9-480002">Auto-Ballooning</a></h3>
 <p>Ballooning is a technique for increasing or decreasing the current memory of a guest.
 Auto-Ballooning is the process of automatically balancing memory amongst multiple
 guests running on a system by taking some memory from the idle guests and giving it to
 the needy guests. Auto-Ballooning is the most essential component for memory
 overcommitment.
 <a
 id="x9-48001r52"></a>
 </p>

 <h4 class="subsectionHead"><span class="titlemark">4.2.1   </span> <a
 href="mainli2.html#QQ2-9-54" id="x9-490001">Hard Ballooning and Soft Ballooning</a></h4>
 <p>Depending upon the type of memory reclaimed ballooning can be classiﬁed into two types
 - hard ballooning and soft ballooning. Soft ballooning is the process of reclaiming memory
 which is free inside the guest while hard ballooning is the process of reclaiming memory
 which is used inside the guest.
 </p>
 <p>
 </p>
 <!--tex4ht:inline--><!--l. 134--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" ><mtable
 columnalign="left" class="align-star">
     <mtr><mtd
 columnalign="right" class="align-odd"></mtd>       <mtd
 class="align-even"><mi
 >S</mi><mi
 >o</mi><mi
 >f</mi><mi
 >t</mi><mi
 >L</mi><mi
 >o</mi><mi
 >w</mi><mi
 >e</mi><mi
 >r</mi><mi
 >B</mi><mi
 >o</mi><mi
 >u</mi><mi
 >n</mi><mi
 >d</mi> <mo
 class="MathClass-rel">=</mo> <mi
 >m</mi><mi
 >a</mi><mi
 >x</mi><mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >U</mi><mi
 >s</mi><mi
 >e</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi><mo
 class="MathClass-punc">,</mo><mi
 >G</mi><mi
 >u</mi><mi
 >e</mi><mi
 >s</mi><mi
 >t</mi><mi
 >R</mi><mi
 >e</mi><mi
 >s</mi><mi
 >e</mi><mi
 >r</mi><mi
 >v</mi><mi
 >e</mi><mi
 >d</mi></mrow><mo
 class="MathClass-close">)</mo></mrow><mspace width="2em"/></mtd>                 <mtd
 columnalign="right" class="align-label"></mtd>       <mtd
 class="align-label">
     <mspace width="2em"/></mtd></mtr><mtr><mtd
 columnalign="right" class="align-odd"></mtd>       <mtd
 class="align-even"><mi
 >S</mi><mi
 >o</mi><mi
 >f</mi><mi
 >t</mi><mi
 >I</mi><mi
 >d</mi><mi
 >l</mi><mi
 >e</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-rel">=</mo> <mi
 >m</mi><mi
 >a</mi><mi
 >x</mi><mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >A</mi><mi
 >l</mi><mi
 >l</mi><mi
 >o</mi><mi
 >c</mi><mi
 >a</mi><mi
 >t</mi><mi
 >e</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-bin">−</mo> <mi
 >S</mi><mi
 >o</mi><mi
 >f</mi><mi
 >t</mi><mi
 >L</mi><mi
 >o</mi><mi
 >w</mi><mi
 >e</mi><mi
 >r</mi><mi
 >B</mi><mi
 >o</mi><mi
 >u</mi><mi
 >n</mi><mi
 >d</mi><mo
 class="MathClass-punc">,</mo> <mn>0</mn></mrow><mo
 class="MathClass-close">)</mo></mrow><mspace width="2em"/></mtd>       <mtd
 columnalign="right" class="align-label"></mtd>       <mtd
 class="align-label">
     <mspace width="2em"/></mtd></mtr><mtr><mtd
 columnalign="right" class="align-odd"></mtd>       <mtd
 class="align-even"><mi
 >H</mi><mi
 >a</mi><mi
 >r</mi><mi
 >d</mi><mi
 >L</mi><mi
 >o</mi><mi
 >w</mi><mi
 >e</mi><mi
 >r</mi><mi
 >B</mi><mi
 >o</mi><mi
 >u</mi><mi
 >n</mi><mi
 >d</mi> <mo
 class="MathClass-rel">=</mo> <mi
 >m</mi><mi
 >a</mi><mi
 >x</mi><mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >L</mi><mi
 >o</mi><mi
 >a</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi><mo
 class="MathClass-punc">,</mo><mi
 >G</mi><mi
 >u</mi><mi
 >e</mi><mi
 >s</mi><mi
 >t</mi><mi
 >R</mi><mi
 >e</mi><mi
 >s</mi><mi
 >e</mi><mi
 >r</mi><mi
 >v</mi><mi
 >e</mi><mi
 >d</mi></mrow><mo
 class="MathClass-close">)</mo></mrow><mspace width="2em"/></mtd>                 <mtd
 columnalign="right" class="align-label"></mtd>       <mtd
 class="align-label">
     <mspace width="2em"/></mtd></mtr><mtr><mtd
 columnalign="right" class="align-odd"></mtd>       <mtd
 class="align-even"><mi
 >H</mi><mi
 >a</mi><mi
 >r</mi><mi
 >d</mi><mi
 >I</mi><mi
 >d</mi><mi
 >l</mi><mi
 >e</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-rel">=</mo> <mi
 >m</mi><mi
 >a</mi><mi
 >x</mi><mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >U</mi><mi
 >s</mi><mi
 >e</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-bin">−</mo> <mi
 >H</mi><mi
 >a</mi><mi
 >r</mi><mi
 >d</mi><mi
 >L</mi><mi
 >o</mi><mi
 >w</mi><mi
 >e</mi><mi
 >r</mi><mi
 >B</mi><mi
 >o</mi><mi
 >u</mi><mi
 >n</mi><mi
 >d</mi><mo
 class="MathClass-punc">,</mo> <mn>0</mn></mrow><mo
 class="MathClass-close">)</mo></mrow><mspace width="2em"/></mtd>          <mtd
 columnalign="right" class="align-label"></mtd>       <mtd
 class="align-label">
 <mspace width="2em"/></mtd></mtr></mtable></math>


 <p>   For soft ballooning, the guest is ballooned down to
 <!--l. 136--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >C</mi><mi
 >u</mi><mi
 >r</mi><mi
 >r</mi><mi
 >e</mi><mi
 >n</mi><mi
 >t</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-bin">−</mo> <mi
 >S</mi><mi
 >o</mi><mi
 >f</mi><mi
 >t</mi><mi
 >I</mi><mi
 >d</mi><mi
 >l</mi><mi
 >e</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi></mrow><mo
 class="MathClass-close">)</mo></mrow></math>, which
 will reclaim the SoftIdleMemory. To reclaim HardIdleMemory, the guest has to be ballooned
 down to <!--l. 136--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >U</mi><mi
 >s</mi><mi
 >e</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-bin">−</mo> <mi
 >H</mi><mi
 >a</mi><mi
 >r</mi><mi
 >d</mi><mi
 >I</mi><mi
 >d</mi><mi
 >l</mi><mi
 >e</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi></mrow><mo
 class="MathClass-close">)</mo></mrow></math>.
 Thus, ballooning down a guest from current memory to
 <!--l. 136--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >C</mi><mi
 >u</mi><mi
 >r</mi><mi
 >r</mi><mi
 >e</mi><mi
 >n</mi><mi
 >t</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-bin">−</mo> <mi
 >S</mi><mi
 >o</mi><mi
 >f</mi><mi
 >t</mi><mi
 >I</mi><mi
 >d</mi><mi
 >l</mi><mi
 >e</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi></mrow><mo
 class="MathClass-close">)</mo></mrow></math>
 will reclaim SoftIdleMemory. After this, ballooning down from
 <!--l. 136--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >C</mi><mi
 >u</mi><mi
 >r</mi><mi
 >r</mi><mi
 >e</mi><mi
 >n</mi><mi
 >t</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-bin">−</mo> <mi
 >S</mi><mi
 >o</mi><mi
 >f</mi><mi
 >t</mi><mi
 >I</mi><mi
 >d</mi><mi
 >l</mi><mi
 >e</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi></mrow><mo
 class="MathClass-close">)</mo></mrow></math> to
 <!--l. 136--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi
 >U</mi><mi
 >s</mi><mi
 >e</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi></math>
 will reclaim no memory. Then, ballooning down from used memory to
 <!--l. 136--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >U</mi><mi
 >s</mi><mi
 >e</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-bin">−</mo> <mi
 >H</mi><mi
 >a</mi><mi
 >r</mi><mi
 >d</mi><mi
 >I</mi><mi
 >d</mi><mi
 >l</mi><mi
 >e</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi></mrow><mo
 class="MathClass-close">)</mo></mrow></math>
 will reclaim the HardIdleMemory. Figure <a
 href="#x9-49001r2">4.2<!--tex4ht:ref: fig:mem2 --></a> shows the two diﬀerent types of
 ballooning.
 </p>
 <hr class="figure" /><div class="figure"
 >


 <a
 id="x9-49001r2"></a>



 <p><img
 src="/images/thesis/mem2.png" alt="PIC"  
 />
 <a
 id="x9-49002"></a>
 <br /> </p>
 <div class="caption"
 ><span class="id">Figure 4.2: </span><span  
 class="content">Soft and hard ballooning</span></div><!--tex4ht:label?: x9-49001r4 -->


 </div><hr class="endfigure" />
 <a
 id="x9-49003r54"></a>
 <h4 class="subsectionHead"><span class="titlemark">4.2.2   </span> <a
 href="mainli2.html#QQ2-9-55" id="x9-500002">Auto-Ballooning Algorithm</a></h4>
 <p>The autoballooning algorithm ﬁrst identiﬁes the guests with idle memory and the guests
 which need more memory. We have already described the method of calculating
 the idle memory earlier. We identify needy guests as the ones whose average
 load memory is greater than a certain threshold. We have kept the threshold to
 <!--l. 146--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mn>9</mn><mn>0</mn><mi
 >%</mi></math> of
 the current memory of the guest. Needy guests are ballooned up in intervals of
 <!--l. 146--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mn>1</mn><mn>0</mn><mi
 >%</mi></math> of
 their current memory. Total needed memory is the sum total of the memory needed
 by all the needy guests(which is 10% of the current memory for each needy
 guest).
 </p>
 <div class="math-display"><!--l. 147--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" ><mrow
 >
 <mi
 >i</mi><mi
 >s</mi><mi
 >N</mi><mi
 >e</mi><mi
 >e</mi><mi
 >d</mi><mi
 >y</mi><mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >g</mi><mi
 >u</mi><mi
 >e</mi><mi
 >s</mi><mi
 >t</mi></mrow><mo
 class="MathClass-close">)</mo></mrow> <mo
 class="MathClass-rel">=</mo>  <mfenced separators=""
 open="{"  close="" ><mrow> <mtable  align="axis" style=""  
 equalrows="false" columnlines="none" equalcolumns="false" class="array"><mtr><mtd
 class="array"  columnalign="left"><mi
 >T</mi><mi
 >r</mi><mi
 >u</mi><mi
 >e</mi><mspace width="1em" class="quad"/></mtd><mtd
 class="array"  columnalign="left"><mstyle
 class="text"><mtext  ></mtext><mstyle
 class="math"><mi
 >L</mi><mi
 >o</mi><mi
 >a</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-rel">≥</mo> <mn>0</mn><mo
 class="MathClass-punc">.</mo><mn>9</mn> <mo
 class="MathClass-bin">∗</mo> <mi
 >C</mi><mi
 >u</mi><mi
 >r</mi><mi
 >r</mi><mi
 >e</mi><mi
 >n</mi><mi
 >t</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi></mstyle><mtext  ></mtext></mstyle></mtd></mtr><mtr><mtd
 class="array"  columnalign="left"><mi
 >F</mi> <mi
 >a</mi><mi
 >l</mi><mi
 >s</mi><mi
 >e</mi><mspace width="1em" class="quad"/></mtd> <mtd
 class="array"  columnalign="left"><mstyle
 class="text"><mtext  >otherwise</mtext></mstyle></mtd></mtr><mtr><mtd
 class="array"  columnalign="left"> <mspace width="1em" class="quad"/> </mtd></mtr> <!--@{}l@{\quad }l@{}--></mtable>                                                 </mrow></mfenced>
 </mrow></math></div>
 <p>


 <!--tex4ht:inline--></p>
 <!--l. 154--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" >
                  <mi
 >N</mi><mi
 >e</mi><mi
 >e</mi><mi
 >d</mi><mi
 >e</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-rel">=</mo> <mn>0</mn><mo
 class="MathClass-punc">.</mo><mn>1</mn> <mo
 class="MathClass-bin">∗</mo> <mi
 >C</mi><mi
 >u</mi><mi
 >r</mi><mi
 >r</mi><mi
 >e</mi><mi
 >n</mi><mi
 >t</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi>
 </math>
 <p>
 </p>
 <p>   The <em><span
 class="cmti-12">host unused memory</span></em> is the amount of memory on the host which is neither used
 by/allocated to any guest nor is being used by the host, and hence, can be given
 away to any host by ballooning it up. Ballooning up a guest reduces the host
 unused memory, while ballooning a guest down decreases it. It is calculated as
 follows.
 </p>
 <!--tex4ht:inline--><!--l. 161--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" ><mtable
 columnalign="left" class="align-star">
 <mtr><mtd
 columnalign="right" class="align-odd"><mi
 >H</mi><mi
 >o</mi><mi
 >s</mi><mi
 >t</mi><mi
 >U</mi><mi
 >n</mi><mi
 >u</mi><mi
 >s</mi><mi
 >e</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-rel">=</mo> <mi
 >A</mi><mi
 >v</mi><mi
 >a</mi><mi
 >i</mi><mi
 >l</mi><mi
 >a</mi><mi
 >b</mi><mi
 >l</mi><mi
 >e</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-bin">−</mo> <mi
 >H</mi><mi
 >y</mi><mi
 >p</mi><mi
 >e</mi><mi
 >r</mi><mi
 >v</mi><mi
 >i</mi><mi
 >s</mi><mi
 >o</mi><mi
 >r</mi><mi
 >E</mi><mi
 >x</mi><mi
 >t</mi><mi
 >r</mi><mi
 >a</mi></mtd><mtd
 class="align-even"><mspace width="2em"/></mtd>                                     <mtd
 columnalign="right" class="align-label">
 </mtd></mtr><mtr><mtd
 columnalign="right" class="align-odd"></mtd>                                                               <mtd
 class="align-even"><mi
 >o</mi><mi
 >r</mi><mspace width="2em"/></mtd>                                   <mtd
 columnalign="right" class="align-label"></mtd><mtd
 class="align-label">
 <mspace width="2em"/></mtd></mtr><mtr><mtd
 columnalign="right" class="align-odd"><mi
 >H</mi><mi
 >o</mi><mi
 >s</mi><mi
 >t</mi><mi
 >U</mi><mi
 >n</mi><mi
 >u</mi><mi
 >s</mi><mi
 >e</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-rel">=</mo> <mi
 >T</mi><mi
 >o</mi><mi
 >t</mi><mi
 >a</mi><mi
 >l</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi></mtd>                        <mtd
 class="align-even"> <mo
 class="MathClass-bin">−</mo> <mi
 >L</mi><mi
 >o</mi><mi
 >a</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-bin">−</mo> <mi
 >T</mi><mi
 >o</mi><mi
 >t</mi><mi
 >a</mi><mi
 >l</mi><mi
 >I</mi><mi
 >d</mi><mi
 >l</mi><mi
 >e</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi><mspace width="2em"/></mtd><mtd
 columnalign="right" class="align-label"></mtd><mtd
 class="align-label">
 <mspace width="2em"/></mtd></mtr></mtable></math>
 <p>Auto-Ballooning can be triggered in two cases -
 <!--l. 162--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >H</mi><mi
 >o</mi><mi
 >s</mi><mi
 >t</mi><mi
 >U</mi><mi
 >n</mi><mi
 >u</mi><mi
 >s</mi><mi
 >e</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-rel">&#x003C;</mo> <mn>0</mn><mo
 class="MathClass-punc">.</mo><mn>1</mn> <mo
 class="MathClass-bin">∗</mo> <mi
 >T</mi><mi
 >o</mi><mi
 >t</mi><mi
 >a</mi><mi
 >l</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi></mrow><mo
 class="MathClass-close">)</mo></mrow></math> or
 <!--l. 162--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >T</mi><mi
 >o</mi><mi
 >t</mi><mi
 >a</mi><mi
 >l</mi><mi
 >N</mi><mi
 >e</mi><mi
 >e</mi><mi
 >d</mi><mi
 >e</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-rel">&#x003E;</mo> <mn>0</mn></mrow><mo
 class="MathClass-close">)</mo></mrow></math>. The ﬁrst
 case is important to keep the amount of swapped memory on the host low. To handle the ﬁrst
 case, ﬁrst the soft idle memory and then the hard idle memory form the guests is ballooned out
 till the <!--l. 162--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mrow ><mo
 class="MathClass-open">(</mo><mrow><mi
 >H</mi><mi
 >o</mi><mi
 >s</mi><mi
 >t</mi><mi
 >U</mi><mi
 >n</mi><mi
 >u</mi><mi
 >s</mi><mi
 >e</mi><mi
 >d</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi> <mo
 class="MathClass-rel">&#x003C;</mo> <mn>0</mn><mo
 class="MathClass-punc">.</mo><mn>2</mn> <mo
 class="MathClass-bin">∗</mo> <mi
 >T</mi><mi
 >o</mi><mi
 >t</mi><mi
 >a</mi><mi
 >l</mi><mi
 >M</mi><mi
 >e</mi><mi
 >m</mi><mi
 >o</mi><mi
 >r</mi><mi
 >y</mi></mrow><mo
 class="MathClass-close">)</mo></mrow></math>. If


 the idle memory is exhausted before this 20% memory becomes unused, swapping is
 inevitable and the job of resolving it is left to the migration service.
 </p>
 <p>   In the second case, either there is a needy guest or memory needs to be reclaimed for a
 guest which would be migrated to the machine. Both the situations are similar except
 that when memory is reclaimed for an incoming guest, there is no needy guest to be
 ballooned up. Depending on the needed memory and idle memory, three situations can
 arise.
    </p>
 <dl class="enumerate"><dt class="enumerate">
 1. </dt><dd
 class="enumerate"><!--l. 167--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi mathvariant="bold">TotalNeededMemory ≤ TotalSoftIdleMemory.</mi></math>
    The requirements of each needy guest can be satisﬁed by just soft-ballooning.
    So, the idle guests are ballooned down to reclaim the needed memory and
    then, the needy guests are ballooned up.
    </dd><dt class="enumerate">
 2. </dt><dd
 class="enumerate"><!--l. 168--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi mathvariant="bold">TotalSoftIdleMemory &#x003C; TotalNeededMemory ≤ TotalHardIdleMemory.</mi></math>
    First, all the soft idle memory is ballooned out. Then, the rest of the needed
    memory is divided among the guests with hard idle memory in proportion of
    their hard idle memory. So, memory reclaimed by hard ballooning for a guest
    is given by
    <!--tex4ht:inline--><!--l. 169--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" >
 <mi>NeedAfterSoftBallooning = TotalNeededMemory − TotalSoftIdleMemory</mi>
 </math>
    <p>


    <!--tex4ht:inline--></p>
 <!--l. 170--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" >
      <mi>HardReclaim =</mi> <mfrac><mrow
 ><mi>GuestHardIdleMem ∗ NeedAfterSoftBallooning</mi></mrow>
             <mrow
 ><mi>TotalHardIdleMemory</mi></mrow></mfrac>
 </math>
    <p> Here, <!--l. 171--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi>GuestHardIdleMem</mi></math>
    is the hard idle memory for that particular guest, while <!--l. 171--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi>TotalHardIdleMem</mi></math>
    is the total hard idle memory. The reason hard ballooning is treated diﬀerently
    from soft ballooning is because soft ballooning is not supposed to have any
    aﬀect on the performance of the guest as it takes away only the free pages. On
    the other hand, hard ballooning also reclaims some of the page caches, which
    might have some eﬀect on the performance of the guest.
    </p>
 </dd><dt class="enumerate">
 3. </dt><dd
 class="enumerate"><!--l. 172--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="inline" ><mi mathvariant="bold">TotalNeededMemory &#x003E; TotalHardIdleMemory.</mi></math>
    This case implies that there is a hotspot on the host. Since the demands of
    all the guests cannot be satisﬁed, the memory is given to them or reclaimed
    from them based on their entitlement. The <em><span
 class="cmti-12">entitled memory</span></em> of each guest is
    calculated as
    <!--tex4ht:inline--><!--l. 173--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" >
 <mi>EntitledMemory = GuestMaximumMemory ∗ MemoryOvercommitmentRatio</mi>
 </math>
    <p> After this, the idle memory and needed memory calculation is done again.


    <!--tex4ht:inline--></p>
 <!--l. 175--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" >
            <mi>IdleMemory = max(CurrentMemory − EntitledMemory, 0)</mi>
 </math>
    <p>
 </p>
 <div class="math-display"><!--l. 176--><math
 xmlns="http://www.w3.org/1998/Math/MathML"  
 display="block" ><mrow
 >
 <mi>NeededMemory =  </mi><mfenced separators=""
 open="{"  close="" ><mrow> <mtable  align="axis" style=""  
 equalrows="false" columnlines="none" equalcolumns="false" class="array"><mtr><mtd
 class="array"  columnalign="left"><mi>0</mi><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mstyle
 class="text"><mtext  >if </mtext><mstyle
 class="math"><mi>!isNeedy(guest)</mi></mstyle><mtext  ></mtext></mstyle>                                     <mspace width="1em" class="quad"/></mtd>
 </mtr><mtr><mtd
 class="array"  columnalign="left"><mi>0</mi><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mi>CurrentMemory &#x003E; EntitledMemory          </mi><mspace width="1em" class="quad"/></mtd>
 </mtr><mtr><mtd
 class="array"  columnalign="left"><mi>min(0.1 ∗ CurrentMemory,                      </mi><mspace width="1em" class="quad"/></mtd>
 </mtr><mtr><mtd
 class="array"  columnalign="left"><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mi>CurrentMemory − EntitledMemory)</mi><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mspace width="1em" class="nbsp" /><mstyle
 class="text"><mtext  >otherwise</mtext></mstyle><mspace width="1em" class="quad"/></mtd></mtr><mtr><mtd
 class="array"  columnalign="left"><mspace width="1em" class="quad"/></mtd></mtr><!--@{}l@{\quad }l@{}--></mtable>                                              </mrow></mfenced>
 </mrow></math></div>
    <p> Then, the idle memory is ballooned out of the guests and the needed memory
    is provided to the needy guests.
 <a
 id="Q1-9-56"></a>
    </p>

    <h4 class="likesubsectionHead"><a
 href="#x9-510003" id="x9-510003">Summary</a></h4>
    <p>In this chapter, we looked at the diﬀerent techniques we use to monitor various
    metrics for hosts and guests. These metrics are used to trigger hotspot and
    make  the  ballooning  service  active.  We  also  looked  at  the  CUSUM  based
    algorithm used to ﬁlter out unecessary spikes in the resource usage proﬁles of
    the host. In the end, we also discussed our technique of autoballooning.</p>
 </dd></dl>


 <!--l. 1--><div class="crosslinks"><p class="noindent">[<a
 href="mainch5.html" >next</a>] [<a
 href="mainch3.html" >prev</a>] [<a
 href="mainch3.html#tailmainch3.html" >prev-tail</a>] [<a
 href="mainch4.html" >front</a>] [<a
 href="main.html#mainch4.html" >up</a>] </p></div>
 <p>   <a
 id="tailmainch4.html"></a>             </p>

 {{< /raw >}}
