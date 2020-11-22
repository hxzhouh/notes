---
author: hxzhouh

---

<h1 id="panic">panic</h1>
<h2 id="panic-的本质">panic 的本质</h2>
<p>panic 的本质其实就是 gopanic，用户代码和 runtime 中都会有对 panic() 的调用，最终会转为这个 runtime.gopanic 函数。</p>
<pre class=" language-go"><code class="prism  language-go"><span class="token keyword">func</span> <span class="token function">gopanic</span><span class="token punctuation">(</span>e <span class="token keyword">interface</span><span class="token punctuation">{</span><span class="token punctuation">}</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token comment">// 获取当前 g</span>
    gp <span class="token operator">:=</span> <span class="token function">getg</span><span class="token punctuation">(</span><span class="token punctuation">)</span>

    <span class="token comment">// 生成一个新的 panic 结构体</span>
    <span class="token keyword">var</span> p _panic
    p<span class="token punctuation">.</span>arg <span class="token operator">=</span> e
    <span class="token comment">// 链上之前的 panic 结构</span>
    p<span class="token punctuation">.</span>link <span class="token operator">=</span> gp<span class="token punctuation">.</span>_panic
    <span class="token comment">// 新节点做整个链表的表头</span>
    gp<span class="token punctuation">.</span>_panic <span class="token operator">=</span> <span class="token punctuation">(</span><span class="token operator">*</span>_panic<span class="token punctuation">)</span><span class="token punctuation">(</span><span class="token function">noescape</span><span class="token punctuation">(</span>unsafe<span class="token punctuation">.</span><span class="token function">Pointer</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>p<span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">)</span>

    <span class="token comment">// 统计</span>
    atomic<span class="token punctuation">.</span><span class="token function">Xadd</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>runningPanicDefers<span class="token punctuation">,</span> <span class="token number">1</span><span class="token punctuation">)</span>

    <span class="token keyword">for</span> <span class="token punctuation">{</span>
        d <span class="token operator">:=</span> gp<span class="token punctuation">.</span>_defer
        <span class="token keyword">if</span> d <span class="token operator">==</span> <span class="token boolean">nil</span> <span class="token punctuation">{</span>
            <span class="token keyword">break</span>
        <span class="token punctuation">}</span>

        <span class="token comment">// 标记 defer 为已启动，暂时保持在链表上</span>
        <span class="token comment">// 这样 traceback 在栈增长或者 GC 的时候，能够找到并更新 defer 的参数栈帧</span>
        <span class="token comment">// 并用 reflectcall 执行 d.fn</span>
        d<span class="token punctuation">.</span>started <span class="token operator">=</span> <span class="token boolean">true</span>

        <span class="token comment">// 记录在 defer 中发生的 panic</span>
        <span class="token comment">// 如果在 defer 的函数调用过程中又发生了新的 panic，那个 panic 会在链表中找到 d</span>
        <span class="token comment">// 然后标记 d._panic(指向当前的 panic) 为 aborted 状态。</span>
        d<span class="token punctuation">.</span>_panic <span class="token operator">=</span> <span class="token punctuation">(</span><span class="token operator">*</span>_panic<span class="token punctuation">)</span><span class="token punctuation">(</span><span class="token function">noescape</span><span class="token punctuation">(</span>unsafe<span class="token punctuation">.</span><span class="token function">Pointer</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>p<span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">)</span>

        p<span class="token punctuation">.</span>argp <span class="token operator">=</span> unsafe<span class="token punctuation">.</span><span class="token function">Pointer</span><span class="token punctuation">(</span><span class="token function">getargp</span><span class="token punctuation">(</span><span class="token number">0</span><span class="token punctuation">)</span><span class="token punctuation">)</span>
        <span class="token function">reflectcall</span><span class="token punctuation">(</span><span class="token boolean">nil</span><span class="token punctuation">,</span> unsafe<span class="token punctuation">.</span><span class="token function">Pointer</span><span class="token punctuation">(</span>d<span class="token punctuation">.</span>fn<span class="token punctuation">)</span><span class="token punctuation">,</span> <span class="token function">deferArgs</span><span class="token punctuation">(</span>d<span class="token punctuation">)</span><span class="token punctuation">,</span> <span class="token function">uint32</span><span class="token punctuation">(</span>d<span class="token punctuation">.</span>siz<span class="token punctuation">)</span><span class="token punctuation">,</span> <span class="token function">uint32</span><span class="token punctuation">(</span>d<span class="token punctuation">.</span>siz<span class="token punctuation">)</span><span class="token punctuation">)</span>
        p<span class="token punctuation">.</span>argp <span class="token operator">=</span> <span class="token boolean">nil</span>

        <span class="token comment">// reflectcall 没有 panic，移除 d</span>
        <span class="token keyword">if</span> gp<span class="token punctuation">.</span>_defer <span class="token operator">!=</span> d <span class="token punctuation">{</span>
            <span class="token function">throw</span><span class="token punctuation">(</span><span class="token string">"bad defer entry in panic"</span><span class="token punctuation">)</span>
        <span class="token punctuation">}</span>
        d<span class="token punctuation">.</span>_panic <span class="token operator">=</span> <span class="token boolean">nil</span>
        d<span class="token punctuation">.</span>fn <span class="token operator">=</span> <span class="token boolean">nil</span>
        gp<span class="token punctuation">.</span>_defer <span class="token operator">=</span> d<span class="token punctuation">.</span>link

        pc <span class="token operator">:=</span> d<span class="token punctuation">.</span>pc
        sp <span class="token operator">:=</span> unsafe<span class="token punctuation">.</span><span class="token function">Pointer</span><span class="token punctuation">(</span>d<span class="token punctuation">.</span>sp<span class="token punctuation">)</span> <span class="token comment">// must be pointer so it gets adjusted during stack copy</span>
        <span class="token function">freedefer</span><span class="token punctuation">(</span>d<span class="token punctuation">)</span>
        <span class="token comment">// p.recovered 字段在 gorecover 函数中已经修改为 true 了</span>
        <span class="token keyword">if</span> p<span class="token punctuation">.</span>recovered <span class="token punctuation">{</span>
            atomic<span class="token punctuation">.</span><span class="token function">Xadd</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>runningPanicDefers<span class="token punctuation">,</span> <span class="token operator">-</span><span class="token number">1</span><span class="token punctuation">)</span>

            gp<span class="token punctuation">.</span>_panic <span class="token operator">=</span> p<span class="token punctuation">.</span>link
            <span class="token comment">// Aborted panics are marked but remain on the g.panic list.</span>
            <span class="token comment">// Remove them from the list.</span>
            <span class="token keyword">for</span> gp<span class="token punctuation">.</span>_panic <span class="token operator">!=</span> <span class="token boolean">nil</span> <span class="token operator">&amp;&amp;</span> gp<span class="token punctuation">.</span>_panic<span class="token punctuation">.</span>aborted <span class="token punctuation">{</span>
                gp<span class="token punctuation">.</span>_panic <span class="token operator">=</span> gp<span class="token punctuation">.</span>_panic<span class="token punctuation">.</span>link
            <span class="token punctuation">}</span>
            <span class="token keyword">if</span> gp<span class="token punctuation">.</span>_panic <span class="token operator">==</span> <span class="token boolean">nil</span> <span class="token punctuation">{</span> <span class="token comment">// 必须已处理完 signal</span>
                gp<span class="token punctuation">.</span>sig <span class="token operator">=</span> <span class="token number">0</span>
            <span class="token punctuation">}</span>
            <span class="token comment">// 把需要恢复的栈帧信息传给 recovery 函数</span>
            gp<span class="token punctuation">.</span>sigcode0 <span class="token operator">=</span> <span class="token function">uintptr</span><span class="token punctuation">(</span>sp<span class="token punctuation">)</span>
            gp<span class="token punctuation">.</span>sigcode1 <span class="token operator">=</span> pc
            <span class="token function">mcall</span><span class="token punctuation">(</span>recovery<span class="token punctuation">)</span>
            <span class="token function">throw</span><span class="token punctuation">(</span><span class="token string">"recovery failed"</span><span class="token punctuation">)</span> <span class="token comment">// mcall 永远不会返回</span>
        <span class="token punctuation">}</span>
    <span class="token punctuation">}</span>
<span class="token punctuation">}</span>

</code></pre>
<p>gopanic 会把当前函数的 defer 一直执行完，其中碰到 recover 的话就会调用 gorecover 函数:</p>
<pre class=" language-go"><code class="prism  language-go"><span class="token keyword">func</span> <span class="token function">gorecover</span><span class="token punctuation">(</span>argp <span class="token builtin">uintptr</span><span class="token punctuation">)</span> <span class="token keyword">interface</span><span class="token punctuation">{</span><span class="token punctuation">}</span> <span class="token punctuation">{</span>
	gp <span class="token operator">:=</span> <span class="token function">getg</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
	p <span class="token operator">:=</span> gp<span class="token punctuation">.</span>_panic
	<span class="token keyword">if</span> p <span class="token operator">!=</span> <span class="token boolean">nil</span> <span class="token operator">&amp;&amp;</span> <span class="token operator">!</span>p<span class="token punctuation">.</span>recovered <span class="token operator">&amp;&amp;</span> argp <span class="token operator">==</span> <span class="token function">uintptr</span><span class="token punctuation">(</span>p<span class="token punctuation">.</span>argp<span class="token punctuation">)</span> <span class="token punctuation">{</span>
		p<span class="token punctuation">.</span>recovered <span class="token operator">=</span> <span class="token boolean">true</span>
		<span class="token keyword">return</span> p<span class="token punctuation">.</span>arg
	<span class="token punctuation">}</span>
	<span class="token keyword">return</span> <span class="token boolean">nil</span>
<span class="token punctuation">}</span>
</code></pre>
<p>很简单，就是改改 recovered 字段。</p>
<p>gopanic 里生成的 panic 对象是带指针的结构体，串成一个链表，用 link 指针相连接，并挂在 g 结构体上:</p>
<pre class=" language-go"><code class="prism  language-go"><span class="token comment">// panics</span>
<span class="token keyword">type</span> _panic <span class="token keyword">struct</span> <span class="token punctuation">{</span>
    argp      unsafe<span class="token punctuation">.</span>Pointer <span class="token comment">// pointer to arguments of deferred call run during panic; cannot move - known to liblink</span>
    arg       <span class="token keyword">interface</span><span class="token punctuation">{</span><span class="token punctuation">}</span>    <span class="token comment">// argument to panic</span>
    link      <span class="token operator">*</span>_panic        <span class="token comment">// link to earlier panic</span>
    recovered <span class="token builtin">bool</span>           <span class="token comment">// whether this panic is over</span>
    aborted   <span class="token builtin">bool</span>           <span class="token comment">// the panic was aborted</span>
<span class="token punctuation">}</span>
</code></pre>
<p>下面这种写法应该会形成一个 _panic 的链表:</p>
<pre class=" language-go"><code class="prism  language-go"><span class="token keyword">package</span> main

<span class="token keyword">func</span> <span class="token function">fxfx</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">defer</span> <span class="token keyword">func</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token keyword">if</span> err <span class="token operator">:=</span> <span class="token function">recover</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> err <span class="token operator">!=</span> <span class="token boolean">nil</span> <span class="token punctuation">{</span>
            <span class="token function">println</span><span class="token punctuation">(</span><span class="token string">"recover here"</span><span class="token punctuation">)</span>
        <span class="token punctuation">}</span>
    <span class="token punctuation">}</span><span class="token punctuation">(</span><span class="token punctuation">)</span>

    <span class="token keyword">defer</span> <span class="token keyword">func</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token function">panic</span><span class="token punctuation">(</span><span class="token number">1</span><span class="token punctuation">)</span>
    <span class="token punctuation">}</span><span class="token punctuation">(</span><span class="token punctuation">)</span>

    <span class="token keyword">defer</span> <span class="token keyword">func</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token function">panic</span><span class="token punctuation">(</span><span class="token number">2</span><span class="token punctuation">)</span>
    <span class="token punctuation">}</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
<span class="token punctuation">}</span>

<span class="token keyword">func</span> <span class="token function">main</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token function">fxfx</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
<span class="token punctuation">}</span>
</code></pre>
<p>非 defer 中的 panic 的话，函数的后续代码就没办法运行了，写代码的时候要注意这个特性。</p>
<h2 id="recovery">recovery</h2>
<p>上面的 panic 恢复过程，最终走到了 recovery:</p>
<pre class=" language-go"><code class="prism  language-go"><span class="token comment">// 把需要恢复的栈帧信息传给 recovery 函数</span>
gp<span class="token punctuation">.</span>sigcode0 <span class="token operator">=</span> <span class="token function">uintptr</span><span class="token punctuation">(</span>sp<span class="token punctuation">)</span>
gp<span class="token punctuation">.</span>sigcode1 <span class="token operator">=</span> pc
<span class="token function">mcall</span><span class="token punctuation">(</span>recovery<span class="token punctuation">)</span>
</code></pre>
<pre class=" language-go"><code class="prism  language-go"><span class="token comment">// 在被 defer 的函数中调用的 recover 从 panic 中已恢复，展开栈继续运行。</span>
<span class="token comment">// 现场恢复为发生 panic 的函数正常返回时的情况</span>
<span class="token keyword">func</span> <span class="token function">recovery</span><span class="token punctuation">(</span>gp <span class="token operator">*</span>g<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token comment">// 之前之前传入的 sp 和 pc 恢复</span>
    sp <span class="token operator">:=</span> gp<span class="token punctuation">.</span>sigcode0
    pc <span class="token operator">:=</span> gp<span class="token punctuation">.</span>sigcode1

    <span class="token comment">// d 的参数需要放在栈上</span>
    <span class="token keyword">if</span> sp <span class="token operator">!=</span> <span class="token number">0</span> <span class="token operator">&amp;&amp;</span> <span class="token punctuation">(</span>sp <span class="token operator">&lt;</span> gp<span class="token punctuation">.</span>stack<span class="token punctuation">.</span>lo <span class="token operator">||</span> gp<span class="token punctuation">.</span>stack<span class="token punctuation">.</span>hi <span class="token operator">&lt;</span> sp<span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token function">print</span><span class="token punctuation">(</span><span class="token string">"recover: "</span><span class="token punctuation">,</span> <span class="token function">hex</span><span class="token punctuation">(</span>sp<span class="token punctuation">)</span><span class="token punctuation">,</span> <span class="token string">" not in ["</span><span class="token punctuation">,</span> <span class="token function">hex</span><span class="token punctuation">(</span>gp<span class="token punctuation">.</span>stack<span class="token punctuation">.</span>lo<span class="token punctuation">)</span><span class="token punctuation">,</span> <span class="token string">", "</span><span class="token punctuation">,</span> <span class="token function">hex</span><span class="token punctuation">(</span>gp<span class="token punctuation">.</span>stack<span class="token punctuation">.</span>hi<span class="token punctuation">)</span><span class="token punctuation">,</span> <span class="token string">"]\n"</span><span class="token punctuation">)</span>
        <span class="token function">throw</span><span class="token punctuation">(</span><span class="token string">"bad recovery"</span><span class="token punctuation">)</span>
    <span class="token punctuation">}</span>

    <span class="token comment">// 让这个 defer 结构体的 deferproc 位置的调用重新返回</span>
    <span class="token comment">// 这次将返回值修改为 1</span>
    gp<span class="token punctuation">.</span>sched<span class="token punctuation">.</span>sp <span class="token operator">=</span> sp
    gp<span class="token punctuation">.</span>sched<span class="token punctuation">.</span>pc <span class="token operator">=</span> pc
    gp<span class="token punctuation">.</span>sched<span class="token punctuation">.</span>lr <span class="token operator">=</span> <span class="token number">0</span>
    gp<span class="token punctuation">.</span>sched<span class="token punctuation">.</span>ret <span class="token operator">=</span> <span class="token number">1</span> <span class="token comment">// ------&gt; 没有调用 deferproc，但直接修改了返回值，所以跳转到 deferproc 的下一条指令位置，且设置了 1，假装作为 deferproc 的返回值</span>
    <span class="token function">gogo</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>gp<span class="token punctuation">.</span>sched<span class="token punctuation">)</span>
<span class="token punctuation">}</span>
</code></pre>
<p>这里比较 trick，实际上 recovery 中传入的 pc 寄存器的值是 call deferproc 的下一条汇编指令的地址:</p>
<pre class=" language-go"><code class="prism  language-go">	<span class="token number">0x0034</span> <span class="token function">00052</span> <span class="token punctuation">(</span>x<span class="token punctuation">.</span><span class="token keyword">go</span><span class="token punctuation">:</span><span class="token number">4</span><span class="token punctuation">)</span>	CALL	runtime<span class="token punctuation">.</span><span class="token function">deferproc</span><span class="token punctuation">(</span>SB<span class="token punctuation">)</span>
	<span class="token number">0x0039</span> <span class="token function">00057</span> <span class="token punctuation">(</span>x<span class="token punctuation">.</span><span class="token keyword">go</span><span class="token punctuation">:</span><span class="token number">4</span><span class="token punctuation">)</span>	TESTL	AX<span class="token punctuation">,</span> AX <span class="token operator">--</span><span class="token operator">--</span><span class="token operator">-</span><span class="token operator">&gt;</span> 传入到 deferproc 中的 pc 寄存器指向的位置
	<span class="token number">0x003b</span> <span class="token function">00059</span> <span class="token punctuation">(</span>x<span class="token punctuation">.</span><span class="token keyword">go</span><span class="token punctuation">:</span><span class="token number">4</span><span class="token punctuation">)</span>	JNE	<span class="token number">135</span>
</code></pre>
<p>和刚注册 defer 结构体链表时的情况不同，panic 时，我们没有调用 deferproc，而是直接跳到了 deferproc 的下一条指令的地址上，并且检查 AX 的值，这里已经被改成 1 了。</p>
<p>所以会直接跳转到函数最后对应该 deferproc 的 deferreturn 位置去。</p>
<p>最后确认一次，runtime.gogo 会把 sched.ret 搬到 AX 寄存器:</p>
<pre class=" language-go"><code class="prism  language-go">TEXT runtime·<span class="token function">gogo</span><span class="token punctuation">(</span>SB<span class="token punctuation">)</span><span class="token punctuation">,</span> NOSPLIT<span class="token punctuation">,</span> $<span class="token number">16</span><span class="token operator">-</span><span class="token number">8</span>
	MOVQ	buf<span class="token operator">+</span><span class="token function">0</span><span class="token punctuation">(</span>FP<span class="token punctuation">)</span><span class="token punctuation">,</span> BX		<span class="token comment">// gobuf</span>
	MOVQ	<span class="token function">gobuf_g</span><span class="token punctuation">(</span>BX<span class="token punctuation">)</span><span class="token punctuation">,</span> DX
	MOVQ	<span class="token function">0</span><span class="token punctuation">(</span>DX<span class="token punctuation">)</span><span class="token punctuation">,</span> CX		<span class="token comment">// make sure g != nil</span>
	<span class="token function">get_tls</span><span class="token punctuation">(</span>CX<span class="token punctuation">)</span>
	MOVQ	DX<span class="token punctuation">,</span> <span class="token function">g</span><span class="token punctuation">(</span>CX<span class="token punctuation">)</span>
	MOVQ	<span class="token function">gobuf_sp</span><span class="token punctuation">(</span>BX<span class="token punctuation">)</span><span class="token punctuation">,</span> SP	<span class="token comment">// restore SP</span>
	MOVQ	<span class="token function">gobuf_ret</span><span class="token punctuation">(</span>BX<span class="token punctuation">)</span><span class="token punctuation">,</span> AX  <span class="token comment">// ----------&gt; 重点在这里</span>
	MOVQ	<span class="token function">gobuf_ctxt</span><span class="token punctuation">(</span>BX<span class="token punctuation">)</span><span class="token punctuation">,</span> DX
</code></pre>
<p>这样所有流程就都打通了。</p>

