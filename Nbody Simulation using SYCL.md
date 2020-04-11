---


---

<h1 id="writing-your-first-second-sycl-program">Writing your <s>first</s> second SYCL program</h1>
<p>In this tutorial I will show you how to port a simple nbody simulation to CPU/GPU/FPGA using <a href="https://www.khronos.org/sycl/">SYCL</a>. If you haven’t yet written your first SYCL program, I suggest you go follow <a href="https://www.codeplay.com/portal/sycl-tutorial-1-the-vector-addition">this example</a> and write a simple vector add.<br>
If you are ever unsure of syntax, please refer to the <a href="https://www.khronos.org/files/sycl/sycl-12-reference-card.pdf">SYCL spec</a> document.</p>
<h1 id="nbody-example">Nbody Example</h1>
<p>This example is adopted from <a href="https://github.com/fbaru-dev/nbody-demo">Fabio Baruffa’s Github</a> where he explains what the code in detail. In short, we’re calculating forces between all particle pairs to get accelerations, then update velocities and positions, repeat for specified number of time steps while reporting energies.</p>
<div class="mermaid"><svg xmlns="http://www.w3.org/2000/svg" id="mermaid-svg-dLDg5xACBKKTZulj" width="100%" style="max-width: 858.34375px;" viewBox="0 0 858.34375 76.625"><g transform="translate(-12, -12)"><g class="output"><g class="clusters"></g><g class="edgePaths"><g class="edgePath" style="opacity: 1;"><path class="path" d="M112.609375,46.25L137.609375,46.25L162.609375,46.25" marker-end="url(#arrowhead43)" style="fill:none"></path><defs><marker id="arrowhead43" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker></defs></g><g class="edgePath" style="opacity: 1;"><path class="path" d="M292.875,38.625977547931384L382.5859375,28.125L472.296875,37.95726450302343" marker-end="url(#arrowhead44)" style="fill:none"></path><defs><marker id="arrowhead44" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker></defs></g><g class="edgePath" style="opacity: 1;"><path class="path" d="M472.296875,54.54273549697657L382.5859375,64.375L292.875,53.874022452068616" marker-end="url(#arrowhead45)" style="fill:none"></path><defs><marker id="arrowhead45" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker></defs></g><g class="edgePath" style="opacity: 1;"><path class="path" d="M623.625,46.25L648.625,46.25L673.625,46.25" marker-end="url(#arrowhead46)" style="fill:none"></path><defs><marker id="arrowhead46" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker></defs></g></g><g class="edgeLabels"><g class="edgeLabel" transform="" style="opacity: 1;"><g transform="translate(0,0)" class="label"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel" transform="" style="opacity: 1;"><g transform="translate(0,0)" class="label"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(382.5859375,64.375)" style="opacity: 1;"><g transform="translate(-64.7109375,-16.25)" class="label"><foreignObject width="129.43359375" height="32.5"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">num timesteps</span></div></foreignObject></g></g><g class="edgeLabel" transform="" style="opacity: 1;"><g transform="translate(0,0)" class="label"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g></g><g class="nodes"><g class="node" id="A" transform="translate(66.3046875,46.25)" style="opacity: 1;"><rect rx="0" ry="0" x="-46.3046875" y="-26.25" width="92.609375" height="52.5"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-36.3046875,-16.25)"><foreignObject width="72.6171875" height="32.5"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Initialize</div></foreignObject></g></g></g><g class="node" id="B" transform="translate(227.7421875,46.25)" style="opacity: 1;"><rect rx="0" ry="0" x="-65.1328125" y="-26.25" width="130.265625" height="52.5"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-55.1328125,-16.25)"><foreignObject width="110.2734375" height="32.5"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">update accel</div></foreignObject></g></g></g><g class="node" id="C" transform="translate(547.9609375,46.25)" style="opacity: 1;"><rect rx="0" ry="0" x="-75.6640625" y="-26.25" width="151.328125" height="52.5"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-65.6640625,-16.25)"><foreignObject width="131.328125" height="32.5"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">update vel/pos</div></foreignObject></g></g></g><g class="node" id="D" transform="translate(767.984375,46.25)" style="opacity: 1;"><rect rx="0" ry="0" x="-94.359375" y="-26.25" width="188.71875" height="52.5"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-84.359375,-16.25)"><foreignObject width="168.73046875" height="32.5"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">report energy/stats</div></foreignObject></g></g></g></g></g></g></svg></div>
<p>Because we’re using all particle pairs resulting in a double nested loop with inner most loop traversing the all the particles we will need to do a global update before starting each time step.</p>
<pre><code>One Device						|CPU+GPU
-------------------------------------------------------------------------------
for each timestep				|for each timestep
	for each particle			|	for CPU/GPU share of particles 
		for each particle		|		for each particle
			compute()			|			compute()
</code></pre>
<h1 id="sycl-steps">SYCL steps</h1>
<p>In order to convert this example to SYCL we’ll need:</p>
<ul>
<li>Compiler</li>
<li>Context<br>
* Device selector - CPU/GPU/Host/Default</li>
<li>Buffer</li>
<li>Queue submit<br>
* Accessors for data</li>
</ul>
<h3 id="compiler-and-runtime">Compiler and Runtime</h3>
<p>We will be using <a href="https://github.com/intel/llvm/tree/sycl">Intel LLVM SYCL compiler</a><br>
along with <a href="https://github.com/intel/compute-runtime">Intel NEO GPU runtime</a> for running on the iGPU that’s available on many Intel CPUs.<br>
Check out <a href="https://software.intel.com/en-us/articles/opencl-drivers">Intel OpenCL drivers</a> for instructions on how to enable OpenCL on Intel CPUs and iGPUs.</p>
<blockquote>
<p>A unified (single OpenCL platform containing a CPU and a GPU OpenCL device) OpenCL runtime is under development</p>
</blockquote>
<h3 id="device-setup-and-compile-test">Device setup and compile test</h3>
<p>Setting up the proper queues is quite expensive so create them higher up the callstack than your compute loop. In this example we will use a GPU selector since we want to target iGPU of the Intel Skylake processor.<br>
We will be using the SYCL namespace <code>using namespace cl::sycl</code> and <code>auto</code> feature of C++11<br>
You don’t have to but if you don’t the lines are going to be too verbose and long…<br>
e.g:<br>
<code>cl::sycl::accessoror&lt;cl::sycl::access::read&gt; a_acc = a_sycl.get_access&lt;cl::sycl::access::mode::read&gt;(cgh);</code><br>
vs<br>
<code>auto a_acc = a_sycl.get_access&lt;access::mode::read&gt;(cgh);</code></p>
<h2 id="convert-the-compute-loop-to-sycl">Convert the compute loop to SYCL</h2>
<h3 id="create-buffers">1. Create buffers</h3>
<p>We will follow simple naming convention for naming our buffers:<br>
<code>particles-&gt;pos_x</code> -&gt; <code>particles_pos_x_d</code> where <code>_d</code> denotes that data lives on the device.</p>
<p>Our buffer allocation section should look like this:</p>
<pre><code>auto particles_pos_x_d = buffer&lt;float, 1&gt;(particles-&gt;pos_x, N);
auto particles_pos_y_d = buffer&lt;float, 1&gt;(particles-&gt;pos_y, N);
auto particles_pos_z_d = buffer&lt;float, 1&gt;(particles-&gt;pos_z, N);
</code></pre>
<h5 id="where-to-place-your-buffer-allocations">Where to place your buffer allocations?</h5>
<p>In SYCL data movement is timed by reference counts. Every time you attach a buffer to a region of memory a counter is incremented and and decremented upon buffer deallocation. Once a buffer goes out of scope (gets deallocated) you can be sure that the host memory was updated with the contents of the said buffer. While this is not the only way of synchronizing memory, it is the simplest. For this first SYCL port we will use this mechanism and place our buffer allocation at the beginning of each time step.<br>
We use an extra set of curly brackets inside the timestepping loop to indicate the proper scope.</p>
<pre><code>for each time step
	{
	create SYCL buffers..
	for each CPU/GPU share of particles
		for each particle
			compute()
	}
</code></pre>
<pre><code>for (int s=1; s&lt;=get_nsteps(); ++s)
{
	{ // buffer scope
		auto particles_pos_x_d = buffer&lt;real_type, 1&gt;(particles-&gt;pos_x, range&lt;1&gt;(n));
		...
		for (i = 0; i &lt; n; i++) // update acceleration
		{
			for (j = 0; j &lt; n; j++)
			{
				dx = particles-&gt;pos_x[j] - particles-&gt;pos_x[i]; //1flop
				...
			}
		...
	} // end buffer scope
}
</code></pre>
<h3 id="make-a-sycl-kernel---submit-to-queue">2. Make a SYCL kernel - Submit to queue</h3>
<pre><code>for (int s=1; s&lt;=get_nsteps(); ++s)
{
	{ // buffer scope
		auto particles_pos_x_d = buffer&lt;real_type, 1&gt;(particles-&gt;pos_x, range&lt;1&gt;(n));
		...
	q.submit([&amp;] (handler&amp; cgh) {
		for (i = 0; i &lt; n; i++)// update acceleration
		{
			...
			for (j = 0; j &lt; n; j++)
			{
				//dx = particles-&gt;pos_x[j] - particles-&gt;pos_x[i]; //1flop
				dx = particles_pos_x[j] - particles_pos_x[i]; //1flop
				...
			}
		...
	} // end buffer scope
}
</code></pre>
<h3 id="create-accessors">3. Create Accessors</h3>
<p>Similarly, we will follow the pattern of naming our accessors as follows:<br>
<code>particles_pos_x_d</code> -&gt; <code>particles_pos_x</code><br>
This naming convention allows us to use  a simple regex expression for converting all the accesses from host pointers to SYCL accessors so that our compute then looks like this:</p>
<pre><code>for (int s=1; s&lt;=get_nsteps(); ++s)
{
	for (i = 0; i &lt; n; i++)// update acceleration
	{
	&lt;...&gt;
	for (j = 0; j &lt; n; j++)
	{
		&lt;...&gt;
		//dx = particles-&gt;pos_x[j] - particles-&gt;pos_x[i]; //1flop
		//dy = particles-&gt;pos_y[j] - particles-&gt;pos_y[i]; //1flop
		//dz = particles-&gt;pos_z[j] - particles-&gt;pos_z[i]; //1flop
		dx = particles_pos_x[j] - particles_pos_x[i]; //1flop
		dy = particles_pos_y[j] - particles_pos_y[i]; //1flop
		dz = particles_pos_z[j] - particles_pos_z[i]; //1flop
		&lt;...&gt;
	}
	&lt;...&gt;
}
</code></pre>
<h2 id="some-quick-software-engineering..">Some Quick Software Engineering…</h2>
<p>It is a good idea to move your computational kernels to a separate file.</p>
<ul>
<li>Use a different compiler for the SYCL offload as opposed to the rest of the code</li>
<li>Easy recompiles with SYCL enabled/disabled for correctness verification</li>
</ul>

