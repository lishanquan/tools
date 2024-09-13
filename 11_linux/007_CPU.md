在Linux系统中，查看CPU使用情况有多种方法，以下是几个常见的命令和工具：

1. ### 使用`lscpu`命令：

   ```bash
   [root@lsq ~]# lscpu
   Architecture:          x86_64	#CPU架构
   CPU op-mode(s):        32-bit, 64-bit	#操作模式
   Byte Order:            Little Endian
   CPU(s):                16	#CPU核心数
   On-line CPU(s) list:   0-15
   Thread(s) per core:    2
   Core(s) per socket:    8
   Socket(s):             1	#线程数
   NUMA node(s):          1
   Vendor ID:             GenuineIntel
   CPU family:            6
   Model:                 85
   Model name:            Intel(R) Xeon(R) Platinum 8269CY CPU @ 2.50GHz
   Stepping:              7
   CPU MHz:               2500.002
   BogoMIPS:              5000.00
   Hypervisor vendor:     KVM
   Virtualization type:   full		#虚拟化类型：硬件辅助的全虚拟化
   L1d cache:             32K
   L1i cache:             32K
   L2 cache:              1024K
   L3 cache:              36608K
   NUMA node0 CPU(s):     0-15
   Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc eagerfpu pni pclmulqdq monitor ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx avx512f avx512dq rdseed adx smap clflushopt clwb avx512cd avx512bw avx512vl xsaveopt xsavec xgetbv1 arat avx512_vnni
   ```

   

2. ### 使用`top`命令：

   - ##### 实时查看系统整体CPU使用情况以及各个进程的CPU使用百分比：

     ```bash
     [root@lsq ~]# top
     top - 09:37:23 up 310 days, 16:44,  1 user,  load average: 0.09, 0.18, 0.30
     Tasks: 240 total,   1 running, 239 sleeping,   0 stopped,   0 zombie
     %Cpu(s):  0.0 us,  0.4 sy,  0.0 ni, 99.6 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
     KiB Mem : 31728828 total,  5356772 free, 22660304 used,  3711752 buff/cache
     KiB Swap:        0 total,        0 free,        0 used.  6959832 avail Mem 
     
       PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND       
         1 root      20   0  126380   3888   1576 S   0.0  0.0   4:13.58 systemd       
         2 root      20   0       0      0      0 S   0.0  0.0   0:00.42 kthreadd       
         3 root      20   0       0      0      0 S   0.0  0.0  38:33.92 ksoftirqd/0   
         5 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 kworker/0:0H   
         7 root      rt   0       0      0      0 S   0.0  0.0   0:06.34 migration/0
     ```

   

   - ##### 在 `top` 界面中，按下 `1` 键可以单独查看每个CPU核心的负载

     ```bash
     [root@lsq ~]# top
     top - 09:39:26 up 310 days, 16:46,  1 user,  load average: 0.05, 0.14, 0.28
     Tasks: 240 total,   1 running, 239 sleeping,   0 stopped,   0 zombie
     %Cpu0  :  1.0 us,  0.7 sy,  0.0 ni, 98.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
     %Cpu1  :  0.3 us,  0.0 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
     %Cpu2  :  0.7 us,  1.0 sy,  0.0 ni, 98.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
     %Cpu3  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
     %Cpu4  :  1.0 us,  1.0 sy,  0.0 ni, 97.7 id,  0.0 wa,  0.0 hi,  0.3 si,  0.0 st
     %Cpu5  :  0.3 us,  0.0 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
     %Cpu6  :  1.3 us,  0.7 sy,  0.0 ni, 98.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
     %Cpu7  :  0.7 us,  0.0 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
     %Cpu8  :  1.0 us,  0.7 sy,  0.0 ni, 98.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
     %Cpu9  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
     %Cpu10 :  1.0 us,  0.7 sy,  0.0 ni, 98.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
     %Cpu11 :  0.3 us,  0.3 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
     %Cpu12 :  1.0 us,  1.0 sy,  0.0 ni, 98.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
     %Cpu13 :  0.3 us,  0.3 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
     %Cpu14 :  1.3 us,  1.3 sy,  0.0 ni, 97.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
     %Cpu15 :  1.3 us,  2.0 sy,  0.0 ni, 96.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
     KiB Mem : 31728828 total,  5405560 free, 22609948 used,  3713320 buff/cache
     KiB Swap:        0 total,        0 free,        0 used.  7010696 avail Mem
     ```

     

3. ###  `mpstat` 命令

   - ##### 查看每个CPU核心的使用率和统计信息：

     ```bash
     [root@lsq ~]# mpstat -P ALL 
     Linux 3.10.0-957.21.3.el7.x86_64 (iZwz90d17v5vn63qib5zlvZ) 	09/13/2024 	_x86_64_	(16 CPU)
     
     09:46:28 AM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
     09:46:28 AM  all    0.74    0.00    0.59    0.01    0.00    0.05    0.00    0.00    0.00   98.61
     09:46:28 AM    0    1.12    0.00    0.89    0.00    0.00    0.08    0.00    0.00    0.00   97.90
     09:46:28 AM    1    0.40    0.00    0.31    0.00    0.00    0.00    0.00    0.00    0.00   99.28
     09:46:28 AM    2    1.05    0.00    0.89    0.00    0.00    0.08    0.00    0.00    0.00   97.98
     09:46:28 AM    3    0.41    0.00    0.41    0.00    0.00    0.00    0.00    0.00    0.00   99.17
     09:46:28 AM    4    1.13    0.00    0.85    0.00    0.00    0.12    0.00    0.00    0.00   97.89
     09:46:28 AM    5    0.42    0.00    0.38    0.00    0.00    0.00    0.00    0.00    0.00   99.20
     09:46:28 AM    6    1.18    0.00    0.85    0.00    0.00    0.13    0.00    0.00    0.00   97.84
     09:46:28 AM    7    0.39    0.00    0.36    0.00    0.00    0.00    0.00    0.00    0.00   99.25
     09:46:28 AM    8    1.14    0.00    0.84    0.00    0.00    0.12    0.00    0.00    0.00   97.89
     09:46:28 AM    9    0.38    0.00    0.34    0.00    0.00    0.00    0.00    0.00    0.00   99.28
     09:46:28 AM   10    1.16    0.00    0.82    0.00    0.00    0.13    0.00    0.00    0.00   97.90
     09:46:28 AM   11    0.37    0.00    0.33    0.00    0.00    0.00    0.00    0.00    0.00   99.29
     09:46:28 AM   12    1.02    0.00    0.78    0.00    0.00    0.08    0.00    0.00    0.00   98.11
     09:46:28 AM   13    0.36    0.00    0.32    0.03    0.00    0.00    0.00    0.00    0.00   99.28
     09:46:28 AM   14    1.00    0.00    0.75    0.01    0.00    0.07    0.00    0.00    0.00   98.17
     09:46:28 AM   15    0.35    0.00    0.32    0.02    0.00    0.00    0.00    0.00    0.00   99.30
     ```

     