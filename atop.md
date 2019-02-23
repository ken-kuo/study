---


---

<h2 id="调研目的">调研目的</h2>
<p>对于某些不容易复现的问题，常常需要调查过去系统记录(cpu, memory, network, storage)来推测可能发生什么问题，目前最常用的是sar，sar是系统就预装的，属于sysstat套件，优点是overhead很低，对整个系统的资源使用记录非常完整，但是缺点是没有记录每个process的状态，当从sar report发现系统负载过大，或是某些资源使用过多时，我们仅能依靠经验法则去推测可能引发这些问题的服务，但这样的方式比较费时，不精确且缺乏佐证，如果有了进程状态的记录，相信能提供更多线索，定位问题也比较精确，sysstat的pidstat完全有能力打印出进程状态，但是没有整合搜集到sar report里(截至sysstat-12.1.3)，所以看不到这些进程信息。因此，调查是否有类似sar的工具，长时间记录系统信息，并且能细化到每个进程的信息，调查后，发现atop工具有这个功能。</p>
<h2 id="atop-安装">atop 安装</h2>
<pre><code>$ yum install atop
</code></pre>
<h2 id="atop-使用">atop 使用</h2>
<p>与top相同，可以即时监控</p>
<pre><code>$ atop
ATOP - ken-escore                                        2019/02/22  11:56:14                                        --------------                                        1d16h1m16s elapsed
PRC | sys    3m21s | user  64.76s  |              | #proc    119 | #trun      1  | #tslpi   217 |              | #tslpu     0  | #zombie    0 | clones 19124 |               | no  procacct |
CPU | sys       0% | user      0%  | irq       0% |              | idle    200%  | wait      0% |              |               | steal     0% | guest     0% | curf 2.66GHz  | curscal   ?% |
cpu | sys       0% | user      0%  | irq       0% |              | idle    100%  | cpu000 w  0% |              |               | steal     0% | guest     0% | curf 2.66GHz  | curscal   ?% |
cpu | sys       0% | user      0%  | irq       0% |              | idle    100%  | cpu001 w  0% |              |               | steal     0% | guest     0% | curf 2.66GHz  | curscal   ?% |
CPL | avg1    0.00 | avg5    0.01  |              | avg15   0.05 |               |              | csw 13361143 | intr 12790e3  |              |              | numcpu     2  |              |
MEM | tot     3.5G | free  992.3M  | cache   2.2G | dirty   0.0M | buff    5.1M  | slab  105.2M | slrec  68.6M | shmem   0.6M  | shrss   0.0M | vmbal   0.0M | hptot   0.0M  | hpuse   0.0M |
SWP | tot     2.0G | free    2.0G  |              |              |               |              |              |               |              |              | vmcom   1.0G  | vmlim   3.7G |
LVM |     ecl-root | busy      0%  | read   35489 |              | write  90264  | KiB/r     58 | KiB/w     14 | MBr/s    0.0  | MBw/s    0.0 |              | avq     5.52  | avio 1.44 ms |
LVM |     ecl-swap | busy      0%  | read     133 |              | write      0  | KiB/r     24 | KiB/w      0 | MBr/s    0.0  | MBw/s    0.0 |              | avq     1.71  | avio 7.54 ms |
DSK |          sdb | busy      0%  | read   32068 |              | write  96532  | KiB/r     60 | KiB/w     12 | MBr/s    0.0  | MBw/s    0.0 |              | avq     4.90  | avio 1.29 ms |
DSK |          sdc | busy      0%  | read    4135 |              | write  18626  | KiB/r     42 | KiB/w      6 | MBr/s    0.0  | MBw/s    0.0 |              | avq     1.30  | avio 2.05 ms |
DSK |          sda | busy      0%  | read     112 |              | write      0  | KiB/r     19 | KiB/w      0 | MBr/s    0.0  | MBw/s    0.0 |              | avq     1.20  | avio 0.09 ms |
NFM | en/workspace | srv 192.168.  | read    413K | write     0K | nread   441K  | nwrit     0K | dread     0K | dwrit     0K  | mread   416K | mwrit     0K |               |              |
NFC | rpc       58 | read       3  | write      0 |              | retxmit    0  | autref    58 |              |               |              |              |               |              |
NET | transport    | tcpi   94584  | tcpo   90374 | udpi     174 | udpo     182  | tcpao     25 | tcppo      5 | tcprs      3  | tcpie      0 | tcpor      3 | udpnp      8  | udpie      0 |
NET | network      | ipi    94777  | ipo    90580 |              | ipfrw      0  | deliv  94774 |              |               |              |              | icmpi      8  | icmpo     13 |
NET | ens3      0% | pcki   15118  | pcko    9437 | sp  100 Mbps | si    0 Kbps  | so    0 Kbps | coll       0 | mlti       0  | erri       0 | erro       0 | drpi       0  | drpo       0 |
NET | lo      ---- | pcki   81403  | pcko   81403 | sp    0 Mbps | si    0 Kbps  | so    0 Kbps | coll       0 | mlti       0  | erri       0 | erro       0 | drpi       0  | drpo       0 |
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
<pre><code>$ systemctl start atop
</code></pre>
<p>可在/var/log/atop下看到二进制的日志</p>
<pre><code>$ ls /var/log/atop/
atop_20190218  atop_20190219  atop_20190220  atop_20190221  atop_20190222  daily.log
</code></pre>
<p>使用atop -r读取某日期日志</p>
<pre><code>$ atop -r /var/log/atop/atop_20190218
ATOP - ken-escore                                         2019/02/18  17:56:52                                         --------------                                          22m28s elapsed
PRC | sys    9.40s | user   3.36s  |              | #proc    113 | #trun      2  | #tslpi   210 |              | #tslpu     0  | #zombie    0 | clones  3627 |               | no  procacct |
CPU | sys       1% | user      2%  | irq       0% |              | idle    187%  | wait     10% |              |               | steal     0% | guest     0% | curf 2.66GHz  | curscal   ?% |
cpu | sys       1% | user      1%  | irq       0% |              | idle     94%  | cpu001 w  4% |              |               | steal     0% | guest     0% | curf 2.66GHz  | curscal   ?% |
cpu | sys       1% | user      1%  | irq       0% |              | idle     93%  | cpu000 w  6% |              |               | steal     0% | guest     0% | curf 2.66GHz  | curscal   ?% |
CPL | avg1    0.00 | avg5    0.03  |              | avg15   0.12 |               |              | csw   262178 | intr  277376  |              |              | numcpu     2  |              |
MEM | tot     3.5G | free    2.1G  | cache   1.1G | dirty   0.2M | buff    3.1M  | slab   67.5M | slrec  37.9M | shmem   0.5M  | shrss   0.0M | vmbal   0.0M | hptot   0.0M  | hpuse   0.0M |
SWP | tot     2.0G | free    2.0G  |              |              |               |              |              |               |              |              | vmcom   1.0G  | vmlim   3.7G |
LVM |     ecl-root | busy     10%  | read   14235 |              | write   5109  | KiB/r     75 | KiB/w     28 | MBr/s    0.8  | MBw/s    0.1 |              | avq    39.43  | avio 6.98 ms |
LVM |     ecl-swap | busy      0%  | read      94 |              | write      0  | KiB/r     23 | KiB/w      0 | MBr/s    0.0  | MBw/s    0.0 |              | avq     1.41  | avio 31.6 ms |
DSK |          sda | busy     10%  | read   13022 |              | write   6840  | KiB/r     70 | KiB/w     15 | MBr/s    0.7  | MBw/s    0.1 |              | avq    36.77  | avio 6.54 ms |
DSK |          sdb | busy      2%  | read    1647 |              | write   1089  | KiB/r     99 | KiB/w     38 | MBr/s    0.1  | MBw/s    0.0 |              | avq     2.25  | avio 9.69 ms |
DSK |          sdc | busy      0%  | read     112 |              | write      0  | KiB/r     19 | KiB/w      0 | MBr/s    0.0  | MBw/s    0.0 |              | avq     1.56  | avio 0.08 ms |
NFM | en/workspace | srv 192.168.  | read      0K | write     0K | nread     0K  | nwrit     0K | dread     0K | dwrit     0K  | mread     0K | mwrit     0K |               |              |
NFC | rpc       24 | read       0  | write      0 |              | retxmit    0  | autref    24 |              |               |              |              |               |              |
NET | transport    | tcpi   11281  | tcpo    9352 | udpi      85 | udpo      91  | tcpao     65 | tcppo      3 | tcprs     14  | tcpie      0 | tcpor      5 | udpnp      6  | udpie      0 |
NET | network      | ipi    11381  | ipo     9476 |              | ipfrw      0  | deliv  11379 |              |               |              |              | icmpi      7  | icmpo     13 |
NET | ens3      0% | pcki   12144  | pcko    8040 | sp  100 Mbps | si   95 Kbps  | so    4 Kbps | coll       0 | mlti       0  | erri       0 | erro       0 | drpi       0  | drpo       0 |
NET | lo      ---- | pcki    1449  | pcko    1449 | sp    0 Mbps | si    1 Kbps  | so    1 Kbps | coll       0 | mlti       0  | erri       0 | erro       0 | drpi       0  | drpo       0 |
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
<pre><code>Figures shown for active processes:
        'g'  - generic info (default)
        'm'  - memory details
        'd'  - disk details
        'n'  - network details
        's'  - scheduling and thread-group info
        'v'  - various info (ppid, user/group, date/time, status, exitcode)
        'c'  - full command line per process
        'o'  - use own output line definition
</code></pre>
<p>排序</p>
<pre><code>Sort list of processes in order of:
        'C'  - cpu activity
        'M'  - memory consumption
        'D'  - disk activity
        'N'  - network activity
        'A'  - most active system resource (auto mode)
</code></pre>
<p>切换时间</p>
<pre><code>Raw file viewing:
        't'  - show next     sample in raw file
        'T'  - show previous sample in raw file
        'b'  - branch to certain time in raw file
        'r'  - rewind to begin of raw file
</code></pre>
<p>还可以打印出thread信息</p>
<pre class=" language-bash"><code class="prism  language-bash">Presentation <span class="token punctuation">(</span>keys shown <span class="token keyword">in</span> header line<span class="token punctuation">)</span>:
        <span class="token string">'y'</span>  - show individual threads                        <span class="token punctuation">(</span>toggle<span class="token punctuation">)</span>
</code></pre>
<p>另外有atopsar命令提供与sar类似的输出</p>
<h2 id="对系统的overhead">对系统的overhead</h2>
<p>因为与atop是一个定期搜集系统信息的服务，所以需要关注下对系统造成的额外负担，以sar和atop做个对比，实验为每1秒搜集一次系统信息，持续10秒</p>
<pre><code>$ sudo /usr/bin/time -v /usr/lib64/sa/sadc -F -L -D 1 10 test.sar
        Command being timed: "/usr/lib64/sa/sadc -F -L -D 1 10 test.sar"
        User time (seconds): 0.00
        System time (seconds): 0.01
        Percent of CPU this job got: 0%
        Elapsed (wall clock) time (h:mm:ss or m:ss): 0:09.00
        Average shared text size (kbytes): 0
        Average unshared data size (kbytes): 0
        Average stack size (kbytes): 0
        Average total size (kbytes): 0
        Maximum resident set size (kbytes): 944
        Average resident set size (kbytes): 0
        Major (requiring I/O) page faults: 0
        Minor (reclaiming a frame) page faults: 517
        Voluntary context switches: 10
        Involuntary context switches: 1
        Swaps: 0
        File system inputs: 0
        File system outputs: 40
        Socket messages sent: 0
        Socket messages received: 0
        Signals delivered: 0
        Page size (bytes): 4096
        Exit status: 0
$ sudo /usr/bin/time -v /usr/bin/atop -R -w atop.log 1 10
        Command being timed: "/usr/bin/atop -R -w atop.log 1 10"
        User time (seconds): 0.23
        System time (seconds): 0.63
        Percent of CPU this job got: 9%
        Elapsed (wall clock) time (h:mm:ss or m:ss): 0:09.08
        Average shared text size (kbytes): 0
        Average unshared data size (kbytes): 0
        Average stack size (kbytes): 0
        Average total size (kbytes): 0
        Maximum resident set size (kbytes): 7860
        Average resident set size (kbytes): 0
        Major (requiring I/O) page faults: 0
        Minor (reclaiming a frame) page faults: 14407
        Voluntary context switches: 14
        Involuntary context switches: 52
        Swaps: 0
        File system inputs: 0
        File system outputs: 144
        Socket messages sent: 0
        Socket messages received: 0
        Signals delivered: 0
        Page size (bytes): 4096
        Exit status: 0
</code></pre>
<p>发现atop对系统的负担比sar大很多，atop花费的时间与CPU使用率远大于sar，但10分钟搜集一次的话，应该还可接受。因为atop只有一个task，所以上面看到9%是单一task在一个CPU占用9%的时间，以server环境几十个CPU来看并不高。</p>
<p>从systemctl status atop看到atop有个-R参数</p>
<pre><code>[ken@ken-escore ~]$ systemctl status atop
● atop.service - Atop advanced performance monitor
   Loaded: loaded (/usr/lib/systemd/system/atop.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2019-02-23 00:00:01 CST; 11h ago
     Docs: man:atop(1)
 Main PID: 20278 (atop)
   CGroup: /system.slice/atop.service
           └─20278 /usr/bin/atop -R -w /var/log/atop/atop_20190223 600
</code></pre>
<p>查下man atop，有说到搜集proportional set size相当花费CPU时间</p>
<pre><code>       R    Gather and calculate the proportional set size of processes (toggle).  Gathering of all values that are needed to calculate the PSIZE of a process is a relatively time-con‐
            suming task, so this key should only be active when analyzing the resident memory consumption of processes.
</code></pre>
<p>proportional set size (PSS) 解释，说明把shared page分配计到每个进程，能更真实反映进程实际用量。</p>
<blockquote>
<p>The “proportional set size” (PSS) of a process is the count of pages it has in memory, where each page is divided by the number of processes sharing it. So if a process has 1000 pages all to itself, and 1000 shared with one other process, its PSS will be 1500. The unique set size (USS), instead, is a simple count of unshared pages.</p>
</blockquote>
<p>在不考虑shared page用量的情况，把-R拿掉看下CPU用量，发现下降到4%</p>
<pre><code>$ sudo /usr/bin/time -v /usr/bin/atop -w atop.log 1 10
        Command being timed: "/usr/bin/atop -w atop.log 1 10"
        User time (seconds): 0.15
        System time (seconds): 0.23
        Percent of CPU this job got: 4%
        Elapsed (wall clock) time (h:mm:ss or m:ss): 0:09.04
        Average shared text size (kbytes): 0
        Average unshared data size (kbytes): 0
        Average stack size (kbytes): 0
        Average total size (kbytes): 0
        Maximum resident set size (kbytes): 7860
        Average resident set size (kbytes): 0
        Major (requiring I/O) page faults: 0
        Minor (reclaiming a frame) page faults: 13067
        Voluntary context switches: 13
        Involuntary context switches: 28
        Swaps: 0
        File system inputs: 0
        File system outputs: 136
        Socket messages sent: 0
        Socket messages received: 0
        Signals delivered: 0
        Page size (bytes): 4096
        Exit status: 0
</code></pre>
<h2 id="一些实验">一些实验</h2>
<p>stress-ng测试在2个CPU开启5个线程，可以看到CPU是接近200%的使用率，而不是100%，证明前面atop 9%用量是所有CPU共占9%。</p>
<pre><code>$ /usr/bin/time -v stress-ng --matrix 5 -t 5s
stress-ng: info:  [19813] dispatching hogs: 5 matrix
stress-ng: info:  [19813] successful run completed in 5.02s
        Command being timed: "stress-ng --matrix 5 -t 5s"
        User time (seconds): 10.00
        System time (seconds): 0.01
        Percent of CPU this job got: 199%
        Elapsed (wall clock) time (h:mm:ss or m:ss): 0:05.02
        Average shared text size (kbytes): 0
        Average unshared data size (kbytes): 0
        Average stack size (kbytes): 0
        Average total size (kbytes): 0
        Maximum resident set size (kbytes): 6300
        Average resident set size (kbytes): 0
        Major (requiring I/O) page faults: 0
        Minor (reclaiming a frame) page faults: 2121
        Voluntary context switches: 15
        Involuntary context switches: 1058
        Swaps: 0
        File system inputs: 0
        File system outputs: 8
        Socket messages sent: 0
        Socket messages received: 0
        Signals delivered: 0
        Page size (bytes): 4096
        Exit status: 0
</code></pre>
<h2 id="仍需考量的问题">仍需考量的问题</h2>
<ol>
<li>在比较大且更复杂的环境，是否对搜集per-task的负担会更大</li>
</ol>

