# Управление процессами

## Цели:
1. Создать скрипт, который получает информацию о процессах через файловую систему /proc.
2. Реализовать вывод не менее следующих полей: PID, PPID, состояние процесса, имя или команда запуска.
3. Проверить работу скрипта на запущенной системе.

Скрипт:

```
#!/bin/bash

printf "%-8s %-13s %-15s %s\n" "PID" "PPID" "STATE" "COMMAND/NAME"
printf "%-8s %-13s %-15s %s\n" "------" "--------"  "----------------" "------------------------------"

for i in /proc/[0-9]*; do
   info=$(cat $i/status)
   pid=$(cat $i/status | grep -Pi "^PID" | awk '{print $2}')
   ppid=$(cat $i/status | grep -Pi "PPID" | awk '{print $2}')
   state=$(cat $i/status | grep -Pi "state" | awk -F: '{print $2}')
   cmd=$(tr -d '\0' < "$i/cmdline" 2>/dev/null)
  # echo $cmd
   if [ -z "$cmd" ]; then
      cmd=$(cat  $i/status | grep -Pi "name" | awk -F: '{print $2}')
   fi
   #name=$(cat  $i/status | grep -Pi "name" | awk -F: '{print $2}')
   #echo "$pid $ppid $state $name"
   printf "%-8s %-13s %-15s %s\n" "$pid" "$ppid" "$state" "$cmd"
done
```

Результат выполнения скрипта:

```
mirana@debian-lvm:~$ sudo bash ps_script2.sh
PID      PPID          STATE           COMMAND/NAME
------   --------      ---------------- ------------------------------
1        0              S (sleeping)   /sbin/init
100      2              I (idle)        kworker/R-mld
101      2              I (idle)        kworker/1
102      2              I (idle)        kworker/R-ipv6_addrconf
1026     2              I (idle)        kworker/R-cfg80211
103      2              I (idle)        kworker/u512
1033     1              S (sleeping)   sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
108      2              I (idle)        kworker/R-kstrp
1083     2              I (idle)        kworker/R-rpciod
1084     2              I (idle)        kworker/R-xprtiod
1088     1              S (sleeping)   /usr/sbin/cron-f
1090     1              S (sleeping)   /usr/sbin/blkmapd
1099     2              I (idle)        kworker/1
11       2              I (idle)        kworker/0
110      2              I (idle)        kworker/u515
110563   2              I (idle)        kworker/0
111      2              I (idle)        kworker/u516
1116     911            S (sleeping)   dhcpcd: [BPF ARP] ens33 192.168.88.111
1117     1              S (sleeping)   nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
1118     1117           S (sleeping)   nginx: worker process
1119     1117           S (sleeping)   nginx: worker process
112      2              I (idle)        kworker/u517
1125     911            S (sleeping)   dhcpcd: [DHCP6 proxy] fe80::36c2:9d55:6c24:42e9
1143     911            S (sleeping)   dhcpcd: [BOOTP proxy] 192.168.88.111
12       2              I (idle)        kworker/u512
1228     1033           S (sleeping)   sshd-session: mirana [priv]
1234     1              S (sleeping)   /usr/lib/systemd/systemd--user
1235     2              S (sleeping)    psimon
1237     1234           S (sleeping)   (sd-pam)
1250     1228           S (sleeping)   sshd-session: mirana@pts/0
1251     1250           S (sleeping)   -bash
127435   2              I (idle)        kworker/0
13       2              I (idle)        kworker/R-mm_percpu_wq
14       2              I (idle)        rcu_tasks_kthread
140065   1251           S (sleeping)   sudobashps_script2.sh
140067   140065         S (sleeping)   sudobashps_script2.sh
140068   140067         S (sleeping)   bashps_script2.sh
15       2              I (idle)        rcu_tasks_rude_kthread
16       2              I (idle)        rcu_tasks_trace_kthread
17       2              S (sleeping)    ksoftirqd/0
18       2              I (idle)        rcu_preempt
19       2              S (sleeping)    rcu_exp_par_gp_kthread_worker/1
2        0              S (sleeping)    kthreadd
20       2              S (sleeping)    rcu_exp_gp_kthread_worker
21       2              S (sleeping)    migration/0
22       2              S (sleeping)    idle_inject/0
223      2              I (idle)        kworker/R-ata_sff
225      2              S (sleeping)    scsi_eh_0
226      2              I (idle)        kworker/R-scsi_tmf_0
227      2              S (sleeping)    scsi_eh_1
228      2              I (idle)        kworker/R-scsi_tmf_1
23       2              S (sleeping)    cpuhp/0
232      2              I (idle)        kworker/R-mpt_poll_0
233      2              I (idle)        kworker/R-mpt/0
234      2              S (sleeping)    scsi_eh_2
235      2              I (idle)        kworker/R-scsi_tmf_2
236      2              S (sleeping)    scsi_eh_3
237      2              I (idle)        kworker/R-scsi_tmf_3
238      2              S (sleeping)    scsi_eh_4
239      2              I (idle)        kworker/R-scsi_tmf_4
24       2              S (sleeping)    cpuhp/1
240      2              S (sleeping)    scsi_eh_5
241      2              I (idle)        kworker/R-scsi_tmf_5
242      2              S (sleeping)    scsi_eh_6
243      2              I (idle)        kworker/R-scsi_tmf_6
244      2              S (sleeping)    scsi_eh_7
245      2              I (idle)        kworker/R-scsi_tmf_7
246      2              S (sleeping)    scsi_eh_8
247      2              I (idle)        kworker/R-scsi_tmf_8
248      2              S (sleeping)    scsi_eh_9
249      2              I (idle)        kworker/R-scsi_tmf_9
25       2              S (sleeping)    idle_inject/1
250      2              S (sleeping)    scsi_eh_10
251      2              I (idle)        kworker/R-scsi_tmf_10
252      2              S (sleeping)    scsi_eh_11
253      2              I (idle)        kworker/R-scsi_tmf_11
254      2              S (sleeping)    scsi_eh_12
255      2              I (idle)        kworker/R-scsi_tmf_12
256      2              S (sleeping)    scsi_eh_13
257      2              I (idle)        kworker/R-scsi_tmf_13
258      2              S (sleeping)    scsi_eh_14
259      2              I (idle)        kworker/R-scsi_tmf_14
26       2              S (sleeping)    migration/1
260      2              S (sleeping)    scsi_eh_15
261      2              I (idle)        kworker/R-scsi_tmf_15
262      2              S (sleeping)    scsi_eh_16
263      2              I (idle)        kworker/R-scsi_tmf_16
264      2              S (sleeping)    scsi_eh_17
265      2              I (idle)        kworker/R-scsi_tmf_17
266      2              S (sleeping)    scsi_eh_18
267      2              I (idle)        kworker/R-scsi_tmf_18
268      2              S (sleeping)    scsi_eh_19
269      2              I (idle)        kworker/R-scsi_tmf_19
27       2              S (sleeping)    ksoftirqd/1
270      2              S (sleeping)    scsi_eh_20
271      2              I (idle)        kworker/R-scsi_tmf_20
272      2              S (sleeping)    scsi_eh_21
273      2              I (idle)        kworker/R-scsi_tmf_21
274      2              S (sleeping)    scsi_eh_22
275      2              I (idle)        kworker/R-scsi_tmf_22
276      2              S (sleeping)    scsi_eh_23
277      2              I (idle)        kworker/R-scsi_tmf_23
278      2              S (sleeping)    scsi_eh_24
279      2              I (idle)        kworker/R-scsi_tmf_24
280      2              S (sleeping)    scsi_eh_25
281      2              I (idle)        kworker/R-scsi_tmf_25
282      2              S (sleeping)    scsi_eh_26
283      2              I (idle)        kworker/R-scsi_tmf_26
284      2              S (sleeping)    scsi_eh_27
285      2              I (idle)        kworker/R-scsi_tmf_27
286      2              S (sleeping)    scsi_eh_28
287      2              I (idle)        kworker/R-scsi_tmf_28
288      2              S (sleeping)    scsi_eh_29
289      2              I (idle)        kworker/R-scsi_tmf_29
290      2              S (sleeping)    scsi_eh_30
291      2              I (idle)        kworker/R-scsi_tmf_30
292      2              S (sleeping)    scsi_eh_31
293      2              I (idle)        kworker/R-scsi_tmf_31
3        2              S (sleeping)    pool_workqueue_release
315      2              I (idle)        kworker/u514
318      2              I (idle)        kworker/u514
322      2              S (sleeping)    scsi_eh_32
323      2              I (idle)        kworker/R-scsi_tmf_32
34       2              S (sleeping)    kdevtmpfs
340      2              I (idle)        kworker/R-kdmflush/254
341      2              I (idle)        kworker/R-kdmflush/254
348      2              I (idle)        kworker/R-md
349      2              I (idle)        kworker/R-md_bitmap
35       2              I (idle)        kworker/R-inet_frag_wq
350      2              I (idle)        kworker/R-raid5wq
351      2              I (idle)        kworker/R-kdmflush/254
352      2              I (idle)        kworker/R-kdmflush/254
354      2              I (idle)        kworker/R-kdmflush/254
355      2              I (idle)        kworker/R-kdmflush/254
356      2              I (idle)        kworker/R-kdmflush/254
36       2              S (sleeping)    kauditd
369      2              S (sleeping)    mdX_raid1
37       2              S (sleeping)    khungtaskd
38       2              S (sleeping)    oom_reaper
4        2              I (idle)        kworker/R-kvfree_rcu_reclaim
40       2              I (idle)        kworker/R-writeback
405      2              S (sleeping)    jbd2/dm-0-8
406      2              I (idle)        kworker/R-ext4-rsv-conversion
41160    2              I (idle)        kworker/u514
42       2              S (sleeping)    kcompactd0
43       2              S (sleeping)    ksmd
439      2              S (sleeping)    irq/57-vmw_vmci
44       2              S (sleeping)    khugepaged
440      2              S (sleeping)    irq/58-vmw_vmci
441      2              S (sleeping)    irq/59-vmw_vmci
45       2              I (idle)        kworker/R-kintegrityd
46       2              I (idle)        kworker/R-kblockd
463      1              S (sleeping)   /usr/lib/systemd/systemd-journald
47       2              I (idle)        kworker/R-blkcg_punt_bio
4774     1              S (sleeping)   /sbin/agetty-o-- \u--noreset--noclear-linux
48       2              S (sleeping)    irq/9-acpi
485      1              S (sleeping)   /usr/sbin/dmeventd-f
492      1              S (sleeping)   /usr/lib/systemd/systemd-udevd
493      2              S (sleeping)    psimon
5        2              I (idle)        kworker/R-rcu_gp
50       2              I (idle)        kworker/R-tpm_dev_wq
51       2              I (idle)        kworker/R-edac-poller
52       2              I (idle)        kworker/R-devfreq_wq
53       2              I (idle)        kworker/R-quota_events_unbound
54       2              I (idle)        kworker/0
55       2              S (sleeping)    kswapd0
56918    2              I (idle)        kworker/u513
575      2              S (sleeping)    irq/16-vmwgfx
576      2              I (idle)        kworker/R-ttm
6        2              I (idle)        kworker/R-sync_wq
62       2              I (idle)        kworker/R-kthrotld
655      2              I (idle)        kworker/R-cryptd
66       2              S (sleeping)    irq/24-pciehp
67       2              S (sleeping)    irq/25-pciehp
68       2              S (sleeping)    irq/26-pciehp
68639    2              I (idle)        kworker/1
68640    2              I (idle)        kworker/1
69       2              S (sleeping)    irq/27-pciehp
7        2              I (idle)        kworker/R-slub_flushwq
70       2              S (sleeping)    irq/28-pciehp
71       2              S (sleeping)    irq/29-pciehp
72       2              S (sleeping)    irq/30-pciehp
73       2              S (sleeping)    irq/31-pciehp
736      2              S (sleeping)    jbd2/dm-1-8
737      2              S (sleeping)    jbd2/sda1-8
738      2              I (idle)        kworker/R-ext4-rsv-conversion
739      2              I (idle)        kworker/R-ext4-rsv-conversion
74       2              S (sleeping)    irq/32-pciehp
740      2              S (sleeping)    jbd2/dm-6-8
742      2              I (idle)        kworker/R-ext4-rsv-conversion
746      1              S (sleeping)   /usr/lib/systemd/systemd-timesyncd
75       2              S (sleeping)    irq/33-pciehp
76       2              S (sleeping)    irq/34-pciehp
76747    2              I (idle)        kworker/0
77       2              S (sleeping)    irq/35-pciehp
773      1              S (sleeping)   /usr/sbin/rpcbind-f-w
78       2              S (sleeping)    irq/36-pciehp
79       2              S (sleeping)    irq/37-pciehp
8        2              I (idle)        kworker/R-netns
80       2              S (sleeping)    irq/38-pciehp
80188    2              I (idle)        kworker/u513
81       2              S (sleeping)    irq/39-pciehp
82       2              S (sleeping)    irq/40-pciehp
83       2              S (sleeping)    irq/41-pciehp
84       2              S (sleeping)    irq/42-pciehp
85       2              S (sleeping)    irq/43-pciehp
86       2              S (sleeping)    irq/44-pciehp
87       2              S (sleeping)    irq/45-pciehp
876      1              S (sleeping)   /usr/bin/dbus-daemon--system--address=systemd:--nofork--nopidfile--systemd-activation--syslog-only
88       2              S (sleeping)    irq/46-pciehp
88707    2              I (idle)        kworker/u513
888      1              S (sleeping)   /usr/lib/systemd/systemd-logind
89       2              S (sleeping)    irq/47-pciehp
894      1              S (sleeping)   /usr/bin/VGAuthService
898      1              S (sleeping)   /usr/bin/vmtoolsd
90       2              S (sleeping)    irq/48-pciehp
909      1              S (sleeping)   dhcpcd: ens33 [ip4] [ip6]
91       2              S (sleeping)    irq/49-pciehp
911      909            S (sleeping)   dhcpcd: [privileged proxy] ens33 [ip4] [ip6]
912      909            S (sleeping)   dhcpcd: [network proxy] ens33 [ip4] [ip6]
913      909            S (sleeping)   dhcpcd: [control proxy] ens33 [ip4] [ip6]
92       2              S (sleeping)    irq/50-pciehp
93       2              S (sleeping)    irq/51-pciehp
94       2              S (sleeping)    irq/52-pciehp
95       2              S (sleeping)    irq/53-pciehp
96       2              S (sleeping)    irq/54-pciehp
97       2              S (sleeping)    irq/55-pciehp
98       2              I (idle)        kworker/R-acpi_thermal_pm
mirana@debian-lvm:~$
```

