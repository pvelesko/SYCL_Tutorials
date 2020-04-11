---


---

<h1 id="sycl-usm---unified-shared-memory">SYCL USM - Unified Shared Memory</h1>
<p>Unified shared memory allows for seamless data transfers between SYCL devices and host without creating buffers.<br>
This feature is available from <a href="https://github.com/intel/llvm">Intel LLVM fork</a> for SYCL using the latest  <a href="https://github.com/intel/compute-runtime">NEO OpenCL runtime</a>. It is enabled by <code>cl_intel_unified_shared_memory_preview</code>  OpenCL extension.</p>
<p>For any additional information please refer to <a href="https://github.com/intel/llvm/blob/a6d7e12b7c2b5487298f5dcc2a848d48adb9b09b/sycl/doc/extensions/USM/USM.adoc">SYCL Proposal for Unified Shared Memory</a></p>
<h2 id="using-usm">Using USM</h2>
<h3 id="shared-allocation">Shared Allocation</h3>
<p>USM memory is managed through allocation functions provided in sycl.hpp<br>
Easiest way to get started is to use<br>
<code>int* usm_int = static_cast&lt;int*&gt;(malloc_shared(SIZE, device, context));</code><br>
This will return a pointer that you can then use in your SYCL kernel launches. No need for buffers.</p>
<h3 id="shared-deallocation">Shared Deallocation</h3>
<p>Don’t forget to free the pointers to avoid memory leaks:<br>
<code>free(usm_int, context);</code></p>
<h3 id="query-the-pointer">Query the pointer</h3>
<p>TODO: get_pointer_info… OTFIP-572</p>
<h3 id="indexing">Indexing</h3>
<p>Unlike buffers (which can be indexed using <code>id</code>, USM pointers can be indexed using basic integers.<br>
For a one-dimensional <code>id</code> get the index by accessing the 0th element:</p>
<pre><code>auto idx = item.get_global_id();
int i = idx[0]; 
usm_int[i] = 999;
</code></pre>
<h2 id="synchronization">Synchronization</h2>
<p>Since kernel launches are non-blocking it is important to check that the computation is finished before launching. You can do so by capturing the event returned by <code>submit()</code>and then calling the <code>event.wait()</code> method:</p>
<pre><code>auto e = q.submit([&amp;] (handler&amp; cgh) {.......
		usm_int[1] = 99;
});
e.wait();
std::cout &lt;&lt; usm_int[1];
</code></pre>
<h1 id="section"></h1>
<h3 id="copying-from-explicitly-allocated-device-memory">Copying from explicitly allocated device memory</h3>
<p>Say you used malloc_device<br>
this is not accesible from host natively. How do you read it?</p>
<h3 id="deallocation">Deallocation</h3>
<p>free( pointer, context)</p>
<h2 id="full-example-code">Full Example Code</h2>
<pre><code>#include &lt;CL/sycl.hpp&gt;
#include &lt;iostream&gt;
using namespace cl::sycl;
#define SIZE 3
#define TYPE int

int main(int, char**) {
  queue q(gpu_selector{});
  device dev = q.get_device();
  context ctx = q.get_context();
  std::cout &lt;&lt; "Running on "
            &lt;&lt; dev.get_info&lt;info::device::name&gt;()
            &lt;&lt; std::endl;

  TYPE* usm_int = static_cast&lt;TYPE*&gt;(malloc_shared(SIZE*sizeof(TYPE), dev, ctx));
  if (usm_int == NULL) std::cout &lt;&lt; "Failed alloc" &lt;&lt; std::endl;
  for (int i = 0; i &lt; SIZE; i++)
  {
    usm_int[i] = 0;
    std::cout &lt;&lt; usm_int[i] &lt;&lt; std::endl;
  }

  //{ // buffer scope
    auto e = q.submit([&amp;] (handler&amp; cgh)
      { // q scope
        cgh.single_task&lt;class oneplusone&gt;([=]
        {
          usm_int[3] = 3;
        } ); // end task scope
      } ); // end q scope
  //} // end buffer scope

  e.wait();
  for (int i = 0; i &lt; SIZE; i++)
  {
    std::cout &lt;&lt; usm_int[i] &lt;&lt; std::endl;
  }

  return 0;
}

</code></pre>
<h2 id="usm-device-pointers">USM Device Pointers</h2>
<p>You can also allocate data directly on the device and use explicit memory copy routines to manage data transfer.</p>
<h3 id="allocation">Allocation</h3>
<p>USM memory is managed through allocation functions provided in sycl.hpp<br>
Easiest way to get started is to use<br>
<code>int* usm_int = static_cast&lt;int*&gt;(malloc_device(SIZE, device, context));</code><br>
This will return a pointer that you can then use in your SYCL kernel launches. No need for buffers.</p>
<h3 id="deallocation-1">Deallocation</h3>
<p>Don’t forget to free the pointers to avoid memory leaks:<br>
<code>free(usm_int, context);</code></p>
<h3 id="usage">Usage</h3>
<p>Data pointed by these allocations lives on the device and cannot be accessed on the host directly. To do so you will need to use explicit memory copies:<br>
<code>queue.memcpy()</code> or <code>handler.memcpy()</code> routines</p>
<h3 id="queue-copy">Queue Copy</h3>
<p>As soon as you have a queue and host data you want to transfer you can do so with <code>queue.memcpy()</code><br>
<code>auto e = q.memcpy(usm_dev_ptr, host_ptr, n*sizeof(TYPE); e.wait();</code><br>
This will create a blocking write to device - guaranteeing the data transfer is complete before proceeding.</p>
<h3 id="handler-copy">Handler Copy</h3>
<p>another way of doing the data transfer is to use the queue generated handler:<br>
<code>cgh.memcpy(usm_dev_ptr, host_ptr, n*sizeof(TYPE);</code><br>
Copy the data in, execute the task using <code>cgh.parallel_for</code>or similar, and copy data back out.<br>
TODO: This doesn’t work - OTFIP-580</p>

