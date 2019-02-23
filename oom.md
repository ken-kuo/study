---


---

<h1 id="oom分析">oom分析</h1>
<h3 id="oom-calltrace">OOM calltrace</h3>
<pre><code>[&lt;ffffffff816a3e81&gt;] dump_stack+0x19/0x1b
[&lt;ffffffff8169f276&gt;] dump_header+0x90/0x229
[&lt;ffffffff810e93ac&gt;] ? ktime_get_ts64+0x4c/0xf0
[&lt;ffffffff8113d37f&gt;] ? delayacct_end+0x8f/0xb0
[&lt;ffffffff811863a4&gt;] oom_kill_process+0x254/0x3d0
[&lt;ffffffff81185e4d&gt;] ? oom_unkillable_task+0xcd/0x120
[&lt;ffffffff81185ef6&gt;] ? find_lock_task_mm+0x56/0xc0
[&lt;ffffffff81186be6&gt;] out_of_memory+0x4b6/0x4f0
[&lt;ffffffff8169fd7a&gt;] __alloc_pages_slowpath+0x5d6/0x724
[&lt;ffffffff8118cdb5&gt;] __alloc_pages_nodemask+0x405/0x420
[&lt;ffffffff811d40d5&gt;] alloc_pages_vma+0xb5/0x200
[&lt;ffffffff811b2370&gt;] handle_mm_fault+0xb60/0xfa0
[&lt;ffffffff816b00b4&gt;] __do_page_fault+0x154/0x450
[&lt;ffffffff816b03e5&gt;] do_page_fault+0x35/0x90
[&lt;ffffffff816ac608&gt;] page_fault+0x28/0x30
</code></pre>
<h3 id="何时发生oom">何时发生OOM</h3>
<pre class=" language-c"><code class="prism ++ language-c"><span class="token keyword">static</span> <span class="token keyword">inline</span> <span class="token keyword">struct</span> page <span class="token operator">*</span>
<span class="token function">__alloc_pages_slowpath</span><span class="token punctuation">(</span>gfp_t gfp_mask<span class="token punctuation">,</span> <span class="token keyword">unsigned</span> <span class="token keyword">int</span> order<span class="token punctuation">,</span>
        <span class="token keyword">struct</span> zonelist <span class="token operator">*</span>zonelist<span class="token punctuation">,</span> <span class="token keyword">enum</span> zone_type high_zoneidx<span class="token punctuation">,</span>
        nodemask_t <span class="token operator">*</span>nodemask<span class="token punctuation">,</span> <span class="token keyword">struct</span> zone <span class="token operator">*</span>preferred_zone<span class="token punctuation">,</span>
        <span class="token keyword">int</span> migratetype<span class="token punctuation">)</span>
<span class="token punctuation">{</span>
        <span class="token keyword">const</span> gfp_t wait <span class="token operator">=</span> gfp_mask <span class="token operator">&amp;</span> __GFP_WAIT<span class="token punctuation">;</span>
        <span class="token keyword">struct</span> page <span class="token operator">*</span>page <span class="token operator">=</span> <span class="token constant">NULL</span><span class="token punctuation">;</span>
        <span class="token keyword">int</span> alloc_flags<span class="token punctuation">;</span>
        <span class="token keyword">unsigned</span> <span class="token keyword">long</span> pages_reclaimed <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">;</span>
        <span class="token keyword">unsigned</span> <span class="token keyword">long</span> did_some_progress<span class="token punctuation">;</span>
        bool sync_migration <span class="token operator">=</span> false<span class="token punctuation">;</span>
        bool deferred_compaction <span class="token operator">=</span> false<span class="token punctuation">;</span>
        bool contended_compaction <span class="token operator">=</span> false<span class="token punctuation">;</span>

        <span class="token comment">/*
         * In the slowpath, we sanity check order to avoid ever trying to
         * reclaim &gt;= MAX_ORDER areas which will never succeed. Callers may
         * be using allocators in order of preference for an area that is
         * too large.
         */</span>
        <span class="token keyword">if</span> <span class="token punctuation">(</span>order <span class="token operator">&gt;=</span> MAX_ORDER<span class="token punctuation">)</span> <span class="token punctuation">{</span>
                <span class="token function">WARN_ON_ONCE</span><span class="token punctuation">(</span><span class="token operator">!</span><span class="token punctuation">(</span>gfp_mask <span class="token operator">&amp;</span> __GFP_NOWARN<span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
                <span class="token keyword">return</span> <span class="token constant">NULL</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/*
         * GFP_THISNODE (meaning __GFP_THISNODE, __GFP_NORETRY and
         * __GFP_NOWARN set) should not cause reclaim since the subsystem
         * (f.e. slab) using GFP_THISNODE may choose to trigger reclaim
         * using a larger set of nodes after it has established that the
         * allowed per node queues are empty and that nodes are
         * over allocated.
         */</span>
        <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token function">IS_ENABLED</span><span class="token punctuation">(</span>CONFIG_NUMA<span class="token punctuation">)</span> <span class="token operator">&amp;&amp;</span>
            <span class="token punctuation">(</span>gfp_mask <span class="token operator">&amp;</span> GFP_THISNODE<span class="token punctuation">)</span> <span class="token operator">==</span> GFP_THISNODE<span class="token punctuation">)</span>
                <span class="token keyword">goto</span> nopage<span class="token punctuation">;</span>

restart<span class="token punctuation">:</span>
        <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span><span class="token punctuation">(</span>gfp_mask <span class="token operator">&amp;</span> __GFP_NO_KSWAPD<span class="token punctuation">)</span><span class="token punctuation">)</span>
                <span class="token function">wake_all_kswapds</span><span class="token punctuation">(</span>order<span class="token punctuation">,</span> zonelist<span class="token punctuation">,</span> high_zoneidx<span class="token punctuation">,</span> preferred_zone<span class="token punctuation">)</span><span class="token punctuation">;</span>

        <span class="token comment">/*
         * OK, we're below the kswapd watermark and have kicked background
         * reclaim. Now things get more complex, so set up alloc_flags according
         * to how we want to proceed.
         */</span>
        alloc_flags <span class="token operator">=</span> <span class="token function">gfp_to_alloc_flags</span><span class="token punctuation">(</span>gfp_mask<span class="token punctuation">)</span><span class="token punctuation">;</span>

        <span class="token comment">/*
         * Find the true preferred zone if the allocation is unconstrained by
         * cpusets.
         */</span>
        <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span><span class="token punctuation">(</span>alloc_flags <span class="token operator">&amp;</span> ALLOC_CPUSET<span class="token punctuation">)</span> <span class="token operator">&amp;&amp;</span> <span class="token operator">!</span>nodemask<span class="token punctuation">)</span>
                <span class="token function">first_zones_zonelist</span><span class="token punctuation">(</span>zonelist<span class="token punctuation">,</span> high_zoneidx<span class="token punctuation">,</span> <span class="token constant">NULL</span><span class="token punctuation">,</span>
                                        <span class="token operator">&amp;</span>preferred_zone<span class="token punctuation">)</span><span class="token punctuation">;</span>

rebalance<span class="token punctuation">:</span>
        <span class="token comment">/* This is the last chance, in general, before the goto nopage. */</span>
        page <span class="token operator">=</span> <span class="token function">get_page_from_freelist</span><span class="token punctuation">(</span>gfp_mask<span class="token punctuation">,</span> nodemask<span class="token punctuation">,</span> order<span class="token punctuation">,</span> zonelist<span class="token punctuation">,</span>
                        high_zoneidx<span class="token punctuation">,</span> alloc_flags <span class="token operator">&amp;</span> <span class="token operator">~</span>ALLOC_NO_WATERMARKS<span class="token punctuation">,</span>
                        preferred_zone<span class="token punctuation">,</span> migratetype<span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token keyword">if</span> <span class="token punctuation">(</span>page<span class="token punctuation">)</span>
                <span class="token keyword">goto</span> got_pg<span class="token punctuation">;</span>

        <span class="token comment">/* Allocate without watermarks if the context allows */</span>
        <span class="token keyword">if</span> <span class="token punctuation">(</span>alloc_flags <span class="token operator">&amp;</span> ALLOC_NO_WATERMARKS<span class="token punctuation">)</span> <span class="token punctuation">{</span>
                <span class="token comment">/*
                 * Ignore mempolicies if ALLOC_NO_WATERMARKS on the grounds
                 * the allocation is high priority and these type of
                 * allocations are system rather than user orientated
                 */</span>
                zonelist <span class="token operator">=</span> <span class="token function">node_zonelist</span><span class="token punctuation">(</span><span class="token function">numa_node_id</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">,</span> gfp_mask<span class="token punctuation">)</span><span class="token punctuation">;</span>

                page <span class="token operator">=</span> <span class="token function">__alloc_pages_high_priority</span><span class="token punctuation">(</span>gfp_mask<span class="token punctuation">,</span> order<span class="token punctuation">,</span>
                                zonelist<span class="token punctuation">,</span> high_zoneidx<span class="token punctuation">,</span> nodemask<span class="token punctuation">,</span>
                                preferred_zone<span class="token punctuation">,</span> migratetype<span class="token punctuation">)</span><span class="token punctuation">;</span>
                <span class="token keyword">if</span> <span class="token punctuation">(</span>page<span class="token punctuation">)</span> <span class="token punctuation">{</span>
                        <span class="token keyword">goto</span> got_pg<span class="token punctuation">;</span>
                <span class="token punctuation">}</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/* Atomic allocations - we can't balance anything */</span>
        <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span>wait<span class="token punctuation">)</span>
                <span class="token keyword">goto</span> nopage<span class="token punctuation">;</span>

        <span class="token comment">/* Avoid recursion of direct reclaim */</span>
        <span class="token keyword">if</span> <span class="token punctuation">(</span>current<span class="token operator">-&gt;</span>flags <span class="token operator">&amp;</span> PF_MEMALLOC<span class="token punctuation">)</span>
                <span class="token keyword">goto</span> nopage<span class="token punctuation">;</span>

        <span class="token comment">/* Avoid allocations with no watermarks from looping endlessly */</span>
        <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token function">test_thread_flag</span><span class="token punctuation">(</span>TIF_MEMDIE<span class="token punctuation">)</span> <span class="token operator">&amp;&amp;</span> <span class="token operator">!</span><span class="token punctuation">(</span>gfp_mask <span class="token operator">&amp;</span> __GFP_NOFAIL<span class="token punctuation">)</span><span class="token punctuation">)</span>
                <span class="token keyword">goto</span> nopage<span class="token punctuation">;</span>

        <span class="token comment">/*
         * Try direct compaction. The first pass is asynchronous. Subsequent
         * attempts after direct reclaim are synchronous
         */</span>
        page <span class="token operator">=</span> <span class="token function">__alloc_pages_direct_compact</span><span class="token punctuation">(</span>gfp_mask<span class="token punctuation">,</span> order<span class="token punctuation">,</span>
                                        zonelist<span class="token punctuation">,</span> high_zoneidx<span class="token punctuation">,</span>
                                        nodemask<span class="token punctuation">,</span>
                                        alloc_flags<span class="token punctuation">,</span> preferred_zone<span class="token punctuation">,</span>
                                        migratetype<span class="token punctuation">,</span> sync_migration<span class="token punctuation">,</span>
                                        <span class="token operator">&amp;</span>contended_compaction<span class="token punctuation">,</span>
                                        <span class="token operator">&amp;</span>deferred_compaction<span class="token punctuation">,</span>
                                        <span class="token operator">&amp;</span>did_some_progress<span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token keyword">if</span> <span class="token punctuation">(</span>page<span class="token punctuation">)</span>
                <span class="token keyword">goto</span> got_pg<span class="token punctuation">;</span>
        sync_migration <span class="token operator">=</span> true<span class="token punctuation">;</span>

        <span class="token comment">/*
         * If compaction is deferred for high-order allocations, it is because
         * sync compaction recently failed. In this is the case and the caller
         * requested a movable allocation that does not heavily disrupt the
         * system then fail the allocation instead of entering direct reclaim.
         */</span>
        <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token punctuation">(</span>deferred_compaction <span class="token operator">||</span> contended_compaction<span class="token punctuation">)</span> <span class="token operator">&amp;&amp;</span>
                                                <span class="token punctuation">(</span>gfp_mask <span class="token operator">&amp;</span> __GFP_NO_KSWAPD<span class="token punctuation">)</span><span class="token punctuation">)</span>
                <span class="token keyword">goto</span> nopage<span class="token punctuation">;</span>

        <span class="token comment">/* Try direct reclaim and then allocating */</span>
        page <span class="token operator">=</span> <span class="token function">__alloc_pages_direct_reclaim</span><span class="token punctuation">(</span>gfp_mask<span class="token punctuation">,</span> order<span class="token punctuation">,</span>
                                        zonelist<span class="token punctuation">,</span> high_zoneidx<span class="token punctuation">,</span>
                                        nodemask<span class="token punctuation">,</span>
                                        alloc_flags<span class="token punctuation">,</span> preferred_zone<span class="token punctuation">,</span>
                                        migratetype<span class="token punctuation">,</span> <span class="token operator">&amp;</span>did_some_progress<span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token keyword">if</span> <span class="token punctuation">(</span>page<span class="token punctuation">)</span>
                <span class="token keyword">goto</span> got_pg<span class="token punctuation">;</span>

        <span class="token comment">/*
         * If we failed to make any progress reclaiming, then we are
         * running out of options and have to consider going OOM
         */</span>
        <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span>did_some_progress<span class="token punctuation">)</span> <span class="token punctuation">{</span>
                <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token punctuation">(</span>gfp_mask <span class="token operator">&amp;</span> __GFP_FS<span class="token punctuation">)</span> <span class="token operator">&amp;&amp;</span> <span class="token operator">!</span><span class="token punctuation">(</span>gfp_mask <span class="token operator">&amp;</span> __GFP_NORETRY<span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
                        <span class="token keyword">if</span> <span class="token punctuation">(</span>oom_killer_disabled<span class="token punctuation">)</span>
                                <span class="token keyword">goto</span> nopage<span class="token punctuation">;</span>
                        <span class="token comment">/* Coredumps can quickly deplete all memory reserves */</span>
                        <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token punctuation">(</span>current<span class="token operator">-&gt;</span>flags <span class="token operator">&amp;</span> PF_DUMPCORE<span class="token punctuation">)</span> <span class="token operator">&amp;&amp;</span>
                            <span class="token operator">!</span><span class="token punctuation">(</span>gfp_mask <span class="token operator">&amp;</span> __GFP_NOFAIL<span class="token punctuation">)</span><span class="token punctuation">)</span>
                                <span class="token keyword">goto</span> nopage<span class="token punctuation">;</span>
                        page <span class="token operator">=</span> <span class="token function">__alloc_pages_may_oom</span><span class="token punctuation">(</span>gfp_mask<span class="token punctuation">,</span> order<span class="token punctuation">,</span>
                                        zonelist<span class="token punctuation">,</span> high_zoneidx<span class="token punctuation">,</span>
                                        nodemask<span class="token punctuation">,</span> preferred_zone<span class="token punctuation">,</span>
                                        migratetype<span class="token punctuation">)</span><span class="token punctuation">;</span>
                        <span class="token keyword">if</span> <span class="token punctuation">(</span>page<span class="token punctuation">)</span>
                                <span class="token keyword">goto</span> got_pg<span class="token punctuation">;</span>

                        <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span><span class="token punctuation">(</span>gfp_mask <span class="token operator">&amp;</span> __GFP_NOFAIL<span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
                                <span class="token comment">/*
                                 * The oom killer is not called for high-order
                                 * allocations that may fail, so if no progress
                                 * is being made, there are no other options and
                                 * retrying is unlikely to help.
                                 */</span>
                                <span class="token keyword">if</span> <span class="token punctuation">(</span>order <span class="token operator">&gt;</span> PAGE_ALLOC_COSTLY_ORDER<span class="token punctuation">)</span>
                                        <span class="token keyword">goto</span> nopage<span class="token punctuation">;</span>
                                <span class="token comment">/*
                                 * The oom killer is not called for lowmem
                                 * allocations to prevent needlessly killing
                                 * innocent tasks.
                                 */</span>
                                <span class="token keyword">if</span> <span class="token punctuation">(</span>high_zoneidx <span class="token operator">&lt;</span> ZONE_NORMAL<span class="token punctuation">)</span>
                                        <span class="token keyword">goto</span> nopage<span class="token punctuation">;</span>
                        <span class="token punctuation">}</span>

                        <span class="token keyword">goto</span> restart<span class="token punctuation">;</span>
                <span class="token punctuation">}</span>
        <span class="token punctuation">}</span>

        <span class="token comment">/* Check if we should retry the allocation */</span>
        pages_reclaimed <span class="token operator">+</span><span class="token operator">=</span> did_some_progress<span class="token punctuation">;</span>
        <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token function">should_alloc_retry</span><span class="token punctuation">(</span>gfp_mask<span class="token punctuation">,</span> order<span class="token punctuation">,</span> did_some_progress<span class="token punctuation">,</span>
                                                pages_reclaimed<span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
                <span class="token comment">/* Wait for some write requests to complete then retry */</span>
                <span class="token function">wait_iff_congested</span><span class="token punctuation">(</span>preferred_zone<span class="token punctuation">,</span> BLK_RW_ASYNC<span class="token punctuation">,</span> HZ<span class="token operator">/</span><span class="token number">50</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
                <span class="token keyword">goto</span> rebalance<span class="token punctuation">;</span>
        <span class="token punctuation">}</span> <span class="token keyword">else</span> <span class="token punctuation">{</span>
                <span class="token comment">/*
                 * High-order allocations do not necessarily loop after
                 * direct reclaim and reclaim/compaction depends on compaction
                 * being called after reclaim so call directly if necessary
                 */</span>
                page <span class="token operator">=</span> <span class="token function">__alloc_pages_direct_compact</span><span class="token punctuation">(</span>gfp_mask<span class="token punctuation">,</span> order<span class="token punctuation">,</span>
                                        zonelist<span class="token punctuation">,</span> high_zoneidx<span class="token punctuation">,</span>
                                        nodemask<span class="token punctuation">,</span>
                                        alloc_flags<span class="token punctuation">,</span> preferred_zone<span class="token punctuation">,</span>
                                        migratetype<span class="token punctuation">,</span> sync_migration<span class="token punctuation">,</span>
                                        <span class="token operator">&amp;</span>contended_compaction<span class="token punctuation">,</span>
                                        <span class="token operator">&amp;</span>deferred_compaction<span class="token punctuation">,</span>
                                        <span class="token operator">&amp;</span>did_some_progress<span class="token punctuation">)</span><span class="token punctuation">;</span>
                <span class="token keyword">if</span> <span class="token punctuation">(</span>page<span class="token punctuation">)</span>
                        <span class="token keyword">goto</span> got_pg<span class="token punctuation">;</span>
        <span class="token punctuation">}</span>

nopage<span class="token punctuation">:</span>
        <span class="token function">warn_alloc_failed</span><span class="token punctuation">(</span>gfp_mask<span class="token punctuation">,</span> order<span class="token punctuation">,</span> <span class="token constant">NULL</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token keyword">return</span> page<span class="token punctuation">;</span>
got_pg<span class="token punctuation">:</span>
        <span class="token keyword">if</span> <span class="token punctuation">(</span>kmemcheck_enabled<span class="token punctuation">)</span>
                <span class="token function">kmemcheck_pagealloc_alloc</span><span class="token punctuation">(</span>page<span class="token punctuation">,</span> order<span class="token punctuation">,</span> gfp_mask<span class="token punctuation">)</span><span class="token punctuation">;</span>

        <span class="token keyword">return</span> page<span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>

