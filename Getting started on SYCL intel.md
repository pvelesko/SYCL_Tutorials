---


---

<h1 id="getting-started">Getting started</h1>
<pre><code>#include &lt;CL/sycl.hpp&gt;
#include &lt;iostream&gt;
using namespace cl::sycl;
int main(int argc, char** argv)
{
  device dev = device(default_selector{});
  queue q = queue(dev);
  std::cout &lt;&lt; dev.get_info&lt;info::device::name&gt;() &lt;&lt; std::endl;
  return 0;
}
</code></pre>
<p><code>device dev = device(default_selector{});</code><br>
Creates a default SYCL device. This could be an OpenCL CPU device, OpenCL GPU device or even a SYCL Host device. You should be careful when using <code>default_selector{}</code> as it may create a device of different type than what you’re expecting.<br>
<code>queue q = queue(dev);</code><br>
Will create a SYCL queue for a particular device. SYCL queues are out-of-order.</p>
<p><code>dev.get_info&lt;info::device::name&gt;()</code><br>
<code>Device</code> class has a templated method <code>get_info</code> that returns different information based on the template parameter. Description of this can be found on page 69 of <a href="https://www.khronos.org/files/sycl/sycl-12-reference-card.pdf">SYCL spec</a></p>
<p><img src="https://lh3.googleusercontent.com/WsxEYuVEnqSq2FEZoIcEBW924yr3ixbKfhEv8e7-DAdoHVZI0nFb2W1bZT1WXLRJhKqCi3GgUx0wVA" alt="enter image description here"></p>
<h2 id="compilation">Compilation</h2>
<p>Let’s take a look at the makefile:</p>
<pre><code>all:
        dpcpp -std=c++14 -fsycl ./main.cpp
</code></pre>
<p>-fsycl enables SYCL and is on by default<br>
-lOpenCL links OpenCL runtime</p>
<pre><code>$ make
$ ./a.out
$ Intel(R) Core(TM) i7-6770HQ CPU @ 2.60GHz
</code></pre>

