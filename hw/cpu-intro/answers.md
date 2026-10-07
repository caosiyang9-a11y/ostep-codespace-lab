# OSTEP Ch.4 Homework Answers

## Q1
- Prediction / 预测：总时间10 ticks，CPU利用率100%，CPU不会空闲
- Reasoning / 理由：两个进程都是纯CPU（5:100），默认SWITCH_ON_IO策略下只有发生I/O才切换。PID 0先跑完5条指令，然后PID 1跑5条，CPU一直有事做。
- Verified result / 验证结果：Total Time 10, CPU Busy 10 (100.00%), IO Busy 0 (0.00%)
- Analysis / 分析：预测完全正确。纯CPU进程在非抢占式调度下就是串行执行，CPU利用率100%，没有任何空闲时间。

## Q2
- Prediction / 预测：总时间约11 ticks，CPU Busy约6次，利用率约55%
- Reasoning / 理由：PID 0是4条纯CPU，PID 1是1条I/O。默认SWITCH_ON_IO下，PID 0先跑完4个CPU tick，然后PID 1开始走完整I/O周期：RUN:io(1) + BLOCKED(5) + RUN:io_done(1) = 7 ticks。CPU Busy = 4 + 2 = 6 ticks。
- Verified result / 验证结果：Total Time 11, CPU Busy 6 (54.55%), IO Busy 5 (45.45%)
- Analysis / 分析：预测正确。I/O阻塞的5个tick里CPU完全空闲，因为PID 0已经跑完了。这就是串行执行的代价，CPU和I/O设备没有并行工作。

## Q3
- Prediction / 预测：总时间约7 ticks，CPU利用率约86%
- Reasoning / 理由：PID 0先跑t=1发起I/O，因为SWITCH_ON_IO，CPU立刻切换到PID 1。PID 1跑4个CPU tick，正好和PID 0的5个I/O阻塞tick并行。
- Verified result / 验证结果：Total Time 7, CPU Busy 6 (85.71%), IO Busy 5 (71.43%)
- Analysis / 分析：预测正确。和Q2对比，总时间从11降到7，CPU利用率从54.55%升到85.71%。核心原因是I/O阻塞期间CPU跑了PID 1的4条指令——CPU和I/O设备并行工作，谁也没闲着。这就是并发的价值。

## Q4
- Prediction / 预测：总时间约11 ticks，CPU利用率约55%
- Reasoning / 理由：SWITCH_ON_END策略下，只有当前进程完全结束才切换。所以PID 0要完整跑完整个I/O周期（7 ticks），PID 1才能开始跑4条CPU。
- Verified result / 验证结果：Total Time 11, CPU Busy 6 (54.55%), IO Busy 5 (45.45%)
- Analysis / 分析：预测正确。和Q2结果完全一样——因为都是串行执行。I/O阻塞的5个tick里CPU完全空闲，因为SWITCH_ON_END不切换进程。

## Q5
- Prediction / 预测：总时间7 ticks，CPU利用率85.71%（即Q3的结果）
- Reasoning / 理由：SWITCH_ON_IO是默认值，所以Q5就是Q3的配置。对比Q4的SWITCH_ON_END，总时间从11降到7，CPU利用率大幅提升。
- Verified result / 验证结果：Total Time 7, CPU Busy 6 (85.71%), IO Busy 5 (71.43%)
- Analysis / 分析：同样的进程，仅仅切换策略不同，总时间差了4个tick（36%），CPU利用率差了31个百分点。SWITCH_ON_IO让CPU在I/O等待期间可以跑其他进程，这是多道程序设计的核心思想。

## Q6
- Prediction / 预测：总时间约30 ticks，CPU利用率约68%
- Reasoning / 理由：IO_RUN_LATER策略下，I/O完成后不抢占CPU，而是排队等当前CPU进程跑完。PID 0的第一条I/O完成后，会排队等PID 1/2/3都跑完才轮到它处理io_done，然后再发起下一条I/O。后面两条I/O的等待时间里CPU会空闲。
- Verified result / 验证结果：Total Time 31, CPU Busy 21 (67.74%), IO Busy 15 (48.39%)
- Analysis / 分析：预测基本正确。IO_RUN_LATER策略下，PID 0的I/O完成后不立即运行，而是排队等所有CPU进程都跑完。结果就是后面两条I/O的等待时间里CPU完全空闲——I/O设备和CPU没有重叠工作。

## Q7
- Prediction / 预测：总时间约21 ticks，CPU利用率接近100%
- Reasoning / 理由：IO_RUN_IMMEDIATE策略下，I/O一完成就立刻抢占CPU，处理完马上发起下一条I/O。这样I/O设备几乎一直有活干，同时CPU也一直有事做。
- Verified result / 验证结果：Total Time 21, CPU Busy 21 (100.00%), IO Busy 15 (71.43%)
- Analysis / 分析：预测正确，CPU利用率真的达到了100%！对比Q6（IO_RUN_LATER）：总时间从31降到21，CPU利用率从67.74%升到100%，I/O Busy从48.39%升到71.43%。核心原理：I/O密集型进程完成后立即运行，让I/O设备保持忙碌，同时CPU也插空处理io操作，CPU从不空闲。

## Q8
- Prediction / 预测：三个种子结果差异很大，因为随机生成的指令序列不同。CPU和I/O的重叠程度取决于两个进程的指令交替情况。
- Reasoning / 理由：每个进程3条指令，每条50%概率CPU或I/O，所以指令序列组合很多。如果两个进程同时发起I/O，CPU会空闲；如果一个在I/O时另一个在跑CPU，利用率就高。
- Verified result / 验证结果：三个种子总时间从15到18不等，差异主要来自指令序列的随机性。
- Analysis / 分析：
  - 种子不同，指令序列不同，总时间和利用率差异很大。
  - 对比不同-S和-I设置：SWITCH_ON_END会让总时间大幅增加（I/O等待时CPU不切换）；IO_RUN_IMMEDIATE会缩短总时间（I/O完成后立即运行，保持I/O设备忙碌）。
  - 核心结论：进程调度策略对系统性能影响巨大，尤其是在CPU和I/O混合负载下。
