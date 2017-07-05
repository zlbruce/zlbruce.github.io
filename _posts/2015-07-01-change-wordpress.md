---
title: 是时候换Blog程序了
author: zlbruce
layout: post
permalink: /2015/07/01/change-wordpress/
categories:
  - Blog
tags:
  - Blog
  - DigitalOcean
---
由于现在的 Blog 是放到 DO 的 VPS 上的，内存只有可怜的 512M，导致现在的经常性的 OOM，虽然我已经增加量 1G 的 swap 空间，仍然作用不大，贴一个日志让大家感受下：

    Jun 29 09:17:43 zlb kernel: [15847900.740029] mysqld invoked oom-killer: gfp_mask=0x201da, order=0, oom_score_adj=0
    Jun 29 09:17:44 zlb kernel: [15847900.742638] mysqld cpuset=/ mems_allowed=0
    Jun 29 09:17:44 zlb kernel: [15847900.742661] CPU: 0 PID: 30714 Comm: mysqld Not tainted 3.13.0-27-generic #50-Ubuntu
    Jun 29 09:17:44 zlb kernel: [15847900.742663] Hardware name: Bochs Bochs, BIOS Bochs 01/01/2011
    Jun 29 09:17:44 zlb kernel: [15847900.742665]  0000000000000000 ffff88001c9f7978 ffffffff817199c4 ffff880019ef5fc0
    Jun 29 09:17:44 zlb kernel: [15847900.742972]  ffff88001c9f7a00 ffffffff817142ff ffffffff81067876 ffff88001c9f79d8
    Jun 29 09:17:44 zlb kernel: [15847900.742975]  ffffffff810c74dc ffffffff811a1b16 ffff88001fffae28 ffffffff8104ea2e
    Jun 29 09:17:44 zlb kernel: [15847900.742977] Call Trace:
    Jun 29 09:17:44 zlb kernel: [15847900.743385]  [<ffffffff817199c4>] dump_stack+0x45/0x56
    Jun 29 09:17:44 zlb kernel: [15847900.743390]  [<ffffffff817142ff>] dump_header+0x7f/0x1f1
    Jun 29 09:17:44 zlb kernel: [15847900.743435]  [<ffffffff81067876>] ? put_online_cpus+0x56/0x80
    Jun 29 09:17:44 zlb kernel: [15847900.743448]  [<ffffffff810c74dc>] ? rcu_oom_notify+0xcc/0xf0
    Jun 29 09:17:44 zlb kernel: [15847900.743486]  [<ffffffff811a1b16>] ? kmem_cache_alloc_trace+0x1c6/0x1f0
    Jun 29 09:17:44 zlb kernel: [15847900.743503]  [<ffffffff8104ea2e>] ? kvm_async_pf_task_wake+0x8e/0x110
    Jun 29 09:17:44 zlb kernel: [15847900.743520]  [<ffffffff8115196e>] oom_kill_process+0x1ce/0x330
    Jun 29 09:17:44 zlb kernel: [15847900.743554]  [<ffffffff812d3395>] ? security_capable_noaudit+0x15/0x20
    Jun 29 09:17:44 zlb kernel: [15847900.743556]  [<ffffffff811520a4>] out_of_memory+0x414/0x450
    Jun 29 09:17:44 zlb kernel: [15847900.743560]  [<ffffffff81158377>] __alloc_pages_nodemask+0xa87/0xb20
    Jun 29 09:17:44 zlb kernel: [15847900.743562]  [<ffffffff81196333>] alloc_pages_current+0xa3/0x160
    Jun 29 09:17:44 zlb kernel: [15847900.743569]  [<ffffffff8114e567>] __page_cache_alloc+0x97/0xc0
    Jun 29 09:17:44 zlb kernel: [15847900.743572]  [<ffffffff8114ff75>] filemap_fault+0x185/0x410
    Jun 29 09:17:44 zlb kernel: [15847900.743578]  [<ffffffff8117485f>] __do_fault+0x6f/0x530
    Jun 29 09:17:44 zlb kernel: [15847900.743581]  [<ffffffff81178a02>] handle_mm_fault+0x492/0xf10
    Jun 29 09:17:44 zlb kernel: [15847900.743588]  [<ffffffff8109df24>] ? arch_vtime_task_switch+0x94/0xa0
    Jun 29 09:17:44 zlb kernel: [15847900.743590]  [<ffffffff8109df54>] ? vtime_common_task_switch+0x24/0x40
    Jun 29 09:17:44 zlb kernel: [15847900.743598]  [<ffffffff81097498>] ? finish_task_switch+0x128/0x170
    Jun 29 09:17:44 zlb kernel: [15847900.743607]  [<ffffffff81720202>] ? mutex_lock+0x12/0x2f
    Jun 29 09:17:44 zlb kernel: [15847900.743615]  [<ffffffff81725924>] __do_page_fault+0x184/0x560
    Jun 29 09:17:44 zlb kernel: [15847900.743638]  [<ffffffff8101b7d9>] ? sched_clock+0x9/0x10
    Jun 29 09:17:44 zlb kernel: [15847900.743641]  [<ffffffff8109d13d>] ? sched_clock_local+0x1d/0x80
    Jun 29 09:17:44 zlb kernel: [15847900.743658]  [<ffffffff811112ec>] ? acct_account_cputime+0x1c/0x20
    Jun 29 09:17:44 zlb kernel: [15847900.743661]  [<ffffffff8109d76b>] ? account_user_time+0x8b/0xa0
    Jun 29 09:17:44 zlb kernel: [15847900.743662]  [<ffffffff8109dd84>] ? vtime_account_user+0x54/0x60
    Jun 29 09:17:44 zlb kernel: [15847900.743665]  [<ffffffff81725d1a>] do_page_fault+0x1a/0x70
    Jun 29 09:17:44 zlb kernel: [15847900.743667]  [<ffffffff81725399>] do_async_page_fault+0x29/0xe0
    Jun 29 09:17:44 zlb kernel: [15847900.743670]  [<ffffffff817221b8>] async_page_fault+0x28/0x30
    Jun 29 09:17:44 zlb kernel: [15847900.743672] Mem-Info:
    Jun 29 09:17:44 zlb kernel: [15847900.743674] Node 0 DMA per-cpu:
    Jun 29 09:17:44 zlb kernel: [15847900.743954] CPU    0: hi:    0, btch:   1 usd:   0
    Jun 29 09:17:44 zlb kernel: [15847900.743957] Node 0 DMA32 per-cpu:
    Jun 29 09:17:44 zlb kernel: [15847900.743959] CPU    0: hi:  186, btch:  31 usd:   4
    Jun 29 09:17:44 zlb kernel: [15847900.743963] active_anon:46557 inactive_anon:46639 isolated_anon:0
    Jun 29 09:17:44 zlb kernel: [15847900.743963]  active_file:37 inactive_file:49 isolated_file:0
    Jun 29 09:17:44 zlb kernel: [15847900.743963]  unevictable:0 dirty:0 writeback:0 unstable:0
    Jun 29 09:17:44 zlb kernel: [15847900.743963]  free:1174 slab_reclaimable:2934 slab_unreclaimable:8396
    Jun 29 09:17:44 zlb kernel: [15847900.743963]  mapped:27514 shmem:28359 pagetables:12997 bounce:0
    Jun 29 09:17:44 zlb kernel: [15847900.743963]  free_cma:0
    Jun 29 09:17:44 zlb kernel: [15847900.743966] Node 0 DMA free:1964kB min:88kB low:108kB high:132kB active_anon:3176kB inactive_anon:3492kB active_file:8kB inactive_file:0kB unevictable:0kB isolated(anon):0kB isolated(file):0kB present:15992kB managed:15908kB mlocked:0kB dirty:0kB writeback:0kB mapped:1768kB shmem:1788kB slab_reclaimable:1132kB slab_unreclaimable:1936kB kernel_stack:1336kB pagetables:716kB unstable:0kB bounce:0kB free_cma:0kB writeback_tmp:0kB pages_scanned:13 all_unreclaimable? yes
    Jun 29 09:17:44 zlb kernel: [15847900.743975] lowmem_reserve[]: 0 471 471 471
    Jun 29 09:17:44 zlb kernel: [15847900.743979] Node 0 DMA32 free:2732kB min:2732kB low:3412kB high:4096kB active_anon:183052kB inactive_anon:183064kB active_file:140kB inactive_file:196kB unevictable:0kB isolated(anon):0kB isolated(file):0kB present:507896kB managed:485940kB mlocked:0kB dirty:0kB writeback:0kB mapped:108288kB shmem:111648kB slab_reclaimable:10604kB slab_unreclaimable:31648kB kernel_stack:3560kB pagetables:51272kB unstable:0kB bounce:0kB free_cma:0kB writeback_tmp:0kB pages_scanned:535 all_unreclaimable? yes
    Jun 29 09:17:44 zlb kernel: [15847900.743997] lowmem_reserve[]: 0 0 0 0
    Jun 29 09:17:44 zlb kernel: [15847900.743999] Node 0 DMA: 33*4kB (E) 191*8kB (UEMR) 17*16kB (MR) 1*32kB (R) 0*64kB 0*128kB 0*256kB 0*512kB 0*1024kB 0*2048kB 0*4096kB = 1964kB
    Jun 29 09:17:44 zlb kernel: [15847900.744036] Node 0 DMA32: 181*4kB (UEM) 134*8kB (UEMR) 13*16kB (EM) 3*32kB (R) 2*64kB (R) 4*128kB (R) 0*256kB 0*512kB 0*1024kB 0*2048kB 0*4096kB = 2740kB
    Jun 29 09:17:44 zlb kernel: [15847900.744047] Node 0 hugepages_total=0 hugepages_free=0 hugepages_surp=0 hugepages_size=2048kB
    Jun 29 09:17:44 zlb kernel: [15847900.744048] 47613 total pagecache pages
    Jun 29 09:17:44 zlb kernel: [15847900.744050] 19172 pages in swap cache
    Jun 29 09:17:44 zlb kernel: [15847900.744053] Swap cache stats: add 3213543, delete 3194371, find 3380499/3672513
    Jun 29 09:17:44 zlb kernel: [15847900.744054] Free swap  = 0kB
    Jun 29 09:17:44 zlb kernel: [15847900.744055] Total swap = 1023996kB
    Jun 29 09:17:44 zlb kernel: [15847900.744056] 130972 pages RAM
    Jun 29 09:17:44 zlb kernel: [15847900.744057] 0 pages HighMem/MovableOnly
    Jun 29 09:17:44 zlb kernel: [15847900.744058] 5489 pages reserved
    Jun 29 09:17:44 zlb kernel: [15847900.744059] [ pid ]   uid  tgid total_vm      rss nr_ptes swapents oom_score_adj name
    Jun 29 09:17:44 zlb kernel: [15847900.744064] [  343]     0   343     4902        0      13       80             0 upstart-udev-br
    Jun 29 09:17:44 zlb kernel: [15847900.744067] [  352]     0   352    12803        0      27      134         -1000 systemd-udevd
    Jun 29 09:17:44 zlb kernel: [15847900.744069] [  365]   102   365     9807        0      24      100             0 dbus-daemon
    Jun 29 09:17:44 zlb kernel: [15847900.744071] [  422]     0   422    10863        1      25       93             0 systemd-logind
    Jun 29 09:17:44 zlb kernel: [15847900.744072] [  444]     0   444     3852        0      11       82             0 upstart-file-br
    Jun 29 09:17:44 zlb kernel: [15847900.744075] [  779]     0   779     3914        0      12      141             0 upstart-socket-
    Jun 29 09:17:44 zlb kernel: [15847900.744076] [  849]     0   849     3955        1      12       41             0 getty
    Jun 29 09:17:44 zlb kernel: [15847900.744078] [  853]     0   853     3955        1      13       41             0 getty
    Jun 29 09:17:44 zlb kernel: [15847900.744080] [  861]     0   861     3955        1      13       39             0 getty
    Jun 29 09:17:44 zlb kernel: [15847900.744081] [  862]     0   862     3955        1      12       39             0 getty
    Jun 29 09:17:44 zlb kernel: [15847900.744083] [  867]     0   867     3955        1      13       41             0 getty
    Jun 29 09:17:44 zlb kernel: [15847900.744085] [  901]     0   901     3786        0      11       69             0 starter
    Jun 29 09:17:44 zlb kernel: [15847900.744087] [  904]     0   904   188212        0      75      606             0 charon
    Jun 29 09:17:44 zlb kernel: [15847900.744089] [  909]     0   909    15341        0      34      173         -1000 sshd
    Jun 29 09:17:44 zlb kernel: [15847900.744090] [  914]     0   914     1092        0       8       36             0 acpid
    Jun 29 09:17:44 zlb kernel: [15847900.744092] [  915]     0   915     5914        1      17       62             0 cron
    Jun 29 09:17:44 zlb kernel: [15847900.744094] [  916]     0   916     4785        0      13       42             0 atd
    Jun 29 09:17:44 zlb kernel: [15847900.744096] [  933]   105   933    83921        0      62      324             0 whoopsie
    Jun 29 09:17:44 zlb kernel: [15847900.744098] [ 1012]   109  1012     1874        1       9       35             0 epmd
    Jun 29 09:17:44 zlb kernel: [15847900.744100] [ 1208]     0  1208     6336       13      16       64             0 master
    Jun 29 09:17:44 zlb kernel: [15847900.744101] [ 1212]   107  1212     6893        0      18       85             0 qmgr
    Jun 29 09:17:44 zlb kernel: [15847900.744103] [ 1226]     0  1226     2671        0      10       43             0 pptpd
    Jun 29 09:17:44 zlb kernel: [15847900.744105] [ 1392]     0  1392     3955        1      13       40             0 getty
    Jun 29 09:17:44 zlb kernel: [15847900.744107] [20601]     0 20601    26260      162      53      939             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744109] [20903]   110 20903    13652        1      28      345             0 freshclam
    Jun 29 09:17:44 zlb kernel: [15847900.744111] [ 2653]   101  2653    64154        0      39     6835             0 rsyslogd
    Jun 29 09:17:44 zlb kernel: [15847900.744113] [ 9233]   109  9233   155041        0      63     9920             0 beam
    Jun 29 09:17:44 zlb kernel: [15847900.744114] [ 9459]   109  9459     1865        0       9       37             0 inet_gethost
    Jun 29 09:17:44 zlb kernel: [15847900.744116] [ 9460]   109  9460     3446        1      12       39             0 inet_gethost
    Jun 29 09:17:44 zlb kernel: [15847900.744118] [30310]     0 30310     6293       28      16      301             0 tmux
    Jun 29 09:17:44 zlb kernel: [15847900.744119] [30311]     0 30311     5704        2      16      562             0 bash
    Jun 29 09:17:44 zlb kernel: [15847900.744130] [30356]     0 30356     1111        1       7       39             0 mysqld_safe
    Jun 29 09:17:44 zlb kernel: [15847900.744133] [30705]   106 30705   223086      731      89    15800             0 mysqld
    Jun 29 09:17:44 zlb kernel: [15847900.744135] [12526]    33 12526    26222      152      51      975             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744137] [12529]    33 12529    30512      149      53     1093             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744139] [12530]    33 12530    30500      126      53     1103             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744141] [12540]    33 12540    30492      125      53     1105             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744142] [12569]    33 12569    30497      313      53      921             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744144] [12572]    33 12572    89683     1048     130     4350             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744146] [12697]    33 12697    89041     1644     122     2873             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744148] [12785]    33 12785    30488      149      53     1076             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744150] [12791]    33 12791    30488      143      53     1082             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744152] [12800]    33 12800    30472      151      53     1056             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744154] [12806]    33 12806    30476      190      53     1023             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744155] [12813]    33 12813    30488      129      53     1096             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744157] [12818]   107 12818     6852        0      17       69             0 pickup
    Jun 29 09:17:44 zlb kernel: [15847900.744159] [12819]    33 12819    30472      151      53     1058             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744161] [12825]    33 12825    30476      146      53     1065             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744163] [12828]    33 12828    30476      225      53      986             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744164] [12829]    33 12829    30488      147      53     1076             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744166] [12835]    33 12835    87146     1278     116     2639             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744168] [12837]    33 12837    88395      508     122     3788             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744169] [12839]    33 12839    30488      171      53     1054             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744171] [12842]    33 12842    30476      140      53     1069             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744173] [12848]    33 12848    30476      140      53     1069             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744175] [12851]    33 12851    30488      133      53     1085             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744176] [12854]    33 12854    30476      141      53     1068             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744178] [12855]    33 12855    30488      257      53      979             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744180] [12860]    33 12860    30482      151      53     1068             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744182] [12861]    33 12861    30476      140      53     1069             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744183] [12862]    33 12862    30488      152      53     1073             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744185] [12865]    33 12865    30476      146      53     1062             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744187] [12872]    33 12872    30482      140      53     1077             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744188] [12873]    33 12873    30476      140      53     1069             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744190] [12875]    33 12875    30476      264      53      947             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744192] [12876]    33 12876    30476      152      53     1057             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744194] [12883]    33 12883    30482      152      53     1067             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744195] [12885]    33 12885    30488      200      53     1023             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744197] [12886]    33 12886    30482      152      53     1064             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744199] [12897]    33 12897    30472      151      53     1057             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744200] [12898]    33 12898    30488      200      53     1032             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744202] [12899]    33 12899    30488      152      53     1072             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744204] [12900]    33 12900    30472      293      53      913             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744205] [12901]    33 12901    30490      140      53     1083             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744207] [12904]    33 12904    30476      140      53     1069             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744209] [12908]    33 12908    30485      136      53     1081             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744225] [12914]    33 12914    30488      234      53      990             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744228] [12919]    33 12919    30476      122      53     1087             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744230] [12921]    33 12921    30482      128      53     1091             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744232] [12925]    33 12925    30466      140      53     1063             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744234] [12926]    33 12926    30472      152      53     1057             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744235] [12933]    33 12933    30488      227      53      995             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744237] [12936]    33 12936    30488      152      53     1073             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744239] [12937]    33 12937    30485      152      53     1066             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744240] [12938]    33 12938    30482      140      53     1077             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744242] [12945]    33 12945    30485      255      53      962             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744246] [12948]    33 12948    30476      233      53      976             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744248] [12949]    33 12949    30470      140      53     1065             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744250] [12954]    33 12954    30488      127      53     1093             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744251] [12955]    33 12955    30488      130      53     1093             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744253] [12958]    33 12958    30466      274      53      929             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744255] [12959]    33 12959    30482      152      53     1067             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744257] [12969]    33 12969    30488      261      53      973             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744258] [12970]    33 12970    30482      152      53     1065             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744260] [12971]    33 12971    30488      137      53     1085             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744262] [12972]    33 12972    30488      130      53     1088             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744263] [12977]    33 12977    30488      139      53     1083             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744265] [12984]    33 12984    30488      140      53     1082             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744267] [12987]    33 12987    30488      123      53     1099             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744269] [12988]    33 12988    30485      138      53     1079             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744270] [12995]    33 12995    30482      248      53      973             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744272] [12996]    33 12996    30488      151      53     1071             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744274] [12998]    33 12998    30488      213      53     1010             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744275] [13001]    33 13001    30488      152      53     1070             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744277] [13006]    33 13006    30482      152      53     1065             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744279] [13012]    33 13012    30488      152      53     1070             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744280] [13013]    33 13013    30488      152      53     1070             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744282] [13014]    33 13014    30488      133      53     1090             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744284] [13026]    33 13026    30488      228      53      997             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744286] [13027]    33 13027    30488      152      53     1070             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744287] [13031]    33 13031    30488      138      53     1083             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744289] [13033]    33 13033    30482      152      53     1065             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744291] [13041]    33 13041    30482      150      53     1068             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744293] [13042]    33 13042    30482      139      53     1078             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744294] [13043]    33 13043    30482      140      53     1077             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744296] [13044]    33 13044    30488      204      53     1032             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744298] [13045]    33 13045    30488      133      53     1089             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744299] [13049]    33 13049    30482      308      53      911             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744301] [13060]    33 13060    30488      145      53     1077             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744303] [13063]    33 13063    30488      135      53     1088             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744312] [13066]    33 13066    30488      318      53      906             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744315] [13069]    33 13069    30592      320      53      917             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744317] [13075]    33 13075    30488      152      53     1070             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744318] [13078]    33 13078    30488      151      53     1071             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744320] [13081]    33 13081    30488      316      53      906             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744322] [13086]    33 13086    30482      140      53     1077             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744323] [13087]    33 13087    30482      138      53     1079             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744325] [13090]    33 13090    30488      131      53     1092             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744327] [13091]    33 13091    30488      136      53     1088             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744329] [13096]    33 13096    30488      133      53     1090             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744330] [13097]    33 13097    30485      152      53     1065             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744332] [13102]    33 13102    30482      138      53     1079             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744334] [13104]    33 13104    30482      134      53     1083             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744335] [13108]    33 13108    30482      140      53     1077             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744337] [13113]    33 13113    30488      152      53     1072             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744339] [13117]    33 13117    30488      133      53     1089             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744340] [13122]    33 13122    30482      152      53     1067             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744342] [13124]    33 13124    30482      137      53     1080             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744344] [13126]    33 13126    30488      313      53      912             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744346] [13127]    33 13127    30488      191      53     1032             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744347] [13132]    33 13132    30472      139      53     1067             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744349] [13137]    33 13137    86636     1646     114     2199             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744351] [13139]    33 13139    30485      152      53     1069             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744352] [13140]    33 13140    30488      126      53     1098             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744354] [13141]    33 13141    30488      134      53     1083             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744356] [13147]    33 13147    30482      287      53      933             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744357] [13151]    33 13151    30476      151      53     1058             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744359] [13153]    33 13153    30488      229      53      993             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744361] [13154]    33 13154    30488      134      53     1083             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744362] [13163]    33 13163    30482      151      53     1066             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744364] [13169]    33 13169    30488      212      53     1012             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744366] [13170]    33 13170    30488      134      53     1090             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744368] [13175]    33 13175    30476      139      53     1070             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744369] [13180]    33 13180    30485      154      53     1063             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744371] [13185]    33 13185    30470      152      53     1051             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744373] [13186]    33 13186    30485      160      53     1065             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744374] [13187]    33 13187    86139     1513     111     1828             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744376] [13194]    33 13194    84169     1654     105     1337             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744378] [13195]    33 13195    84681     1116     104     1271             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744380] [13196]    33 13196    30485      151      53     1066             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744381] [13198]    33 13198    84169     1821     107     1324             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744383] [13199]    33 13199    30470      152      53     1051             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744385] [13202]    33 13202    86121     1434     110     1848             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744386] [13205]    33 13205    30476      152      53     1057             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744388] [13208]    33 13208    84153     1643     105     1355             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744398] [13209]    33 13209    30482      130      53     1087             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744401] [13212]    33 13212    30439      206      53      975             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744402] [13214]    33 13214    84582     1571     105     1470             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744404] [13216]    33 13216    30485      246      53      973             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744406] [13218]    33 13218    84556     1643     104     1345             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744407] [13220]    33 13220    84390     1436     103     1477             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744409] [13226]    33 13226    84454     1371     106     1543             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744411] [13227]    33 13227    84305     1147     104     1431             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744412] [13228]    33 13228    30482      159      53     1059             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744414] [13230]    33 13230    30482      148      53     1069             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744416] [13232]    33 13232    30482      140      53     1077             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744417] [13233]    33 13233    30482      140      53     1077             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744419] [13234]    33 13234    30482      139      53     1078             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744421] [13235]    33 13235    30482      137      53     1080             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744423] [13251]    33 13251    84582     2072     108     1482             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744425] [13257]    33 13257    30476      152      53     1057             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744426] [13259]    33 13259    30485      140      53     1078             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744428] [13263]    33 13263    30488      144      53     1074             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744430] [13264]    33 13264    30488      133      53     1080             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744431] [13270]    33 13270    30485      152      53     1065             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744433] [13271]    33 13271    30488      139      53     1085             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744435] [13272]    33 13272    30485      152      53     1065             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744437] [13280]    33 13280    84262     1302     106     1593             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744438] [13289]    33 13289    84043     1516     106     1247             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744440] [13291]    33 13291    84009     1500     104     1327             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744442] [13292]    33 13292    84073     1571     103     1228             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744444] [13294]    33 13294    84017     1588     103     1382             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744445] [13296]    33 13296    83951     1448     101     1232             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744447] [13300]    33 13300    84015     1593     102     1367             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744449] [13303]    33 13303    83816     1268     101     1158             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744450] [13305]    33 13305    83823     1289     102     1185             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744452] [13307]    33 13307    83817     1486     100     1113             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744454] [13309]    33 13309    83970     1752     102     1354             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744456] [13315]    33 13315    83624     1634     101      820             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744457] [13318]    33 13318    83496     1296     102      934             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744459] [13320]    33 13320    83815     1537     101      942             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744461] [13322]    33 13322    83495     1273      99      920             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744462] [13324]    33 13324    83819     1557     100      944             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744464] [13326]    33 13326    83408     1314      98      880             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744466] [13329]    33 13329    83460     1295      98      895             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744468] [13332]    33 13332    83420     1296      99      837             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744469] [13333]    33 13333    83340      989      95      844             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744471] [13334]    33 13334    83163      708      99      837             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744473] [13336]    33 13336    83275     2259      99      826             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744482] [13338]    33 13338    83276     2508      95      516             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744484] [13341]    33 13341    83196     2415      96      338             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744486] [13342]    33 13342    83196     2392      95      378             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744488] [13343]    33 13343    49053      775      89      252             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744490] [13346]    33 13346    46740      599      88      149             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744491] [13348]    33 13348    46806      752      83       69             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744493] [13351]    33 13351    46806      711      89       68             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744495] [13352]    33 13352    30150      324      58       48             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744497] [13354]    33 13354    12318      288      28        7             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744498] [13358]    33 13358    27220      344      52       11             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744500] [13360]    33 13360    11586      134      24        0             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744502] [13363]    33 13363    11236       38      18        0             0 php-cgi
    Jun 29 09:17:44 zlb kernel: [15847900.744504] [13365]    33 13365    26222      239      48      881             0 apache2
    Jun 29 09:17:44 zlb kernel: [15847900.744512] Out of memory: Kill process 30705 (mysqld) score 43 or sacrifice child

可以看到有无数的 apache2 和 php-cgi 进程，占用的大量内存，最后导致 mysqld 被杀死，页面访问不了。

而这种情况也不是一次两次了，实在忍受不了，所以打算换成静态 Blog，把 apache 也一并换为 nginx。希望可以降低一些内存占用。
