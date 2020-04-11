---


---

<h1 id="offloading-to-multiple-devices-using-sycl">Offloading to multiple devices using SYCL</h1>
<p>In this tutorial I will show how to offload to multiple devices using SYCL. In particular, I will demonstrate how to run a SYCL kernel on CPU and GPU (Intel CPU with integrated GPU) simultaneously.</p>
<h2 id="requirements">Requirements</h2>
<p>The basic requirement is to have 2 OpenCL platforms. This could be a CPU + discrete GPU or, in my case, Intel CPU with integrated graphics. <a href="https://github.com/intel/compute-runtime">Here’s how to set them up</a></p>
<h2 id="basics">Basics</h2>
<p>In order to take advantage of two devices, you will have to share work between them.<br>
It is possible to simply add another queue and submit a parallel_for with nd_range where each nd_range operates on different sections of the same array using range and offset options. This will not give you any speedup, however. At least not in this code. Because the buffers (and accessors) you created span the entire array, SYCL runtime will not be able to overlap the execution of two queues. It will write only the memory you touched, however, producing the correct result. ###NEEDS FACTCHECK###<br>
<code>git checkout sycl2devBase</code></p>
<h3 id="duplicating-buffers">Duplicating Buffers</h3>
<p>In this example we have just the acceleration buffers that are being written to. ###in the original code implementation where that write is actually unnecessary### Overlapping reads is trivial but writes is not. In order to overlap computation we need to allow for overlapping of these writes.<br>
One thing to try is duplicating buffers. Since we are executing on different parts of host memory and I previously stated that only the touched memory will be written out this should do the trick, right?<br>
####still not clear, <a href="https://jira.devtools.intel.com/browse/OTFIP-573">https://jira.devtools.intel.com/browse/OTFIP-573</a>###</p>
<h3 id="split-buffers">Split buffers</h3>
<p>Seems like we need these original buffers span only the memory region the partial computation will touch.<br>
<code>std::vector&lt;buffer&lt;real_type, 1&gt;&gt; particles_acc_x_d;</code><br>
<code>particles_acc_x_d.push_back( 	buffer&lt;real_type, 1&gt;(particles-&gt;acc_x + offsets[qi], range&lt;1&gt;(shares[qi])));</code><br>
Doesn’t speed up, and results wrong because offsets<br>
Don’t need to offset your nd_range if you offest your buffer<br>
Problem here:<br>
<code>dx = particles_pos_x[j] - particles_pos_x[i];</code> indexed by i</p>
<p>One major issue here is that your buffers are offset<br>
when you index by i from nd_range, it indexes into correct device side<br>
but wrong host side<br>
I tried using offsets from host side but they were not copied in!</p>
<ul>
<li>Separate queue for each device</li>
<li>One set of buffers
<ul>
<li>A set of sub-buffers for each device</li>
</ul>
</li>
<li></li>
</ul>
<h3 id="sycl-sub-buffers">SYCL sub-buffers</h3>
<p>We need to use sub-buffers<br>
once you submit a kernel with  accessors to host-side SYCL buffer, that entire memory space is</p>
<p><img src="https://picasaweb.google.com/100509739067198757048/6734780794550257649#6734780797936452162" alt="SYCL sub-buffer submits"></p>
<p>Why would I create sub-buffers instead of separate buffers?</p>
<h2 id="todo">TODO</h2>
<p>Buffer offesets</p>

