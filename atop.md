<h2 id="调研目的">调研目的</h2>
<p>对于某些不容易复现的问题，常常需要调查过去系统记录(cpu, memory, network, storage)来推测可能发生什么问题，目前最常用的是sar，sar是系统就预装的，属于sysstat套件，优点是overhead很低，对整个系统的资源使用记录非常完整，但是缺点是没有记录每个process的状态，当从sar report发现系统负载过大，或是某些资源使用过多时，我们仅能依靠经验法则去推测可能引发这些问题的服务，但这样的方式比较费时，不精确且缺乏佐证，如果有了进程状态的记录，相信能提供更多线索，定位问题也比较精确，sysstat的pidstat完全有能力打印出进程状态，但是没有整合搜集到sar report里(截至sysstat-12.1.3)，所以看不到这些进程信息。因此，调查是否有类似sar的工具，长时间记录系统信息，并且能细化到每个进程的信息，调查后，发现atop工具有这个功能。</p>
<h2 id="atop-安装">atop 安装</h2>
<pre class=" language-bash"><code class="prism  language-bash">$ yum <span class="token function">install</span> atop
</code></pre>
<h2 id="atop-使用">atop 使用</h2>
<p>与top相同，可以即时监控</p>
<pre class=" language-bash"><code class="prism  language-bash">$ atop
ATOP - ken-escore                                        2019/02/22  11:56:14                                        --------------                                        1d16h1m16s elapsed
PRC <span class="token operator">|</span> sys    3m21s <span class="token operator">|</span> user  64.76s  <span class="token operator">|</span>              <span class="token operator">|</span> <span class="token comment">#proc    119 | #trun      1  | #tslpi   217 |              | #tslpu     0  | #zombie    0 | clones 19124 |               | no  procacct |</span>
CPU <span class="token operator">|</span> sys       0% <span class="token operator">|</span> user      0%  <span class="token operator">|</span> irq       0% <span class="token operator">|</span>              <span class="token operator">|</span> idle    200%  <span class="token operator">|</span> <span class="token function">wait</span>      0% <span class="token operator">|</span>              <span class="token operator">|</span>               <span class="token operator">|</span> steal     0% <span class="token operator">|</span> guest     0% <span class="token operator">|</span> curf 2.66GHz  <span class="token operator">|</span> curscal   ?% <span class="token operator">|</span>
cpu <span class="token operator">|</span> sys       0% <span class="token operator">|</span> user      0%  <span class="token operator">|</span> irq       0% <span class="token operator">|</span>              <span class="token operator">|</span> idle    100%  <span class="token operator">|</span> cpu000 w  0% <span class="token operator">|</span>              <span class="token operator">|</span>               <span class="token operator">|</span> steal     0% <span class="token operator">|</span> guest     0% <span class="token operator">|</span> curf 2.66GHz  <span class="token operator">|</span> curscal   ?% <span class="token operator">|</span>
cpu <span class="token operator">|</span> sys       0% <span class="token operator">|</span> user      0%  <span class="token operator">|</span> irq       0% <span class="token operator">|</span>              <span class="token operator">|</span> idle    100%  <span class="token operator">|</span> cpu001 w  0% <span class="token operator">|</span>              <span class="token operator">|</span>               <span class="token operator">|</span> steal     0% <span class="token operator">|</span> guest     0% <span class="token operator">|</span> curf 2.66GHz  <span class="token operator">|</span> curscal   ?% <span class="token operator">|</span>
CPL <span class="token operator">|</span> avg1    0.00 <span class="token operator">|</span> avg5    0.01  <span class="token operator">|</span>              <span class="token operator">|</span> avg15   0.05 <span class="token operator">|</span>               <span class="token operator">|</span>              <span class="token operator">|</span> csw 13361143 <span class="token operator">|</span> intr 12790e3  <span class="token operator">|</span>              <span class="token operator">|</span>              <span class="token operator">|</span> numcpu     2  <span class="token operator">|</span>              <span class="token operator">|</span>
MEM <span class="token operator">|</span> tot     3.5G <span class="token operator">|</span> <span class="token function">free</span>  992.3M  <span class="token operator">|</span> cache   2.2G <span class="token operator">|</span> dirty   0.0M <span class="token operator">|</span> buff    5.1M  <span class="token operator">|</span> slab  105.2M <span class="token operator">|</span> slrec  68.6M <span class="token operator">|</span> shmem   0.6M  <span class="token operator">|</span> shrss   0.0M <span class="token operator">|</span> vmbal   0.0M <span class="token operator">|</span> hptot   0.0M  <span class="token operator">|</span> hpuse   0.0M <span class="token operator">|</span>
SWP <span class="token operator">|</span> tot     2.0G <span class="token operator">|</span> <span class="token function">free</span>    2.0G  <span class="token operator">|</span>              <span class="token operator">|</span>              <span class="token operator">|</span>               <span class="token operator">|</span>              <span class="token operator">|</span>              <span class="token operator">|</span>               <span class="token operator">|</span>              <span class="token operator">|</span>              <span class="token operator">|</span> vmcom   1.0G  <span class="token operator">|</span> vmlim   3.7G <span class="token operator">|</span>
LVM <span class="token operator">|</span>     ecl-root <span class="token operator">|</span> busy      0%  <span class="token operator">|</span> <span class="token function">read</span>   35489 <span class="token operator">|</span>              <span class="token operator">|</span> <span class="token function">write</span>  90264  <span class="token operator">|</span> KiB/r     58 <span class="token operator">|</span> KiB/w     14 <span class="token operator">|</span> MBr/s    0.0  <span class="token operator">|</span> MBw/s    0.0 <span class="token operator">|</span>              <span class="token operator">|</span> avq     5.52  <span class="token operator">|</span> avio 1.44 ms <span class="token operator">|</span>
LVM <span class="token operator">|</span>     ecl-swap <span class="token operator">|</span> busy      0%  <span class="token operator">|</span> <span class="token function">read</span>     133 <span class="token operator">|</span>              <span class="token operator">|</span> <span class="token function">write</span>      0  <span class="token operator">|</span> KiB/r     24 <span class="token operator">|</span> KiB/w      0 <span class="token operator">|</span> MBr/s    0.0  <span class="token operator">|</span> MBw/s    0.0 <span class="token operator">|</span>              <span class="token operator">|</span> avq     1.71  <span class="token operator">|</span> avio 7.54 ms <span class="token operator">|</span>
DSK <span class="token operator">|</span>          sdb <span class="token operator">|</span> busy      0%  <span class="token operator">|</span> <span class="token function">read</span>   32068 <span class="token operator">|</span>              <span class="token operator">|</span> <span class="token function">write</span>  96532  <span class="token operator">|</span> KiB/r     60 <span class="token operator">|</span> KiB/w     12 <span class="token operator">|</span> MBr/s    0.0  <span class="token operator">|</span> MBw/s    0.0 <span class="token operator">|</span>              <span class="token operator">|</span> avq     4.90  <span class="token operator">|</span> avio 1.29 ms <span class="token operator">|</span>
DSK <span class="token operator">|</span>          sdc <span class="token operator">|</span> busy      0%  <span class="token operator">|</span> <span class="token function">read</span>    4135 <span class="token operator">|</span>              <span class="token operator">|</span> <span class="token function">write</span>  18626  <span class="token operator">|</span> KiB/r     42 <span class="token operator">|</span> KiB/w      6 <span class="token operator">|</span> MBr/s    0.0  <span class="token operator">|</span> MBw/s    0.0 <span class="token operator">|</span>              <span class="token operator">|</span> avq     1.30  <span class="token operator">|</span> avio 2.05 ms <span class="token operator">|</span>
DSK <span class="token operator">|</span>          sda <span class="token operator">|</span> busy      0%  <span class="token operator">|</span> <span class="token function">read</span>     112 <span class="token operator">|</span>              <span class="token operator">|</span> <span class="token function">write</span>      0  <span class="token operator">|</span> KiB/r     19 <span class="token operator">|</span> KiB/w      0 <span class="token operator">|</span> MBr/s    0.0  <span class="token operator">|</span> MBw/s    0.0 <span class="token operator">|</span>              <span class="token operator">|</span> avq     1.20  <span class="token operator">|</span> avio 0.09 ms <span class="token operator">|</span>
NFM <span class="token operator">|</span> en/workspace <span class="token operator">|</span> srv 192.168.  <span class="token operator">|</span> <span class="token function">read</span>    413K <span class="token operator">|</span> <span class="token function">write</span>     0K <span class="token operator">|</span> nread   441K  <span class="token operator">|</span> nwrit     0K <span class="token operator">|</span> dread     0K <span class="token operator">|</span> dwrit     0K  <span class="token operator">|</span> mread   416K <span class="token operator">|</span> mwrit     0K <span class="token operator">|</span>               <span class="token operator">|</span>              <span class="token operator">|</span>
NFC <span class="token operator">|</span> rpc       58 <span class="token operator">|</span> <span class="token function">read</span>       3  <span class="token operator">|</span> <span class="token function">write</span>      0 <span class="token operator">|</span>              <span class="token operator">|</span> retxmit    0  <span class="token operator">|</span> autref    58 <span class="token operator">|</span>              <span class="token operator">|</span>               <span class="token operator">|</span>              <span class="token operator">|</span>              <span class="token operator">|</span>               <span class="token operator">|</span>              <span class="token operator">|</span>
NET <span class="token operator">|</span> transport    <span class="token operator">|</span> tcpi   94584  <span class="token operator">|</span> tcpo   90374 <span class="token operator">|</span> udpi     174 <span class="token operator">|</span> udpo     182  <span class="token operator">|</span> tcpao     25 <span class="token operator">|</span> tcppo      5 <span class="token operator">|</span> tcprs      3  <span class="token operator">|</span> tcpie      0 <span class="token operator">|</span> tcpor      3 <span class="token operator">|</span> udpnp      8  <span class="token operator">|</span> udpie      0 <span class="token operator">|</span>
NET <span class="token operator">|</span> network      <span class="token operator">|</span> ipi    94777  <span class="token operator">|</span> ipo    90580 <span class="token operator">|</span>              <span class="token operator">|</span> ipfrw      0  <span class="token operator">|</span> deliv  94774 <span class="token operator">|</span>              <span class="token operator">|</span>               <span class="token operator">|</span>              <span class="token operator">|</span>              <span class="token operator">|</span> icmpi      8  <span class="token operator">|</span> icmpo     13 <span class="token operator">|</span>
NET <span class="token operator">|</span> ens3      0% <span class="token operator">|</span> pcki   15118  <span class="token operator">|</span> pcko    9437 <span class="token operator">|</span> sp  100 Mbps <span class="token operator">|</span> si    0 Kbps  <span class="token operator">|</span> so    0 Kbps <span class="token operator">|</span> coll       0 <span class="token operator">|</span> mlti       0  <span class="token operator">|</span> erri       0 <span class="token operator">|</span> erro       0 <span class="token operator">|</span> drpi       0  <span class="token operator">|</span> drpo       0 <span class="token operator">|</span>
NET <span class="token operator">|</span> lo      ---- <span class="token operator">|</span> pcki   81403  <span class="token operator">|</span> pcko   81403 <span class="token operator">|</span> sp    0 Mbps <span class="token operator">|</span> si    0 Kbps  <span class="token operator">|</span> so    0 Kbps <span class="token operator">|</span> coll       0 <span class="token operator">|</span> mlti       0  <span class="token operator">|</span> erri       0 <span class="token operator">|</span> erro       0 <span class="token operator">|</span> drpi       0  <span class="token operator">|</span> drpo       0 <span class="token operator">|</span>
                                                                        *** system and process activity since boot ***
  PID        SYSCPU         USRCPU         VGROW          RGROW        RUID             EUID            ST         EXC         THR         S        CPUNR          CPU        CMD         1/4
    4        75.51s          0.00s            0K             0K        root             root            N-           -           1         S            0           0%        kworker/0:0
 1784        25.42s         19.24s        650.4M         24620K        root             root            N-           -          64         S            0           0%        ceph-osd
  436        41.49s          0.00s            0K             0K        root             root            N-           -           1         S            0           0%        xfsaild/dm-0
 1610        10.39s         10.10s        238.4M         29416K        root             root            N-           -          15         S            0           0%        ceph-mon
  839         4.46s         11.81s          2.3G         23412K        root             root            N-           -           3         S            0           0%        rsyslogd
 1230         2.87s         13.26s        560.1M         16720K        root             root            N-           -           5         S            0           0%        tuned
</code></pre>
<p>和sar相同，可启动atop服务，预设每10分钟搜集一次系统信息</p>
<pre class=" language-bash"><code class="prism  language-bash">$ systemctl start atop
</code></pre>
<p>可在/var/log/atop下看到二进制的日志</p>
<pre class=" language-bash"><code class="prism  language-bash">$ <span class="token function">ls</span> /var/log/atop/
atop_20190218  atop_20190219  atop_20190220  atop_20190221  atop_20190222  daily.log
</code></pre>
<p>使用atop -r读取某日期日志</p>
<pre class=" language-bash"><code class="prism  language-bash">$ atop -r /var/log/atop/atop_20190218
ATOP - ken-escore                                         2019/02/18  17:56:52                                         --------------                                          22m28s elapsed
PRC <span class="token operator">|</span> sys    9.40s <span class="token operator">|</span> user   3.36s  <span class="token operator">|</span>              <span class="token operator">|</span> <span class="token comment">#proc    113 | #trun      2  | #tslpi   210 |              | #tslpu     0  | #zombie    0 | clones  3627 |               | no  procacct |</span>
CPU <span class="token operator">|</span> sys       1% <span class="token operator">|</span> user      2%  <span class="token operator">|</span> irq       0% <span class="token operator">|</span>              <span class="token operator">|</span> idle    187%  <span class="token operator">|</span> <span class="token function">wait</span>     10% <span class="token operator">|</span>              <span class="token operator">|</span>               <span class="token operator">|</span> steal     0% <span class="token operator">|</span> guest     0% <span class="token operator">|</span> curf 2.66GHz  <span class="token operator">|</span> curscal   ?% <span class="token operator">|</span>
cpu <span class="token operator">|</span> sys       1% <span class="token operator">|</span> user      1%  <span class="token operator">|</span> irq       0% <span class="token operator">|</span>              <span class="token operator">|</span> idle     94%  <span class="token operator">|</span> cpu001 w  4% <span class="token operator">|</span>              <span class="token operator">|</span>               <span class="token operator">|</span> steal     0% <span class="token operator">|</span> guest     0% <span class="token operator">|</span> curf 2.66GHz  <span class="token operator">|</span> curscal   ?% <span class="token operator">|</span>
cpu <span class="token operator">|</span> sys       1% <span class="token operator">|</span> user      1%  <span class="token operator">|</span> irq       0% <span class="token operator">|</span>              <span class="token operator">|</span> idle     93%  <span class="token operator">|</span> cpu000 w  6% <span class="token operator">|</span>              <span class="token operator">|</span>               <span class="token operator">|</span> steal     0% <span class="token operator">|</span> guest     0% <span class="token operator">|</span> curf 2.66GHz  <span class="token operator">|</span> curscal   ?% <span class="token operator">|</span>
CPL <span class="token operator">|</span> avg1    0.00 <span class="token operator">|</span> avg5    0.03  <span class="token operator">|</span>              <span class="token operator">|</span> avg15   0.12 <span class="token operator">|</span>               <span class="token operator">|</span>              <span class="token operator">|</span> csw   262178 <span class="token operator">|</span> intr  277376  <span class="token operator">|</span>              <span class="token operator">|</span>              <span class="token operator">|</span> numcpu     2  <span class="token operator">|</span>              <span class="token operator">|</span>
MEM <span class="token operator">|</span> tot     3.5G <span class="token operator">|</span> <span class="token function">free</span>    2.1G  <span class="token operator">|</span> cache   1.1G <span class="token operator">|</span> dirty   0.2M <span class="token operator">|</span> buff    3.1M  <span class="token operator">|</span> slab   67.5M <span class="token operator">|</span> slrec  37.9M <span class="token operator">|</span> shmem   0.5M  <span class="token operator">|</span> shrss   0.0M <span class="token operator">|</span> vmbal   0.0M <span class="token operator">|</span> hptot   0.0M  <span class="token operator">|</span> hpuse   0.0M <span class="token operator">|</span>
SWP <span class="token operator">|</span> tot     2.0G <span class="token operator">|</span> <span class="token function">free</span>    2.0G  <span class="token operator">|</span>              <span class="token operator">|</span>              <span class="token operator">|</span>               <span class="token operator">|</span>              <span class="token operator">|</span>              <span class="token operator">|</span>               <span class="token operator">|</span>              <span class="token operator">|</span>              <span class="token operator">|</span> vmcom   1.0G  <span class="token operator">|</span> vmlim   3.7G <span class="token operator">|</span>
LVM <span class="token operator">|</span>     ecl-root <span class="token operator">|</span> busy     10%  <span class="token operator">|</span> <span class="token function">read</span>   14235 <span class="token operator">|</span>              <span class="token operator">|</span> <span class="token function">write</span>   5109  <span class="token operator">|</span> KiB/r     75 <span class="token operator">|</span> KiB/w     28 <span class="token operator">|</span> MBr/s    0.8  <span class="token operator">|</span> MBw/s    0.1 <span class="token operator">|</span>              <span class="token operator">|</span> avq    39.43  <span class="token operator">|</span> avio 6.98 ms <span class="token operator">|</span>
LVM <span class="token operator">|</span>     ecl-swap <span class="token operator">|</span> busy      0%  <span class="token operator">|</span> <span class="token function">read</span>      94 <span class="token operator">|</span>              <span class="token operator">|</span> <span class="token function">write</span>      0  <span class="token operator">|</span> KiB/r     23 <span class="token operator">|</span> KiB/w      0 <span class="token operator">|</span> MBr/s    0.0  <span class="token operator">|</span> MBw/s    0.0 <span class="token operator">|</span>              <span class="token operator">|</span> avq     1.41  <span class="token operator">|</span> avio 31.6 ms <span class="token operator">|</span>
DSK <span class="token operator">|</span>          sda <span class="token operator">|</span> busy     10%  <span class="token operator">|</span> <span class="token function">read</span>   13022 <span class="token operator">|</span>              <span class="token operator">|</span> <span class="token function">write</span>   6840  <span class="token operator">|</span> KiB/r     70 <span class="token operator">|</span> KiB/w     15 <span class="token operator">|</span> MBr/s    0.7  <span class="token operator">|</span> MBw/s    0.1 <span class="token operator">|</span>              <span class="token operator">|</span> avq    36.77  <span class="token operator">|</span> avio 6.54 ms <span class="token operator">|</span>
DSK <span class="token operator">|</span>          sdb <span class="token operator">|</span> busy      2%  <span class="token operator">|</span> <span class="token function">read</span>    1647 <span class="token operator">|</span>              <span class="token operator">|</span> <span class="token function">write</span>   1089  <span class="token operator">|</span> KiB/r     99 <span class="token operator">|</span> KiB/w     38 <span class="token operator">|</span> MBr/s    0.1  <span class="token operator">|</span> MBw/s    0.0 <span class="token operator">|</span>              <span class="token operator">|</span> avq     2.25  <span class="token operator">|</span> avio 9.69 ms <span class="token operator">|</span>
DSK <span class="token operator">|</span>          sdc <span class="token operator">|</span> busy      0%  <span class="token operator">|</span> <span class="token function">read</span>     112 <span class="token operator">|</span>              <span class="token operator">|</span> <span class="token function">write</span>      0  <span class="token operator">|</span> KiB/r     19 <span class="token operator">|</span> KiB/w      0 <span class="token operator">|</span> MBr/s    0.0  <span class="token operator">|</span> MBw/s    0.0 <span class="token operator">|</span>              <span class="token operator">|</span> avq     1.56  <span class="token operator">|</span> avio 0.08 ms <span class="token operator">|</span>
NFM <span class="token operator">|</span> en/workspace <span class="token operator">|</span> srv 192.168.  <span class="token operator">|</span> <span class="token function">read</span>      0K <span class="token operator">|</span> <span class="token function">write</span>     0K <span class="token operator">|</span> nread     0K  <span class="token operator">|</span> nwrit     0K <span class="token operator">|</span> dread     0K <span class="token operator">|</span> dwrit     0K  <span class="token operator">|</span> mread     0K <span class="token operator">|</span> mwrit     0K <span class="token operator">|</span>               <span class="token operator">|</span>              <span class="token operator">|</span>
NFC <span class="token operator">|</span> rpc       24 <span class="token operator">|</span> <span class="token function">read</span>       0  <span class="token operator">|</span> <span class="token function">write</span>      0 <span class="token operator">|</span>              <span class="token operator">|</span> retxmit    0  <span class="token operator">|</span> autref    24 <span class="token operator">|</span>              <span class="token operator">|</span>               <span class="token operator">|</span>              <span class="token operator">|</span>              <span class="token operator">|</span>               <span class="token operator">|</span>              <span class="token operator">|</span>
NET <span class="token operator">|</span> transport    <span class="token operator">|</span> tcpi   11281  <span class="token operator">|</span> tcpo    9352 <span class="token operator">|</span> udpi      85 <span class="token operator">|</span> udpo      91  <span class="token operator">|</span> tcpao     65 <span class="token operator">|</span> tcppo      3 <span class="token operator">|</span> tcprs     14  <span class="token operator">|</span> tcpie      0 <span class="token operator">|</span> tcpor      5 <span class="token operator">|</span> udpnp      6  <span class="token operator">|</span> udpie      0 <span class="token operator">|</span>
NET <span class="token operator">|</span> network      <span class="token operator">|</span> ipi    11381  <span class="token operator">|</span> ipo     9476 <span class="token operator">|</span>              <span class="token operator">|</span> ipfrw      0  <span class="token operator">|</span> deliv  11379 <span class="token operator">|</span>              <span class="token operator">|</span>               <span class="token operator">|</span>              <span class="token operator">|</span>              <span class="token operator">|</span> icmpi      7  <span class="token operator">|</span> icmpo     13 <span class="token operator">|</span>
NET <span class="token operator">|</span> ens3      0% <span class="token operator">|</span> pcki   12144  <span class="token operator">|</span> pcko    8040 <span class="token operator">|</span> sp  100 Mbps <span class="token operator">|</span> si   95 Kbps  <span class="token operator">|</span> so    4 Kbps <span class="token operator">|</span> coll       0 <span class="token operator">|</span> mlti       0  <span class="token operator">|</span> erri       0 <span class="token operator">|</span> erro       0 <span class="token operator">|</span> drpi       0  <span class="token operator">|</span> drpo       0 <span class="token operator">|</span>
NET <span class="token operator">|</span> lo      ---- <span class="token operator">|</span> pcki    1449  <span class="token operator">|</span> pcko    1449 <span class="token operator">|</span> sp    0 Mbps <span class="token operator">|</span> si    1 Kbps  <span class="token operator">|</span> so    1 Kbps <span class="token operator">|</span> coll       0 <span class="token operator">|</span> mlti       0  <span class="token operator">|</span> erri       0 <span class="token operator">|</span> erro       0 <span class="token operator">|</span> drpi       0  <span class="token operator">|</span> drpo       0 <span class="token operator">|</span>
                                                                        *** system and process activity since boot ***
  PID      SYSCPU       USRCPU       VGROW        RGROW       RDDSK        WRDSK       RUID          EUID           ST      EXC        THR       S      CPUNR        CPU      CMD         1/4
    1       2.90s        0.64s      189.5M        7208K      90401K          92K       root          root           N-        -          1       S          1         0%      systemd
 3117       0.64s        0.41s      652.7M       27576K      77673K        2876K       root          root           N-        -         64       S          0         0%      ceph-osd
  881       0.25s        0.55s      349.8M       28996K      15808K           8K       root          root           N-        -          2       S          1         0%      firewalld
  839       0.43s        0.27s        2.3G       24344K      205.5M         180K       root          root           N-        -          3       S          0         0%      rsyslogd
  265       0.65s        0.00s          0K           0K          0K           0K       root          root           N-        -          1       S          0         0%      kworker/0:2
 1793       0.30s        0.29s      233.1M       25988K      26400K       19068K       root          root           N-        -         15       S          0         0%      ceph-mon
  436       0.54s        0.00s          0K           0K         48K           0K       root          root           N-        -          1       S          0         0%      xfsaild/dm-0

</code></pre>
<p>切换想调查的资源</p>
<pre class=" language-bash"><code class="prism  language-bash">Figures shown <span class="token keyword">for</span> active processes:
        <span class="token string">'g'</span>  - generic info <span class="token punctuation">(</span>default<span class="token punctuation">)</span>
        <span class="token string">'m'</span>  - memory details
        <span class="token string">'d'</span>  - disk details
        <span class="token string">'n'</span>  - network details
        <span class="token string">'s'</span>  - scheduling and thread-group info
        <span class="token string">'v'</span>  - various info <span class="token punctuation">(</span>ppid, user/group, date/time, status, exitcode<span class="token punctuation">)</span>
        <span class="token string">'c'</span>  - full <span class="token function">command</span> line per process
        <span class="token string">'o'</span>  - use own output line definition
</code></pre>
<p>排序</p>
<pre class=" language-bash"><code class="prism  language-bash">Sort list of processes <span class="token keyword">in</span> order of:
        <span class="token string">'C'</span>  - cpu activity
        <span class="token string">'M'</span>  - memory consumption
        <span class="token string">'D'</span>  - disk activity
        <span class="token string">'N'</span>  - network activity
        <span class="token string">'A'</span>  - <span class="token function">most</span> active system resource <span class="token punctuation">(</span>auto mode<span class="token punctuation">)</span>
</code></pre>
<p>切换时间</p>
<pre class=" language-bash"><code class="prism  language-bash">Raw <span class="token function">file</span> viewing:
        <span class="token string">'t'</span>  - show next     sample <span class="token keyword">in</span> raw <span class="token function">file</span>
        <span class="token string">'T'</span>  - show previous sample <span class="token keyword">in</span> raw <span class="token function">file</span>
        <span class="token string">'b'</span>  - branch to certain <span class="token function">time</span> <span class="token keyword">in</span> raw <span class="token function">file</span>
        <span class="token string">'r'</span>  - rewind to begin of raw <span class="token function">file</span>
</code></pre>
<p>还可以打印出thread信息</p>
<pre class=" language-bash"><code class="prism  language-bash">Presentation <span class="token punctuation">(</span>keys shown <span class="token keyword">in</span> header line<span class="token punctuation">)</span>:
        <span class="token string">'y'</span>  - show individual threads                        <span class="token punctuation">(</span>toggle<span class="token punctuation">)</span>
</code></pre>
<p>另外有atopsar命令提供与sar类似的输出</p>
<h2 id="对系统的overhead">对系统的overhead</h2>
<p>因为与atop是一个定期搜集系统信息的服务，所以需要关注下对系统造成的额外负担，以sar和atop做个对比，实验为每1秒搜集一次系统信息，持续10秒</p>
<pre class=" language-bash"><code class="prism  language-bash">$ <span class="token function">sudo</span> /usr/bin/time -v /usr/lib64/sa/sadc -F -L -D 1 10 test.sar
        Command being timed: <span class="token string">"/usr/lib64/sa/sadc -F -L -D 1 10 test.sar"</span>
        User <span class="token function">time</span> <span class="token punctuation">(</span>seconds<span class="token punctuation">)</span>: 0.00
        System <span class="token function">time</span> <span class="token punctuation">(</span>seconds<span class="token punctuation">)</span>: 0.01
        Percent of CPU this job got: 0%
        Elapsed <span class="token punctuation">(</span>wall clock<span class="token punctuation">)</span> <span class="token function">time</span> <span class="token punctuation">(</span>h:mm:ss or m:ss<span class="token punctuation">)</span>: 0:09.00
        Average shared text size <span class="token punctuation">(</span>kbytes<span class="token punctuation">)</span>: 0
        Average unshared data size <span class="token punctuation">(</span>kbytes<span class="token punctuation">)</span>: 0
        Average stack size <span class="token punctuation">(</span>kbytes<span class="token punctuation">)</span>: 0
        Average total size <span class="token punctuation">(</span>kbytes<span class="token punctuation">)</span>: 0
        Maximum resident <span class="token keyword">set</span> size <span class="token punctuation">(</span>kbytes<span class="token punctuation">)</span>: 944
        Average resident <span class="token keyword">set</span> size <span class="token punctuation">(</span>kbytes<span class="token punctuation">)</span>: 0
        Major <span class="token punctuation">(</span>requiring I/O<span class="token punctuation">)</span> page faults: 0
        Minor <span class="token punctuation">(</span>reclaiming a frame<span class="token punctuation">)</span> page faults: 517
        Voluntary context switches: 10
        Involuntary context switches: 1
        Swaps: 0
        File system inputs: 0
        File system outputs: 40
        Socket messages sent: 0
        Socket messages received: 0
        Signals delivered: 0
        Page size <span class="token punctuation">(</span>bytes<span class="token punctuation">)</span>: 4096
        Exit status: 0
$ <span class="token function">sudo</span> /usr/bin/time -v /usr/bin/atop -R -w atop.log 1 10
        Command being timed: <span class="token string">"/usr/bin/atop -R -w atop.log 1 10"</span>
        User <span class="token function">time</span> <span class="token punctuation">(</span>seconds<span class="token punctuation">)</span>: 0.23
        System <span class="token function">time</span> <span class="token punctuation">(</span>seconds<span class="token punctuation">)</span>: 0.63
        Percent of CPU this job got: 9%
        Elapsed <span class="token punctuation">(</span>wall clock<span class="token punctuation">)</span> <span class="token function">time</span> <span class="token punctuation">(</span>h:mm:ss or m:ss<span class="token punctuation">)</span>: 0:09.08
        Average shared text size <span class="token punctuation">(</span>kbytes<span class="token punctuation">)</span>: 0
        Average unshared data size <span class="token punctuation">(</span>kbytes<span class="token punctuation">)</span>: 0
        Average stack size <span class="token punctuation">(</span>kbytes<span class="token punctuation">)</span>: 0
        Average total size <span class="token punctuation">(</span>kbytes<span class="token punctuation">)</span>: 0
        Maximum resident <span class="token keyword">set</span> size <span class="token punctuation">(</span>kbytes<span class="token punctuation">)</span>: 7860
        Average resident <span class="token keyword">set</span> size <span class="token punctuation">(</span>kbytes<span class="token punctuation">)</span>: 0
        Major <span class="token punctuation">(</span>requiring I/O<span class="token punctuation">)</span> page faults: 0
        Minor <span class="token punctuation">(</span>reclaiming a frame<span class="token punctuation">)</span> page faults: 14407
        Voluntary context switches: 14
        Involuntary context switches: 52
        Swaps: 0
        File system inputs: 0
        File system outputs: 144
        Socket messages sent: 0
        Socket messages received: 0
        Signals delivered: 0
        Page size <span class="token punctuation">(</span>bytes<span class="token punctuation">)</span>: 4096
        Exit status: 0
</code></pre>
<p>发现atop对系统的负担比sar大很多，atop花费的时间与CPU使用率远大于sar，但10分钟搜集一次的话，应该还可接受。</p>
<p>仍需考量的问题</p>
<ol>
<li>在比较大且更复杂的环境，是否对搜集per-task的负担会更大</li>
</ol>
