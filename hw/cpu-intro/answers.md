## Q1
- Prediction / 预测:
Time   PID 0   PID 1   CPU   IOs
1    RUN     READY    1
2    RUN     READY    1
3    RUN     READY    1
4    RUN     READY    1
5    RUN     READY    1
6    READY   RUN      1
7    READY   RUN      1
8    READY   RUN      1
9    READY   RUN      1
10   READY   RUN      1
- Reasoning / 理由:两个进程都只包含 CPU 指令（没有 I/O）。根据提示，系统默认在当前进程执行完毕或者发起 I/O 之前，都不会切换进程。因此，PID 0 会先连续运行 5 个 tick，然后 PID 1 再连续运行 5 个 tick。总耗时 10 个 tick，期间 CPU 完全没有空闲，利用率是 100%。
- Verified result / 验证结果:Total Time: 10, CPU Busy: 100%
- Analysis / 分析:Total Time: 10, CPU Busy: 100%

## Q2
- Prediction / 预测:
Time   PID 0   PID 1   CPU   IOs
  1    RUN     READY    1
  2    RUN     READY    1
  3    RUN     READY    1
  4    RUN     READY    1
  5    DONE    RUN:io   1
  6    DONE    BLOCKED  0     1
  7    DONE    BLOCKED  0     1
  8    DONE    BLOCKED  0     1
  9    DONE    BLOCKED  0     1
 10    DONE    BLOCKED  0     1
 11*   DONE    RUN:io_done 1
- Reasoning / 理由:PID 0 先连续执行完 4 条 CPU 指令（tick 1-4）。PID 1 在第 5 个 tick 发起 I/O 操作，占用 1 tick 的 CPU，随后进入 5 个 tick 的阻塞状态（tick 6-10），此时 CPU 空闲。第 11 个 tick，I/O 完成，PID 1 再次占用 1 tick 的 CPU 处理完成信号，随后结束。
- Verified result / 验证结果:Total Time: 11, CPU Busy: 54.55%, IO Busy: 45.45%
- Analysis / 分析：预测完全正确。因为 PID 0 已经提前结束，在 PID 1 阻塞（等待 I/O 设备）的 5 个 tick 里，CPU 没有其他进程可以运行，所以只能空闲等待，导致总时间被拉长到 11 个 tick。

## Q3
- Prediction / 预测:Time   PID 0   PID 1   CPU   IOs
1    RUN     READY    1     1
2    BLOCKED RUN      1     1
3    BLOCKED RUN      1     1
4    BLOCKED RUN      1     1
5    BLOCKED RUN      1     1
6    BLOCKED DONE     0     1
7*   RUN:io_done DONE 1
- Reasoning / 理由:PID 0 先发起 I/O 占用 1 tick CPU，随后进入 5 tick 阻塞。在 PID 0 阻塞期间，PID 1（4条CPU指令）抢到了 CPU，在 tick 2-5 跑完前 4 条，tick 6 时 PID 1 已经结束。此时 CPU 虽然空闲，但 PID 0 的 I/O 尚未完成，直到 tick 7 才重新占用 CPU 处理完成信号。因为 PID 1 充分利用了 PID 0 的阻塞时间，所以 CPU 利用率比 Q2 高得多。
- Verified result / 验证结果:Total Time: 7, CPU Busy: 85.71%
- Analysis / 分析:预测基本正确。虽然总时间是7，但 CPU 利用率是 85.71%（6/7），因为在 tick 6 时 PID 1 已经结束，而 PID 0 还在等 I/O，导致 CPU 有 1 个 tick 的空闲

## Q4
- Prediction / 预测:
Time   PID 0   PID 1   CPU   IOs
  1    RUN     READY    1     1
  2    BLOCKED READY    0     1
  3    BLOCKED READY    0     1
  4    BLOCKED READY    0     1
  5    BLOCKED READY    0     1
  6    BLOCKED READY    0     1
  7*   RUN:io_done READY 1
  8    DONE    RUN      1
  9    DONE    RUN      1
 10    DONE    RUN      1
 11    DONE    RUN      1
- Reasoning / 理由:因为设置了 -S SWITCH_ON_END，系统在当前进程结束（或发起 I/O 后进入阻塞，不切换）之前不会调度其他进程。导致 PID 0 阻塞时，CPU 空闲了 5 个 tick（tick 2-6）。PID 0 结束后，PID 1 才得以运行，总时间被拉长到了 11 tick。
- Verified result / 验证结果:Total Time: 11, CPU Busy: 54.55%
- Analysis / 分析:预测完全正确。SWITCH_ON_END 导致系统在进程进入阻塞时无法切换，CPU 在 tick 2-6 期间空闲，拉长了总时间。

## Q5
- Prediction / 预测:
Time   PID 0   PID 1   CPU   IOs
  1    RUN     READY    1     1
  2    BLOCKED RUN      1     1
  3    BLOCKED RUN      1     1
  4    BLOCKED RUN      1     1
  5    BLOCKED RUN      1     1
  6    BLOCKED DONE     0     1
  7*   RUN:io_done DONE 1
- Reasoning / 理由:设置 -S SWITCH_ON_IO 允许系统在进程发起 I/O 进入阻塞时，立即切换给其他就绪进程。因此，当 PID 0 在 tick 1 发起 I/O 并阻塞后，CPU 在 tick 2 立刻被分配给 PID 1。PID 1 用 4 个 tick 完成 CPU 运算，而 PID 0 在 tick 7 完成 I/O 并结束。总时间 7 tick。
- Verified result / 验证结果:Total Time: 7, CPU Busy: 6 (85.71%)
- Analysis / 分析:预测完全正确。这与 Q3 的结果完全一致（因为 Q3 的默认行为就是 SWITCH_ON_IO），而与 Q4 形成鲜明对比。证明 SWITCH_ON_IO 使得 CPU 在进程阻塞时不会空闲。

## Q6
- Prediction / 预测:
Time   PID 0   PID 1   PID 2   PID 3   CPU   IOs
  1    RUN:io  READY   READY   READY   1     1
  2    BLOCKED RUN     READY   READY   1     1
  3    BLOCKED RUN     READY   READY   1     1
  4    BLOCKED RUN     READY   READY   1     1
  5    BLOCKED RUN     READY   READY   1     1
  6    BLOCKED RUN     READY   READY   1     1
  7    BLOCKED DONE    RUN     READY   1     1
  8    BLOCKED DONE    RUN     READY   1     1
  9    BLOCKED DONE    RUN     READY   1     1
 10    BLOCKED DONE    RUN     READY   1     1
 11    BLOCKED DONE    RUN     READY   1     1
 12    BLOCKED DONE    DONE    RUN     1     1
 13    BLOCKED DONE    DONE    RUN     1     1
 14    BLOCKED DONE    DONE    RUN     1     1
 15    BLOCKED DONE    DONE    RUN     1     1
 16    BLOCKED DONE    DONE    RUN     1     1
 17    RUN:io_done DONE DONE   DONE    1
- Reasoning / 理由:根据 -S SWITCH_ON_IO 和 -I IO_RUN_LATER（默认值），系统在 PID 0 阻塞后会立即调度其他就绪进程（PID 1），但在 PID 0 I/O 完成后，不会立即重新运行它，而是让它在队列中等待（IO_RUN_LATER）。因此 PID 1、2、3 依次执行完各自的 5 个 CPU 指令后，PID 0 才在最后被调度，完成 io_done。
- Verified result / 验证结果:Total Time: 17, CPU Busy: 16 (94.12%), IO Busy: 1 (5.88%)
- Analysis / 分析: 预测基本正确。总时间为 17 tick，CPU 利用率高达 94.12%。

## Q7
- Prediction / 预测:
Time   PID 0   PID 1   PID 2   PID 3   CPU   IOs
  1    RUN:io  READY   READY   READY   1     1
  2    BLOCKED RUN     READY   READY   1     1
  3    BLOCKED RUN     READY   READY   1     1
  4    BLOCKED RUN     READY   READY   1     1
  5    BLOCKED RUN     READY   READY   1     1
  6    BLOCKED RUN     READY   READY   1     1
  7*   RUN:io_done RUN  READY  READY   1     1
  8    DONE    RUN     READY   READY   1     1
  9    DONE    DONE    RUN     READY   1     1
 10    DONE    DONE    RUN     READY   1     1
 11    DONE    DONE    RUN     READY   1     1
 12    DONE    DONE    RUN     READY   1     1
 13    DONE    DONE    RUN     READY   1     1
 14    DONE    DONE    DONE    RUN     1     1
 15    DONE    DONE    DONE    RUN     1     1
 16    DONE    DONE    DONE    RUN     1     1
 17    DONE    DONE    DONE    RUN     1     1
 18    DONE    DONE    DONE    RUN     1     1
- Reasoning / 理由:设置 -I IO_RUN_IMMEDIATE 后，当 PID 0 的 I/O 在 tick 7 完成时，系统会立即重新运行 PID 0，而不是让它等待。因此 PID 0 在 tick 7 直接抢占 CPU 完成 io_done 并结束。随后 PID 1、2、3 才依次执行完它们的 CPU 任务。
- Verified result / 验证结果:Total Time: 18, CPU Busy: 17 (94.44%), IO Busy: 1 (5.56%)
- Analysis / 分析:预测基本正确。IO_RUN_IMMEDIATE 使得 I/O 频繁的进程可以立即响应，改善了 I/O 效率，但代价是需要中断当前进程的 CPU 任务，导致总时间略微拉长（18 tick）。

## Q8
- Prediction / 预测:Q8使用随机指令，预测默认和 `IO_RUN_IMMEDIATE` 效率较高，`SWITCH_ON_END` 会导致 CPU 大量空闲。
- Reasoning / 理由:Q8的指令随机混合了 CPU 和 I/O。`SWITCH_ON_END` 会阻止系统在 I/O 阻塞时切换进程，导致 CPU 空转；而 `IO_RUN_IMMEDIATE` 可以让频繁 I/O 的进程快速响应，减少 I/O 等待。
- Verified result / 验证结果: - 默认: Total Time 15, CPU Busy 53.33%
  -  -I IO_RUN_IMMEDIATE: Total Time 15, CPU Busy 53.33%
  -  -S SWITCH_ON_END: Total Time 18, CPU Busy 44.44%
- Analysis / 分析:结果证实了预测。`SWITCH_ON_END` 由于不允许在 I/O 阻塞时切换进程，导致 CPU 空闲时间增加，总时间被拉长到 18 tick。而默认策略和 `IO_RUN_IMMEDIATE` 允许切换，CPU 利用率相对较高，总时间只有 15 tick。

